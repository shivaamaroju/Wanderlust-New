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
                withCredentials([file(credentialsId: "${K8S_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    
                    // The sed command worked fine in your logs
                    sh "sed -i 's|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:.*|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER}|' deployment.yaml"
                    
                    // Apply the manifest
                    sh "kubectl apply -f deployment.yaml --kubeconfig=\$KUBECONFIG"
                    
                    // FIXED: Changed "deployment/${IMAGE_NAME}" to "deployment/wanderlust-deployment"
                    // to match your actual K8s resource name
                    sh "kubectl rollout status deployment/wanderlust-deployment --kubeconfig=\$KUBECONFIG"
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
