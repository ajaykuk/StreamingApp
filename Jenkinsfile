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
                            
                            
                            'StreamingApp/frontend': 'malik/streaming-app-frontend',
                            'StreamingApp/backend/adminService': 'malik/streaming-backend-admin-service',
                            'StreamingApp/backend/authService': 'malik/streaming-backend-auth-service',
                            'StreamingApp/backend/chatService': 'malik/streaming-backend-chat-service',
                            'StreamingApp/backend/streamingService': 'malik/streaming-backend-streaming-service' 
                             ]
                        
                        for (String localFolder : services.keySet()) {
        // Fetch the ECR repo name dynamically based on the current folder
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
