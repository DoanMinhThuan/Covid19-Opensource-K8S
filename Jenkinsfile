pipeline {
  agent {
    kubernetes {
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
    GIT_REPO = 'github.com/DoanMinhThuan/Covid-Project.git' 
    GIT_CRED_ID = 'github-pat'
    EMAIL_GIT = 'jenkins@bot.com'
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
                  docker.withRegistry('', REGISTRY_CRED) {
                    sh "docker build -t ${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER} ."
                    sh "docker push ${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER}"
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
                  }
                }
              }
            }
          }
        }
      }
    }

    stage('Update Manifest & GitOps') {
      steps {
        script {
            withCredentials([string(credentialsId: GIT_CRED_ID, variable: 'GITHUB_TOKEN')]) {
                sh """
                    # 1. Cấu hình Git ảo
                    git config user.email "${EMAIL_GIT}"
                    git config user.name "Jenkins Bot"

                    # 2. Sửa file YAML bằng lệnh sed (Tìm và thay thế version cũ bằng version mới)
                    # Lưu ý: File nằm trong thư mục k8s/
                    sed -i "s|${DOCKER_IMAGE_BE}.*|${DOCKER_IMAGE_BE}:${env.BUILD_NUMBER}|g" k8s/covid-app.yaml
                    sed -i "s|${DOCKER_IMAGE_FE}.*|${DOCKER_IMAGE_FE}:${env.BUILD_NUMBER}|g" k8s/covid-app.yaml

                    # 3. Commit và Push ngược lên GitHub
                    git add k8s/covid-app.yaml
                    git commit -m "Jenkins Update: Images to version ${env.BUILD_NUMBER}"
                    
                    # Push dùng Token
                    git push https://${GITHUB_TOKEN}@${GIT_REPO} HEAD:main
                """
            }
        }
      }
    }
  }
}