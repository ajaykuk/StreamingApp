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
                        def services = [
                            
                            
                            'frontend': 'malik/streaming-app-frontend',
                            'backend/adminService': 'malik/streaming-backend-admin-service',
                            'backend/authservice': 'malik/streaming-backend-auth-service',
                            'backend/chatservice': 'malik/streaming-backend-chat-service',
                            'backend/streamingservice': 'malik/streaming-backend-streaming-service' 
                             ]
                        
                        for (String localFolder : services.keySet()) {
                            def ecrRepo = services[localFolder]
        
                            echo "Building local folder: ${localFolder} | Pushing to ECR: ${ecrRepo}"
        
                            sh "cp \${ENV_FILE} ./${localFolder}/.env"
                            sh "docker build -t ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG} ./${localFolder}"
        
                            sh "docker push ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG}"
                            sh "docker rmi ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG}"
    }
                    }
                }
            }
        }
    }
}
