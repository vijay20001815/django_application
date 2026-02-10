pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-note-app"
    }

    stages {

        stage("Clone Repository") {
            steps {
                echo "Cloning GitHub repository"
                git url: "https://github.com/LondheShubham153/django-notes-app.git",
                    branch: "main"
            }
        }

        stage("Build Docker Image") {
            steps {
                echo "Building Docker image"
                sh """
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                """
            }
        }

        stage("Push Image to Docker Hub") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerHub",
                        usernameVariable: "DOCKER_USER",
                        passwordVariable: "DOCKER_PASS"
                    )
                ]) {
                    sh """
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:latest
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage("Deploy to Kubernetes (MicroK8s)") {
            steps {
                dir("notesapp") {
                    withKubeConfig(credentialsId: "kubernetes") {
                        sh """
                            echo "Checking cluster access"
                            kubectl get nodes

                            echo "Deploying application"
                            kubectl apply -f deployment.yaml
                            kubectl apply -f service.yaml

                            echo "Waiting for rollout"
                            kubectl rollout status deployment/django-notes-app
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            sh "docker system prune -f || true"
        }
    }
}
