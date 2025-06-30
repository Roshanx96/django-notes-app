pipeline {
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Roshanx96/django-notes-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t notes-app:latest .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                sh '''
                    docker tag notes-app roshanx/notes-app:latest
                    docker push roshanx/notes-app:latest
                '''
            }
        }

        stage('Run the Container') {
            steps {
                sh '''
                    docker stop mycontainer || true
                    docker rm -f mycontainer || true
                    docker run --name mycontainer -d -p 8000:8000 roshanx/notes-app:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!!!"
        }
    }
}
