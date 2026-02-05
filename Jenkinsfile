pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "123456789012"
        ECR_REPO_NAME = "hello-world-dotnet"
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build .NET Application') {
            steps {
                sh '''
                  dotnet --version
                  dotnet restore
                  dotnet publish -c Release -o publish
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                  aws ecr get-login-password --region $AWS_REGION \
                  | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed to ECR: $ECR_REPO:$IMAGE_TAG"
        }
        failure {
            echo "❌ Jenkins pipeline failed"
        }
    }
}
