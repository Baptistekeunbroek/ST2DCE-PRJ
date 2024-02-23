pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
            // Set a default value for VARIABLE if it's not defined
            def variable = env.VARIABLE ?: 'default_value'
            // Build the Docker image with the specified tag
            docker.build("bkdockerefrei/st2dce:${env.BUILD_ID}")
        }
    }
}
        stage('Push Docker Image') {
            steps {
                script {
                    // Push the built Docker image to the Docker registry
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("bkdockerefrei/st2dce:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (Development)') {
            steps {
                // Apply Kubernetes deployment configuration for development environment
                sh 'kubectl apply -f kubernetes/deployment-development.yaml'
            }
        }

        stage('Deploy to Kubernetes (Production)') {
            steps {
                // Apply Kubernetes deployment configuration for production environment
                sh 'kubectl apply -f kubernetes/deployment-production.yaml'
            }
        }
    }
}
