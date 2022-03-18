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
        stage('Download Ansible code') {
            steps{
                git credentialsId: 'git-repo-creds', url: 'https://github.com/naveenkumarsp/ansible-webserver'
            }
        }
        stage('Run ansible lint against playbook') {
            steps{
                sh 'docker run --rm -v $WORKSPACE/playbook:data cytopia/ansible-lint:4 apache-install.yml'
                sh 'docker run --rm -v $WORKSPACE/playbook:data cytopia/ansible-lint:4 website-update.yml'
                sh 'docker run --rm -v $WORKSPACE/playbook:data cytopia/ansible-lint:4 website-test.yml'
            }
        }
    } 
}