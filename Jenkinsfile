pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'

        FRONTEND_REPO = '113892881104.dkr.ecr.us-east-1.amazonaws.com/devops-challenge-frontend'
        BACKEND_REPO  = '113892881104.dkr.ecr.us-east-1.amazonaws.com/devops-challenge-backend'

        ECR_REGISTRY = '113892881104.dkr.ecr.us-east-1.amazonaws.com'
    }

    stages {
        stage('Checkout code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    sh 'docker build -t frontend:latest ./frontend'
                    sh 'docker build -t backend:latest ./backend'
                }
            }
        }

        stage('Authenticate to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    script {
                        sh '''
                            aws --version
                            aws ecr get-login-password --region $AWS_REGION | \
                            docker login --username AWS --password-stdin $ECR_REGISTRY
                        '''
                    }
                }
            }
        }

        stage('Tag and Push images to ECR') {
            steps {
                script {
                    sh '''
                        docker tag frontend:latest $FRONTEND_REPO:latest
                        docker tag backend:latest $BACKEND_REPO:latest

                        docker push $FRONTEND_REPO:latest
                        docker push $BACKEND_REPO:latest
                    '''
                }
            }
        }

        stage('Update ECS services') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    script {
                        sh '''
                            aws ecs update-service \
                                --cluster devops-challenge-cluster \
                                --service devops-challenge-frontend-service \
                                --force-new-deployment \
                                --region $AWS_REGION

                            aws ecs update-service \
                                --cluster devops-challenge-cluster \
                                --service devops-challenge-backend-service \
                                --force-new-deployment \
                                --region $AWS_REGION
                        '''
                    }
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