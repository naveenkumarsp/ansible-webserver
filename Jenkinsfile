pipeline{
    agent { label "agentfarm" }
    stages {
        stage('Delete the workspace') {
            steps{
                cleanWs()
            }
        }
        stage('Install Ansible') {
            steps{
                script{
                    def ansible_exists = fileExists '/usr/bin/ansible'
                    if (ansible_exists == true){
                        echo "Skipping ansible installation as it already exists"
                    } else {
                        sh 'sudo apt-get update -y && sudo apt-get upgrade -y'
                        sh 'sudo apt-get install -y wget tree unzip ansible python3-pip python3-apt'
                    }
                }
            }
        }
        stage('Third stage') {
            steps{
                echo "Third Stage"
            }
        }
    } 
}