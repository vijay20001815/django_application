pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-note-app"
    }

    stages {

        stage("Build Docker Image") {
            steps {
                echo "🐳 Building Docker image"
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage("Push Image to Docker Hub") {
            steps {
                echo "📤 Pushing image to Docker Hub"
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerHub",
                        usernameVariable: "DOCKER_USER",
                        passwordVariable: "DOCKER_PASS"
                    )
                ]) {
                    sh '''
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:latest
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                echo "☸️ Deploying to Kubernetes"
                withKubeConfig(credentialsId: "kubernetes") {
                    sh """
                        echo "📂 Workspace root"
                        pwd
                        ls -l

                        kubectl get nodes
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl rollout status deployment/django-notes-app
                    """
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning Docker"
            sh "docker system prune -f || true"
        }
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
