pipeline {
    agent {
        kubernetes {
            label 'docker'
            defaultContainer 'jnlp'
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:20.10  # เลือก Docker image ที่มี CLI สำหรับใช้งาน docker command
                command:
                - cat
                volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
              volumes:
              - name: docker-socket
                hostPath:
                  path: /var/run/docker.sock
            """
        }
    }
    environment {
        DOCKER_REPO = 'tsdadmin/villageaccountingapi'  // ชื่อ Docker repository ของคุณ
        DOCKER_CREDENTIALS_ID = 'docker-hub'           // Jenkins credentials ID สำหรับ Docker Hub
        GIT_CREDENTIALS_ID = 'gitlab-token'            // Jenkins credentials ID สำหรับ GitLab หรือ GitHub
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
                container('docker') {
                    script {
                        // สร้าง Docker image โดยใช้ Jenkins build ID เป็น tag
                        docker.build("${DOCKER_REPO}:${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                container('docker') {
                    script {
                        // Push Docker image ไปยัง Docker Hub โดยใช้ credentials สำหรับ Docker Hub
                        docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                            docker.image("${DOCKER_REPO}:${env.BUILD_ID}").push()
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            container('docker') {
                script {
                    // ลบ Docker image หลังจาก build เสร็จสิ้น
                    docker.image("${DOCKER_REPO}:${env.BUILD_ID}").remove()
                }
            }
        }
    }
}
