pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Build') {
            steps {
                // Use 'dir' to navigate to the project directory before running Maven commands
                dir('.') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Set a default value for VARIABLE if it's not defined
                    def variable = env.VARIABLE ?: 'default_value'
                    // Build the Docker image with the specified tag and build argument
                    docker.build("bkdockerefrei/ST2DCE-PRJ:${env.BUILD_ID}").withBuildArg("VARIABLE=${variable}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the built Docker image to the Docker registry
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("bkdockerefrei/ST2DCE-PRJ:${env.BUILD_ID}").push()
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
