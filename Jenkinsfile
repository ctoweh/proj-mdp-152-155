pipeline {
    agent any

    environment {
        ANSIBLE_INVENTORY = '/home/centos/hosts.ini'
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: 'project-1']], 
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: 'https://github.com/ctoweh/proj-mdp-152-155.git']]])
            }
        }

        stage('Setup Build Environment') {
            steps {
                ansiblePlaybook(
                    inventory: "${env.ANSIBLE_INVENTORY}",
                    playbook: 'build.yml'
                )
            }
        }

        stage('Build') {
            steps {
                sshagent(['centos']) {
                    sh 'cd proj-mdp-152-155 && mvn clean install'
                    sh 'scp target/WebAppCal.war centos@13.40.166.40:/usr/local/tomcat9/webapps/'
                }
            }
        }

        stage('Setup Deploy Environment') {
            steps {
                ansiblePlaybook(
                    inventory: "${env.ANSIBLE_INVENTORY}",
                    playbook: 'deploy.yml'
                )
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['centos']) {
                    sh 'ssh centos@13.40.166.40 "sudo /usr/local/tomcat9/bin/shutdown.sh && sudo /usr/local/tomcat9/bin/startup.sh"'
                }
            }
        }
    }
}
