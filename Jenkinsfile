pipeline {
    agent any
    environment {
        DOCKER_HUB_USER = "shivaamaroju"
        IMAGE_NAME = "wanderlust"
        REGISTRY_ID = "docker-hub-token" 
        // Add your Jenkins Credentials ID for K8s here
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
        // Ensure the ID matches what you saved in Jenkins Credentials
        withCredentials([file(credentialsId: 'aks-kubeconfig-file', variable: 'KUBECONFIG')]) {
            // Updating the image tag (from your previous log)
            sh "sed -i 's|image:.*|image: shivaamaroju/wanderlust:1|' deployment.yaml"
            
            // Applying the changes
            sh 'kubectl apply -f deployment.yaml --kubeconfig=$KUBECONFIG'
            
            // Optional: Verify rollout
            sh 'kubectl rollout status deployment/wanderlust --kubeconfig=$KUBECONFIG'
        }
    }
}
}
