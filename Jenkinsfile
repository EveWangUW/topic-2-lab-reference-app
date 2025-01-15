pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'docker build . -t $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
            }
        }
        stage('Push Image') {
            steps {
                sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com'
                sh 'aws ecr describe-repositories --repository-names capstone-repo-test-eve || aws ecr create-repository --repository-name capstone-repo-test-eve'
                sh 'docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker pull $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
                sh 'docker rm -f capstone-repo-test-eve || true'
                sh 'docker run -d -p "2002:80" --name capstone-repo-test-eve $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/capstone-repo-test-eve'
            }
        }
    }
}