pipeline{
    agent { label "agentfarm" }
    environment{
        KEY_FILE = '/home/ubuntu/.ssh/cprime-public-16-mar22.pem'
        USER = 'ubuntu'
    }
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
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 apache-install.yml'
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-update.yml'
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-test.yml'
            }
        }
        stage('Send Slack notification') {
            steps {
                slackSend color: 'warning', message: "Mr Deeds: Please approve ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.JOB_URL} | Open>)"
            }
        }
        stage('Request Input') {
            steps {
                input 'Please approve or deny this build'
            }
        }
        stage('Install apache & update website') {
            steps {
                sh 'ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/apache-install.yml'
                sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-playbook/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-update.yml'
            }
        }
        stage('Test website') {
            steps {
                sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-playbook/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-test.yml'
            }
        }
        
    }
    post {
        success {
            slackSend color: 'good',  message: "Build $JOB_NAME $BUILD_NUMBER was successful! :) "
        }
        failure {
            slackSend color: 'danger', message: "Build $JOB_NAME $BUILD_NUMBER Failed :( "
        }
    } 
}