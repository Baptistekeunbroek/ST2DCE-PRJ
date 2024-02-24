def dockerImage = ''
pipeline {

    agent {
        label 'jenkins-slave'
      }

    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Baptistekeunbroek/ST2DCE-PRJ.git'
            }
        }
        
        stage('Get environment variables') {
            steps {
                script {
                    pom = readMavenPom file: 'pom.xml'
                    echo "Project ArtifactID: ${pom.getArtifactId()}"
                    env.ARTIFACT_ID = pom.getArtifactId()
                    echo "Project Version: ${env.BUILD_ID}"
                    pom.version = env.BUILD_ID
                    writeMavenPom file: 'pom.xml', model: pom
                }
            }
        }
    
        stage('Building docker image') {
          steps {
            script {
                dockerImage = docker.build("hbjprojectdevops/${env.ARTIFACT_ID}:${env.BUILD_ID}", "--build-arg=\"VARIABLE=${env.BUILD_ID}\" .")
            }
          }
        }
        
        stage('Publish docker Image') {
          steps {
            script {
              withDockerRegistry(credentialsId: 'DockerHubHBJ') {
                dockerImage.push()
              }
            }
          }
        }
    
        stage('Update Deployment YAML') {
            steps {
                dir('kubernetes') {
                    script {
                        def yamlDevFile = readFile('deployment-development.yaml')
                        def updatedDevYaml = yamlDevFile.replaceAll(/image: hbjprojectdevops\/st2dce:VERSION_PLACEHOLDER/, "image: hbjprojectdevops/st2dce:${BUILD_ID}")
                        writeFile(file: 'deployment-development.yaml', text: updatedDevYaml)
                        def yamlProdFile = readFile('deployment-production.yaml')
                        def updatedProdYaml = yamlProdFile.replaceAll(/image: hbjprojectdevops\/st2dce:VERSION_PLACEHOLDER/, "image: hbjprojectdevops/st2dce:${BUILD_ID}")
                        writeFile(file: 'deployment-production.yaml', text: updatedProdYaml)
                    }
                }
            }
        }
        
        stage('Deploy to Development') {
            steps {
                dir('kubernetes') {
                    bat "kubectl apply -f deployment-development.yaml"
                }
            }
        }
        
        stage('Test development') {
            steps {
                script{
                    // Execute kubectl port-forward in the background
                    bat "start /B cmd /c kubectl port-forward -n development service/myapp-development 8081:80"
                    bat "curl http://localhost:8081/"
                }
            }
        }
        
        stage('Deploy to Production') {
            steps {
                dir('kubernetes') {
                    bat "kubectl apply -f deployment-production.yaml"
                }
            }
        }
    }
}
