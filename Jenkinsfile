pipeline {
    agent any
    
    triggers {
        githubPush() 
    }

    options {
        skipDefaultCheckout(false)
    }
    
    environment {
        AWS_REGION = 'us-east-1' 
        ECR_REGISTRY = '490600801130.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_TAG = "${env.BUILD_ID}"
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                checkout scm // Pulls the repository code back into the clean workspace
            }
        }

        stage('ECR Login') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
            }
        }
        
        stage('Build and Push Images') {
            steps {
                script {
                    // Map each folder to its ECR repo and its specific Jenkins Secret File ID
                    def services = [
                        'frontend':               [repo: 'malik/streaming-app-frontend',             credId: 'env-frontend'],
                        'backend/adminService':   [repo: 'malik/streaming-backend-admin-service',    credId: 'env-admin'],
                        'backend/authService':    [repo: 'malik/streaming-backend-auth-service',     credId: 'env-auth'],
                        'backend/chatService':    [repo: 'malik/streaming-backend-chat-service',     credId: 'env-chat'],
                        'backend/streamingService':[repo: 'malik/streaming-backend-streaming-service', credId: 'env-streaming']
                    ]
                    
                    for (String localFolder : services.keySet()) {
                        def ecrRepo = services[localFolder].repo
                        def envCredId = services[localFolder].credId
    
                        echo "Building local folder: ${localFolder} | Pushing to ECR: ${ecrRepo} | Env ID: ${envCredId}"
    
                        // Pull the specific .env file for THIS service inside the loop
                        withCredentials([file(credentialsId: envCredId, variable: 'ENV_FILE')]) {
                            sh "cp \${ENV_FILE} ./${localFolder}/.env"
                            
                            sh "docker build -t ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG} ./${localFolder}"
                            sh "docker push ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG}"
                            
                            // Clean up the local image to save disk space
                            sh "docker rmi ${ECR_REGISTRY}/${ecrRepo}:${IMAGE_TAG}"
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                  // Update kubeconfig so Jenkins has access to the cluster
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name my-eks-cluster"
                    
                    // Deploy the new images using the Helm chart
                    sh "helm upgrade --install streaming-app ./streaming-mern-stack --set imageTag=${IMAGE_TAG}"                }
            }
        }
    }
}