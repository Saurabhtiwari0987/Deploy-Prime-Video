pipeline {
    agent any

    tools {
        nodejs 'node16' // Ensure 'node16' is configured in Jenkins under Manage Jenkins > Global Tool Configuration
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Saurabhtiwari0987/Deploy-Prime-Video.git'
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image tagged as 'prime-clone:latest'
                    sh "docker build -t prime-clone:latest ."
                }
            }
        }
        stage ("Trivy Image Scan") {
            steps {
                sh "trivy image saurabhtiwari09876/prime-clone:latest"
            }
        }
        stage('Tag & Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry([ credentialsId: 'dockerhub', url: 'https://app.docker.com/accounts/saurabhtiwari09876' ]) {
                        sh "docker tag prime-clone:latest saurabhtiwari09876/prime-clone:latest"
                        sh "docker push saurabhtiwari09876/prime-clone:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'k8s', variable: 'KUBECONFIG_FILE')]) {
                        sh """
                            # Set the KUBECONFIG environment variable
                            export KUBECONFIG=${KUBECONFIG_FILE}
                            
                            # Navigate to the Kubernetes manifests directory
                            cd Kubernetes
                            
                            # Apply all Kubernetes manifests
                            kubectl apply -f .
                            
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
            // Optional: Add notifications (e.g., email, Slack) here
        }
        failure {
            echo '❌ Deployment failed.'
            // Optional: Add notifications (e.g., email, Slack) here
        }
    }
}
