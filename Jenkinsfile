pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_TAG', description: 'Docker Image Tag (optional)', defaultValue: 'latest')
    }

    environment {
        DOCKER_CREDENTIALS = 'dockerhub-credentials'  // still here if you push later
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.DOCKER_IMAGE_TAG?.trim()) {
                        error("❌ Docker image tag cannot be empty or blank!")
                    }
                    echo "✅ Using Docker tag: ${params.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Roshanx96/django-notes-app.git'
                echo "✅ Repository checked out"
            }
        }

        stage('Update docker-compose.yml with Dynamic Tag') {
            steps {
                script {
                    // Dynamically replace the image tag in the docker-compose.yml
                    sh """
                    sed -i 's/notes-app:.*/notes-app:${params.DOCKER_IMAGE_TAG}/' docker-compose.yml
                    """
                    echo "✅ Docker image tag in docker-compose.yml updated to: ${params.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Run with Docker Compose') {
            steps {
                script {
                    // Stop any existing containers
                    sh 'docker-compose down || true'

                    // Rebuild and run containers
                    sh 'docker-compose up --build -d'

                    echo "🚀 Docker Compose stack is running"
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "💥 Pipeline failed! Check the logs above."
        }
    }
}
