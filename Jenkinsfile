pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_TAG', description: 'Docker Image Tag', defaultValue: 'latest')
    }

    environment {
        DOCKER_CREDENTIALS = 'dockerhub-credentials' // Jenkins credentials ID
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.DOCKER_IMAGE_TAG?.trim()) {
                        error("❌ Docker image tag cannot be empty or blank!")
                    } else {
                        echo "✅ Valid tag provided: '${params.DOCKER_IMAGE_TAG}'"
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Roshanx96/django-notes-app.git'
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
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push roshanx/django-notes-app:${DOCKER_IMAGE_TAG}
                        docker logout
                    '''
                    echo "✅ Docker image pushed to Docker Hub"
                }
            }
        }

        stage('Run the Container') {
            steps {
                script {
                    sh '''
                        docker rm -f mycontainer || true
                        docker run -d --name mycontainer -p 8000:8000 roshanx/django-notes-app:${DOCKER_IMAGE_TAG}
                    '''
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
