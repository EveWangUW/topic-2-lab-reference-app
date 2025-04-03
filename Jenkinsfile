// pipeline {
//     agent any
    
//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }
//         stage('Build') {
//             steps {
//                 sh 'docker build . -t $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
//             }
//         }
//         stage('Push Image') {
//             steps {
//                 sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com'
//                 sh 'aws ecr describe-repositories --repository-names capstone-repo-test-eve || aws ecr create-repository --repository-name capstone-repo-test-eve'
//                 sh 'docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
//             }
//         }
//         stage('Deploy') {
//             steps {
//                 sh 'docker pull $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
//                 sh 'docker rm -f capstone-repo-test-eve || true'
//                 sh 'docker run -d -p "2002:80" --name capstone-repo-test-eve $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
//             }
//         }
//     }
// }
pipeline {
    agent any
    
    environment {
        ACR_NAME = 'ACR1eve2wang'
        IMAGE_NAME = 'Image1'
        CONTAINER_NAME = 'Container1'
        HOST_PORT = '2012'
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
                    // Implicitly uses 'latest' tag
                    sh "docker build -t ${ACR_NAME}.azurecr.io/${IMAGE_NAME} ."
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    withCredentials([azureServicePrincipal('AZURE_CREDENTIALS_ID')]) {
                        sh "az login --service-principal -u \$AZURE_CLIENT_ID -p \$AZURE_CLIENT_SECRET -t \$AZURE_TENANT_ID"
                        sh "az acr login --name ${ACR_NAME}"
                    }
                    sh "docker push ${ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh "docker rm -f ${CONTAINER_NAME} || true"
                    sh """
                        docker pull ${ACR_NAME}.azurecr.io/${IMAGE_NAME}
                        docker run -d -p ${HOST_PORT} \
                            --name ${CONTAINER_NAME} \
                            ${ACR_NAME}.azurecr.io/${IMAGE_NAME}
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}