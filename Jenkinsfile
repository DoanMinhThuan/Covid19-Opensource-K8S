pipeline {
    agent any
    
    environment {      
        // Ví dụ (bạn hãy sửa lại dòng dưới):
        DOCKER_IMAGE_FE = '22120359/covid-frontend' 
        DOCKER_IMAGE_BE = '22120359/covid-backend'
        SSH_CRED_ID = 'ec2-ssh-key'
        // ID này phải trùng khớp với ID bạn vừa tạo trong Jenkins
        REGISTRY_CRED = 'dockerhub-id' 
        APP_SERVER_IP = '172.31.35.113'
        APP_USER = 'ubuntu'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                // Lấy code mới nhất từ GitHub về
                checkout scm
            }
        }
        
        stage('Build & Push Images') {
            parallel {
                stage('Backend') {
                    steps {
                        script {
                            dir('server') {
                                echo '--- Building Backend Image ---'
                                docker.withRegistry('', REGISTRY_CRED) {
                                    // Build image và gắn tag theo số lần build (env.BUILD_NUMBER)
                                    def backendImage = docker.build("${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER}")
                                    
                                    // Push image lên DockerHub
                                    backendImage.push()
                                    
                                    // Push thêm tag 'latest' để tiện deploy sau này
                                    backendImage.push('latest')
                                }
                            }
                        }
                    }
                }
                
                stage('Frontend') {
                    steps {
                        script {
                            dir('client') {
                                echo '--- Building Frontend Image ---'
                                docker.withRegistry('', REGISTRY_CRED) {
                                    def frontendImage = docker.build("${DOCKER_IMAGE_FE}:${env.BUILD_NUMBER}")
                                    frontendImage.push()
                                    frontendImage.push('latest')
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                script {
                    echo '--- Deploying to App Server ---'
                    
                    sshagent([SSH_CRED_ID]) {
                        
                        // 1. Copy file docker-compose.yml sang server
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yml ${APP_USER}@${APP_SERVER_IP}:/home/ubuntu/docker-compose.yml"
                        
                        // 2. SSH sang để pull và up container
                        sh """
                            ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER_IP} '
                                cd /home/ubuntu
                                docker compose down
                                docker compose pull
                                docker compose up -d
                            '
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Dọn dẹp docker rác sau khi build xong để tiết kiệm ổ cứng
            sh 'docker system prune -f' 
        }
    }
}