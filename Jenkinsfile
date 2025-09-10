pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'pragya2902/chattingo'
        VPS_IP          = '98.83.13.215'
        VPS_USERNAME    = 'ubuntu'
        VPS_DEPLOY_PATH = '/home/ubuntu/chattingo'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Backend') {
            steps {
                script {
                    // Build backend with context at project root
                    docker.build("${DOCKER_REGISTRY}-backend:${env.BUILD_NUMBER}", "-f backend/Dockerfile .")
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    // Change to frontend directory for build context
                    dir('frontend') {
                        docker.build("${DOCKER_REGISTRY}-frontend:${env.BUILD_NUMBER}", "-f Dockerfile .")
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                // adjust if your agents already have deps preinstalled
                sh 'cd backend && mvn -B test'
                sh 'cd frontend && npm ci && npm test -- --watchAll=false'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
                sh "docker push ${DOCKER_REGISTRY}-backend:${env.BUILD_NUMBER}"
                sh "docker push ${DOCKER_REGISTRY}-frontend:${env.BUILD_NUMBER}"
            }
        }

        stage('Deploy to VPS') {
            steps {
                sshagent(credentials: ['vps-ssh-credentials']) {
                    // ensure remote path exists
                    sh "ssh -o StrictHostKeyChecking=no ${VPS_USERNAME}@${VPS_IP} 'mkdir -p ${VPS_DEPLOY_PATH}'"
                    // copy compose file
                    sh "scp -o StrictHostKeyChecking=no docker-compose.yml ${VPS_USERNAME}@${VPS_IP}:${VPS_DEPLOY_PATH}/docker-compose.yml"

                    // run the remote deployment atomically
                    sh """
                    ssh -o StrictHostKeyChecking=no -T ${VPS_USERNAME}@${VPS_IP} << 'EOF'
                    set -e
                    cd ${VPS_DEPLOY_PATH}
                    sed -i "s|image: .*backend.*|image: ${DOCKER_REGISTRY}-backend:${BUILD_NUMBER}|g" docker-compose.yml
                    sed -i "s|image: .*frontend.*|image: ${DOCKER_REGISTRY}-frontend:${BUILD_NUMBER}|g" docker-compose.yml
                    docker-compose pull
                    docker-compose down
                    docker-compose up -d
                    EOF
                    """
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
        always  { sh 'docker logout || true' }
    }
}
