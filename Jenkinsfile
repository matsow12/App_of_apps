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

        stage('Tests'){
            steps{
                script{
                    sh "pip3 install -r test/selenium/requirements.txt"
                    sh "python3 -m pytest test/selenium/test.py"
                }
            }
        }

        stage('Run terraform'){
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/matsow12/Terraform'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init && terraform apply -auto-approve -var-file="terraform.tfvars"'
                    } 
                }
            }
        }

        stage('Run Ansible'){
            steps{
                script{
                    sh "ansible-galaxy install -r requirements.yml"
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", "BACKEND_IMAGE=$backendImage:$backendDockerTag"]){
                        ansiblePlaybook inventory: 'inventory', playbook: 'playbook'
                    }
                }
            }
        }
        

    }
    post{
        always{
            sh "docker-compose down"
            cleanWs()
        }
    }
}