pipeline {
  agent {
    kubernetes {
      // Định nghĩa một Pod tạm thời để thực hiện việc Build
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:dind
            command: ['dockerd-entrypoint.sh']
            args: ['--storage-driver', 'overlay2']
            tty: true
            securityContext:
              privileged: true
      '''
    }
  }
  environment {
    // --- CẤU HÌNH ---
    DOCKER_IMAGE_FE = '22120359/covid-frontend'
    DOCKER_IMAGE_BE = '22120359/covid-backend'
    REGISTRY_CRED = 'dockerhub-id'
  }
  
  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    
    stage('Build & Push Docker') {
      parallel {
        stage('Backend') {
          steps {
            container('docker') {
              dir('server') {
                script {
                  // Đăng nhập Docker Hub
                  docker.withRegistry('', REGISTRY_CRED) {
                    // Build và Push phiên bản theo số Build (v1, v2...)
                    sh "docker build -t ${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER} ."
                    sh "docker push ${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER}"
                    
                    // Push thêm tag 'latest'
                    sh "docker build -t ${DOCKER_IMAGE_BE}:latest ."
                    sh "docker push ${DOCKER_IMAGE_BE}:latest"
                  }
                }
              }
            }
          }
        }
        
        stage('Frontend') {
          steps {
            container('docker') {
              dir('client') {
                script {
                  docker.withRegistry('', REGISTRY_CRED) {
                    sh "docker build -t ${DOCKER_IMAGE_FE}:${env.BUILD_NUMBER} ."
                    sh "docker push ${DOCKER_IMAGE_FE}:${env.BUILD_NUMBER}"
                    
                    sh "docker build -t ${DOCKER_IMAGE_FE}:latest ."
                    sh "docker push ${DOCKER_IMAGE_FE}:latest"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}