pipeline {
    agent any
    environment {
        DOCKER_REPO = 'tsdadmin/villageaccountingapi' // ชื่อ Docker repository ของคุณ
        DOCKER_CREDENTIALS_ID = 'jen-dockerhub' // Jenkins credentials ID สำหรับ Docker Hub
        GIT_CREDENTIALS_ID = 'gitlab-token' // Jenkins credentials ID สำหรับ GitLab หรือ GitHub
    }
    stages {
        stage('Clone Repository') {
            steps {
                // ดึงโค้ดจาก Git repository โดยใช้ credentials สำหรับ Git
                git credentialsId: GIT_CREDENTIALS_ID, url: 'https://gitlab.com/villageaccounting/villageaccapi.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // สร้าง Docker image โดยใช้ Jenkins build ID เป็น tag
                    docker.build("${DOCKER_REPO}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image ไปยัง Docker Hub โดยใช้ credentials สำหรับ Docker Hub
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_REPO}:${env.BUILD_ID}").push()
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                // ลบ Docker image หลังจาก build เสร็จสิ้น
                docker.image("${DOCKER_REPO}:${env.BUILD_ID}").remove()
            }
        }
    }
}
