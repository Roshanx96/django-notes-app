pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to use')
    }

    environment {
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.DOCKER_IMAGE_TAG || !params.DOCKER_IMAGE_TAG.matches("^[a-zA-Z0-9._-]{1,128}$")) {
                        error "❌ Invalid Docker image tag: '${params.DOCKER_IMAGE_TAG}'"
                    } else {
                        echo "✅ Docker image tag '${params.DOCKER_IMAGE_TAG}' is valid"
                    }
                }
            }
        }

        stage('Checkout Repository') {
            steps {
                git url: 'https://github.com/Roshanx96/django-notes-app.git'
                echo "✅ Repository checked out"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t roshanx/django-notes-app:${params.DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built: roshanx/django-notes-app:${params.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push roshanx/django-notes-app:${params.DOCKER_IMAGE_TAG}
                            docker logout
                        """
                        echo "✅ Docker image pushed to Docker Hub"
                    }
                }
            }
        }

        stage('Run the Container') {
            steps {
                script {
                    sh """
                        docker rm -f mycontainer || true
                        docker run -d --name mycontainer -p 8000:8000 roshanx/django-notes-app:${params.DOCKER_IMAGE_TAG}
                    """
                    echo "🚀 Container 'mycontainer' is running on port 8000"
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Build, push, and container run completed successfully!"
        }
        failure {
            echo "💥 Pipeline failed! Check logs for details."
        }
    }
}
