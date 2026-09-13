pipeline {
    agent any

    environment {
        APP_NAME       = "node-demo-app"
        TARGET_USER    = "ec2-user"
        TARGET_HOST    = "10.0.1.233" // <-- Put your target EC2 node private IP here
        CONTAINER_PORT = "3000"
        HOST_PORT      = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image Archive') {
            steps {
                sh "docker build -t ${APP_NAME}:latest ."
                sh "docker save ${APP_NAME}:latest -o ${APP_NAME}.tar"
            }
        }

        stage('Deploy to Target Node') {
            steps {
                sshagent(['target-ssh-key']) {
                    sh "scp -o StrictHostKeyChecking=no ${APP_NAME}.tar ${TARGET_USER}@${TARGET_HOST}:/tmp/${APP_NAME}.tar"
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        docker load -i /tmp/${APP_NAME}.tar
                        rm -f /tmp/${APP_NAME}.tar
                        if [ \$(docker ps -aq -f name=${APP_NAME}) ]; then
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                        fi
                        docker run -d --name ${APP_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${APP_NAME}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            sh "rm -f ${APP_NAME}.tar"
        }
    }
}