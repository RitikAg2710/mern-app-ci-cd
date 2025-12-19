pipeline {
    agent any

    environment {
        AWS_REGION   = "ap-south-1"
        ACCOUNT_ID   = "940925916864"
        ECR_REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG    = "latest"
        NAMESPACE    = "mern-app"
    }

    stages {

        stage('Verify Workspace') {
            steps {
                sh '''
                echo "📂 Current directory:"
                pwd
                echo "📁 Listing files:"
                ls -l
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t frontend Application-Code/frontend
                docker build -t backend  Application-Code/backend
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-jenkins']]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION |
                    docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Tag & Push Images') {
            steps {
                sh '''
                docker tag frontend:latest $ECR_REGISTRY/frontend:$IMAGE_TAG
                docker tag backend:latest  $ECR_REGISTRY/backend:$IMAGE_TAG

                docker push $ECR_REGISTRY/frontend:$IMAGE_TAG
                docker push $ECR_REGISTRY/backend:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl rollout restart deployment/frontend -n $NAMESPACE
                kubectl rollout restart deployment/backend  -n $NAMESPACE

                kubectl rollout status deployment/frontend -n $NAMESPACE
                kubectl rollout status deployment/backend  -n $NAMESPACE
                '''
            }
        }
    }
}
