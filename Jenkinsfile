pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ACCOUNT_ID = "940925916864"
        ECR_REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SAEEMSAKADIRI/mern-app-ci-cd.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t frontend app/Application-Code/frontend
                docker build -t backend  app/Application-Code/backend
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-creds']]) {
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
                kubectl apply -f terraform/eks/k8s-manifests/mongo
                kubectl apply -f terraform/eks/k8s-manifests/backend
                kubectl apply -f terraform/eks/k8s-manifests/frontend
                '''
            }
        }
    }
}
