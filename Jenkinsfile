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
                    // Update image tag in deployment.yaml
                    sh "sed -i 's|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:.*|image: ${DOCKER_HUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER}|' deployment.yaml"
                    
                    // Apply manifests
                    sh "kubectl apply -f deployment.yaml --kubeconfig=\$KUBECONFIG"
                    
                    // Wait for rollout
                    
                    // Fetch External IP
                    echo "Waiting for Azure to assign Public IP..."
                    script {
                        def externalIp = ""
                        while(externalIp == "" || externalIp.contains("pending") || externalIp == "null") {
                            externalIp = sh(script: "kubectl get svc wanderlust-service --kubeconfig=\$KUBECONFIG -o jsonpath='{.status.loadBalancer.ingress[0].ip}'", returnStdout: true).trim()
                            if(externalIp == "" || externalIp.contains("pending") || externalIp == "null") {
                                sleep 10
                            }
                        }
                        echo "--- DEPLOYMENT SUCCESSFUL ---"
                        echo "Your Application is live at: http://${externalIp}"
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed. Run 'kubectl describe pod' to check for errors."
        }
    }
}
