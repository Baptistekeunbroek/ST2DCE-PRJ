pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Build') {
            steps {
                
                dir('.') {
                    sh 'mvn clean package'
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    // Utilisez une valeur par défaut pour VARIABLE si elle n'est pas définie
                    def variable = env.VARIABLE ?: 'default_value'
                    docker.build("bkdockerefrei/st2dce:${env.BUILD_ID}").withBuildArg("VARIABLE=${variable}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("bkdockerefrei/st2dce:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (Development)') {
            steps {
                sh 'kubectl apply -f kubernetes/deployment-development.yaml'
            }
        }

        stage('Deploy to Kubernetes (Production)') {
            steps {
                sh 'kubectl apply -f kubernetes/deployment-production.yaml'
            }
        }
    }
}
