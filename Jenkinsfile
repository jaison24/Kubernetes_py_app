pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "my-fastapi-app"  // Your image name
        DOCKER_REGISTRY = "docker.io"   // If using Docker Hub, or specify your custom registry
        K8S_DEPLOYMENT = "fastapi-deployment"
        K8S_SERVICE = "fastapi-service"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out the code from Git repository'
                git credentialsId: 'Jaison_PAT', url: 'https://github.com/jaison24/Kubernetes_py_app.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image for FastAPI app'
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        // Optional: Uncomment this section if you want to push the Docker image to Docker Hub
        // stage('Push Docker Image to Registry') {
        //     steps {
        //         echo 'Pushing Docker image to Docker registry'
        //         script {
        //             docker.withRegistry("https://${DOCKER_REGISTRY}", 'jaison-docker-creds') {
        //                 docker.image("${DOCKER_IMAGE}:latest").push()
        //             }
        //         }
        //     }
        // }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes cluster'
                script {
                    // Access the kubeconfig file from Jenkins secrets and store it in a temporary location
                    withCredentials([file(credentialsId: 'jenkins-kubeconfig', variable: 'KUBECONFIG')]) {
                        // Copy the kubeconfig to a temporary file path in the workspace
                        sh 'cp $KUBECONFIG /tmp/kubeconfig'
                        
                        // Apply the Kubernetes deployment using the kubeconfig
                        sh 'kubectl apply -f deploy.yaml --kubeconfig /tmp/kubeconfig'
                    }
                }
            }
        }

        stage('Test Deployment') {
            steps {
                echo 'Testing FastAPI deployment'
                script {
                    // Optional: Run tests or curl to check if FastAPI is up
                    sh 'curl http://localhost:8000'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
    }
}
