pipeline {
    agent any

    environment {
        IMAGE_NAME = "vijayarajvira/hello-docker"
        CONTAINER_NAME = "hello-container"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/vijayarajvira/devops-learning.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                    echo "🏗️ Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo '🔐 Logging into Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing Docker images...'
                script {
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy using Docker Compose') {
    steps {
        echo '🚀 Deploying using Docker Compose...'
        script {
           sh '''
echo "🧹 Removing old containers..."
docker rm -f hello-container || true
docker compose down
echo "🆕 Deploying new version..."
docker compose pull
docker compose up -d
'''


             }
    }
}

        stage('Health Check') {
            steps {
                echo '🩺 Checking service health...'
                script {
                    sh 'sleep 5'
                    def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8088", returnStdout: true).trim()
                    if (status == '200') {
                        echo "✅ Service is healthy and responding correctly!"
                    } else {
                        error("❌ Health check failed! HTTP status: ${status}")
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Image pushed and service redeployed via Docker Compose."
        }
        failure {
            echo "❌ Pipeline failed. Check the Jenkins logs for details."
        }
    }
}

