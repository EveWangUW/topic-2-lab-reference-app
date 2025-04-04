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
        AZURE_ACR_NAME = 'acrevewang1'
        IMAGE_NAME = 'capstone-repo-test-eve-3'
    }
    
    stages {
        stage('Install Azure CLI') {
            steps {
                script {
                    sh '''
                        curl -sL https://aka.ms/InstallAzureCLIDeb > install.sh
                        chmod +x install.sh
                        ./install.sh  # Requires root access
                        az --version
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Implicit 'latest' tag
                    sh "docker build . -t ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'azure-acr-service-principal',  // Match the ID you set in Jenkins
                            usernameVariable: 'AZURE_CLIENT_ID',
                            passwordVariable: 'AZURE_CLIENT_SECRET'
                        )
                    ]) {
                        // Azure login
                        sh "az login --service-principal -u \$AZURE_CLIENT_ID -p \$AZURE_CLIENT_SECRET --tenant f6b6dd5b-f02f-441a-99a0-162ac5060bd2"
                        
                        // ACR login + Docker push
                        sh "az acr login --name ${AZURE_ACR_NAME}"
                        sh "docker push ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                    }
                }
            }
        }
                
        stage('Deploy') {
            steps {
                script {
                    sh "docker pull ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                    sh "docker rm -f ${IMAGE_NAME} || true"
                    sh """
                        docker run -d \
                        -p 2003:80 \
                        --name ${IMAGE_NAME} \
                        ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}
                    """
                }
            }
        }
    }
}