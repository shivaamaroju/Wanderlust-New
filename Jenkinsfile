pipeline {
    agent any
    environment {
        DOCKER_HUB_USER = "shivaamaroju"
        IMAGE_NAME = "wanderlust"
        REGISTRY_ID = "docker-hub-token" 
        K8S_CREDENTIAL_ID = "aks-kubeconfig-file" 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_ID}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                // Using withCredentials (file) because withKubeConfig method was missing/failing
                withCredentials([file(credentialsId: "${K8S_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    
                    // Dynamically update the deployment.yaml with the current build number
                    sh "sed -i 's|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:.*|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER}|' deployment.yaml"
                    
                    // Apply the updated manifest
                    sh "kubectl apply -f deployment.yaml --kubeconfig=\$KUBECONFIG"
                    
                    // Verify the rollout was successful
                    sh "kubectl rollout status deployment/${IMAGE_NAME} --kubeconfig=\$KUBECONFIG"
                }
            }
        }
    }

    post {
        success {
            echo "Successfully built, pushed, and deployed ${IMAGE_NAME}:${env.BUILD_NUMBER} to AKS!"
        }
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
    }
}
