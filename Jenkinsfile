pipeline {
    agent any
    tools {
        maven 'M3'
    }
    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-amazon-corretto'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }
    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/CarlaMirandaArellano/EFT_DevOps.git'
            }
        }

        stage('Compile and Package Application') {
            steps {
                sh 'mvn clean package -DskipTests'
                sh 'ls -lah target/'
            }
        }

        stage('Build Container Image') {
            steps {
                sh 'docker build -t imagen_usuarios .'
                sh 'docker images | grep imagen_usuarios'
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'db-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS'),
                    string(credentialsId: 'db-url-eft', variable: 'DB_URL')
                ]) {
                    sh '''
                        docker stop contenedor_usuarios || true
                        docker rm contenedor_usuarios || true
                        docker run -d -p 9090:8080 \
                          --name contenedor_usuarios \
                          -e SPRING_DATASOURCE_URL=$DB_URL \
                          -e SPRING_DATASOURCE_USERNAME=$DB_USER \
                          -e SPRING_DATASOURCE_PASSWORD=$DB_PASS \
                          imagen_usuarios
                    '''
                }
            }
        }

        stage('Verify Application Availability') {
            steps {
                sh 'sleep 45'
                sh 'curl -f -s http://localhost:9090/usuariosBuild/user || exit 1'
                sh 'docker ps | grep contenedor_usuarios'
                sh 'docker logs contenedor_usuarios | tail -20'
            }
        }

    }
    post {
        success {
            echo 'Pipeline exitoso - API disponible en http://EC2_IP:9090/usuariosBuild/user'
        }
        failure {
            echo 'Pipeline fallido - revisar Console Output'
        }
    }
}
