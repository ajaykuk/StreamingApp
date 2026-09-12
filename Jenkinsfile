pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1' 
        ECR_REGISTRY = '490600801130.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_TAG = "${env.BUILD_ID}"
    }
    stages {
        stage('ECR Login') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
            }
        }
        stage('Build and Push Images') {
            steps {
                // Injects the .env file securely from Jenkins credentials
                withCredentials([file(credentialsId: 'my-env-file', variable: 'ENV_FILE')]) {
                    script {
                        // Replace these with your actual 5 folder/ECR repo names
                        def services = ['malik/streaming-app-frontend', 'malik/streaming-backend-admin-service', 'malik/streaming-backend-auth-service', 'malik/streaming-backend-chat-service', 'malik/streaming-backend-streaming-service']
                        
                        for (service in services) {
                            // Copies the secure .env file into the current service's folder before building
                            sh "cp \${ENV_FILE} ./${service}/.env"
                            
                            // Builds the image for the current service
                            sh "docker build -t ${ECR_REGISTRY}/${service}:${IMAGE_TAG} ./${service}"
                            
                            // Pushes the newly built image to ECR
                            sh "docker push ${ECR_REGISTRY}/${service}:${IMAGE_TAG}"
                        }
                    }
                }
            }
        }
    }
}
