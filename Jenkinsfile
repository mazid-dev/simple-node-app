pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mazidhossain/simmple-node-app'
        DOCKER_CRED = credentials('docker-hub-cred')
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/mazid-dev/simple-node-app.git', 
                    branch: 'main', 
                    credentialsId: 'github-token'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "kubectl set image deployment/simple-app-deployment simple-app=${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "kubectl rollout status deployment/simple-app-deployment"
                }
            }
        }
    }
    post {
        success {
            echo "🎉 Deployment successful! Access your app via minikube service simple-app-service"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}