pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        VPS_SSH_CREDENTIALS = credentials('vps-ssh-credentials')
        DOCKER_REGISTRY = 'pragya2902/chattingo'
        VPS_IP = '98.83.13.215'
        VPS_USERNAME = 'ubuntu'
        VPS_DEPLOY_PATH = '/home/ubuntu/chattingo'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Backend') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}-backend:${env.BUILD_NUMBER}", "-f backend/Dockerfile .")
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}-frontend:${env.BUILD_NUMBER}", "-f frontend/Dockerfile .")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Run backend tests
                    sh 'cd backend && mvn test'
                    
                    // Run frontend tests if any
                    sh 'cd frontend && npm test -- --watchAll=false'
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    
                    // Push backend image
                    sh "docker push ${DOCKER_REGISTRY}-backend:${env.BUILD_NUMBER}"
                    
                    // Push frontend image
                    sh "docker push ${DOCKER_REGISTRY}-frontend:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to VPS') {
            steps {
                script {
                    // SSH into VPS and deploy
                    sshagent(credentials: ['vps-ssh-credentials']) {
                        // Create deployment directory if it doesn't exist
                        sh "ssh -o StrictHostKeyChecking=no ${VPS_USERNAME}@${VPS_IP} 'mkdir -p ${VPS_DEPLOY_PATH}'"
                        
                        // Copy docker-compose and env files
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yml ${VPS_USERNAME}@${VPS_IP}:${VPS_DEPLOY_PATH}/"
                        
                        // Update images in docker-compose
                        sh """
                        ssh -o StrictHostKeyChecking=no ${VPS_USERNAME}@${VPS_IP} ""\"
                            cd ${VPS_DEPLOY_PATH}
                            sed -i \"s|image: .*backend.*|image: ${DOCKER_REGISTRY}-backend:${env.BUILD_NUMBER}|g\" docker-compose.yml
                            sed -i \"s|image: .*frontend.*|image: ${DOCKER_REGISTRY}-frontend:${env.BUILD_NUMBER}|g\" docker-compose.yml
                            docker-compose pull
                            docker-compose down
                            docker-compose up -d
                        """
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            // Add notification (e.g., Slack, Email)
        }
        failure {
            echo 'Pipeline failed!'
            // Add failure notification
        }
        always {
            // Clean up
            sh 'docker logout'
        }
    }
}
