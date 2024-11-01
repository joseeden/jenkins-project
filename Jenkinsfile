pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage('Setup') {
            steps {
                // Create and activate a virtual environment, then install dependencies
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        stage('Test') {
            steps {
                // Activate the virtual environment and run tests
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*' -x 'venv/*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/opt/
                    
                    # Deploy to prodserver
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /opt/myapp.zip -d /opt/app/
                        
                        if [ ! -d "/opt/app/venv" ]; then
                            python3 -m venv /opt/app/venv
                        fi
                        
                        . /opt/app/venv/bin/activate
                        pip install -r /opt/app/requirements.txt
                        
                        sudo service flaskapp restart
                    EOF
                    '''
                    
                }
            }
        }
    }
}
