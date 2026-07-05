pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'pratik1012'
        IMAGE_NAME      = 'my-web-app'
        IMAGE_TAG       = "${BUILD_NUMBER}" // Tracks builds cleanly instead of just using 'latest'
        GITHUB_REPO     = 'https://github.com/PratikChavare10/my-web-app1.git'
    }

    stages {
        stage('Fetch Code') {
            steps {
                // Fetches code from your GitHub repository
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                // 'docker-hub-credentials' must match the ID of credentials stored in Jenkins
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy/Update Minikube') {
            steps {
                script {
                    echo "Updating Minikube Deployment..."
                    // Applies manifest changes and forces Kubernetes to pull the newly built tagged image
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl set image deployment/web-app-deployment httpd-container=${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local build images..."
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker logout"
        }
    }
}
