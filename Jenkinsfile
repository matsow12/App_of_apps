def frontendDockerTag =""
def backendDockerTag =""
def frontendImage = "matsow12/Frontend"
def backendImage ="matsow12/Backend"
def dockerRegistry =""
def registryCredentials ="docker_hub"

pipeline{
    agent{
        label 'agent'
    }
    tools{
        terraform 'Terragorm'
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
                sh "docker rm -f Frontend Backend"
            }
        }

        stage('Adjust version') {
            steps{
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    currentbuild.description = "Backend:${backendDockerTag}, Frontend:${frontendDockerTag}"
                }    
            }
        }

        stage('Deploy application') {
            steps{
                script {
                    withENV(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag","BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        docker.withRegistry("$dockerRegistry","$registryCredentials")
                    }
                    sh "docker-compose up -d "
                }
            }
        }


    }
}