pipeline {
    agent none
    environment {
        ANSIBLE_INVENTORY = '/home/centos/hosts.ini'
    }
    stages {
        stage('Clone Repository') {
            agent {
                label 'ansible-master'  
            }
            steps {
                git branch: 'project-1', url: 'https://github.com/ctoweh/proj-mdp-152-155.git'
            }
        }
        stage('Setup Build Environment') {
            agent {
                label 'ansible-master'  
            }
            steps {
                ansiblePlaybook(
                    inventory: "${env.ANSIBLE_INVENTORY}",
                    playbook: 'build.yml'
                )
            }
        }
        stage('Build') {
            agent {
                label 'ansible-master'  
            }
            steps {
                script {
                    // Fetch and add the host key of the deployment server
                    sshagent(credentials: ['jenkins']) {
                        sh '''
                            ssh-keyscan 13.40.131.4 >> ~/.ssh/known_hosts
                            ssh centos@13.40.131.4 "cd proj-mdp-152-155 && mvn clean install"
                            scp target/WebAppCal.war centos@13.40.166.40:/usr/local/tomcat9/webapps/
                        '''
                    }
                }
            }
        }
        stage('Setup Deploy Environment') {
            agent {
                label 'ansible-master'  
            }
            steps {
                ansiblePlaybook(
                    inventory: "${env.ANSIBLE_INVENTORY}",
                    playbook: 'deploy.yml'
                )
            }
        }
        stage('Deploy') {
            agent {
                label 'ansible-master'  
            }
            steps {
                script {
                    // Fetch and add the host key of the deployment server
                    sshagent(credentials: ['jenkins']) {
                        sh '''
                            ssh-keyscan 13.40.166.40 >> ~/.ssh/known_hosts
                            ssh centos@13.40.166.40 "sudo /usr/local/tomcat9/bin/shutdown.sh && sudo /usr/local/tomcat9/bin/startup.sh"
                        '''
                    }
                }
            }
        }
    }
}
