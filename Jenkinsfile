pipeline {
    agent any

    environment {
        AZURE_ACR_NAME = 'labacrdevops3'
        IMAGE_NAME = 'docker-image-devops'
        ACR_LOGIN_SERVER = "${AZURE_ACR_NAME}.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    sh "docker build . -t ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'jenkins-acr-scope-map-cred',  // Match the ID you set in Jenkins
                            usernameVariable: 'ACR_USER',
                            passwordVariable: 'ACR_PASS'
                        )
                    ]) {
                        sh """
                            docker login ${ACR_LOGIN_SERVER} -u \$ACR_USER -p \$ACR_PASS
                            docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "docker pull ${ACR_LOGIN_SERVER}/${IMAGE_NAME}"
                sh "docker rm -f ${IMAGE_NAME} || true"
                sh "docker run -d -p 2004:80 --name ${IMAGE_NAME} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}"
            }
        }
    }
}