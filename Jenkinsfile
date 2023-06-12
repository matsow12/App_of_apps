def frontendDockerTag =""
def backendDockerTag =""
def frontendImage = "matsow12/frontend"
def backendImage ="matsow12/backend"
def dockerRegistry =""
def registryCredentials ="docker_hub"

pipeline{
    agent{
        label 'agent'
    }
    tools{
        terraform 'Terraform'
    }
    stages{
        stage('Get Code') {
            steps{
                checkout scm
            }
        }

        stage('Clean Containers') {
            steps{
                sh "docker rm -f frontend backend"
            }
        }

        stage('Adjust version') {
            steps{
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    // currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }  
            }
        }

        stage('Deploy application') {
            steps{
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag","BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        docker.withRegistry("$dockerRegistry","$registryCredentials"){
                                sh "docker-compose up -d "
                        }
                    }
                    
                }
            }
        }


    }
}