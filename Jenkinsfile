pipeline {
    agent any
    
    environment {
        ACR_REGISTRY = 'labacrdevops3.azurecr.io'
        ACR_REPO = 'acrrepo1'
        DOCKER_IMAGE = "${ACR_REGISTRY}/${ACR_REPO}" //labacrdevops3.azurecr.io/acrrepo1
        CONTAINER_NAME = "mycontainer"
    }
    
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps {
                script {
                    // Add build context (.) and include registry URL in tag
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    // Secure login using Jenkins credentials
                    sh "docker login ${ACR_REGISTRY} --username StudentToken --password <Password>"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
                
        stage('Deploy') {
            steps {
                script {
                    // Ensure registry name consistency (fixed typo)
                    sh "docker pull ${DOCKER_IMAGE}:latest"
                    sh "docker rm -f ${CONTAINER_NAME} || true"
                    sh """
                        docker run -d -p 2003:80 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
    }
}