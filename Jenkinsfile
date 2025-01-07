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

        stage('Push Docker Image to Registry') {
            steps {
                echo 'Pushing Docker image to Docker registry'
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'jaison-docker-creds') {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes cluster'
                script {
                    sh '''
                    kubectl apply -f - <<EOF
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${K8S_DEPLOYMENT}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: fastapi-app
                      template:
                        metadata:
                          labels:
                            app: fastapi-app
                        spec:
                          containers:
                          - name: fastapi-container
                            image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            ports:
                            - containerPort: 8000
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${K8S_SERVICE}
                    spec:
                      selector:
                        app: fastapi-app
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 8000
                      type: ClusterIP
                    EOF
                    '''
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
