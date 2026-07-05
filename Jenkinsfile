pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'pratik1012'
        IMAGE_NAME      = 'my-web-app'
        IMAGE_TAG       = "${BUILD_NUMBER}" 
        GITHUB_REPO     = 'https://github.com/PratikChavare10/my-web-app1.git'
        
        // १. kubeconfig फाईलचा अचूक पाथ इथे डिफाईन केला आहे
        KUBECONFIG      = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Fetch Code') {
            steps {
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
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // २. Groovy string interpolation ची सिक्युरिटी वॉर्निंग घालवण्यासाठी सिंगल कोट्स ('') वापरले आहेत
                    sh 'echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin'
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy/Update Minikube') {
            steps {
                script {
                    echo "Updating Minikube Deployment..."
                    
                    // ३. इथे स्पष्टपणे --kubeconfig चा वापर करून अचूक फाईल सिलेक्ट केली आहे
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml"
                    sh "kubectl --kubeconfig=${KUBECONFIG} set image deployment/web-app-deployment httpd-container=${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local build images..."
            // ४. 'latest' इमेज देखील क्लीन करणे गरजेचे आहे जेणेकरून सर्व्हरची डिस्क स्पेस भरणार नाही
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest || true"
            sh "docker logout"
        }
    }
}
