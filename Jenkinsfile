pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Setup and Install Dependencies') {
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

        stage('Package and Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    // Zip the application files and deploy to the production server
                    sh '''
                    zip -r myapp.zip ./* -x '*.git*' -x 'venv/*'
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/tmp/
                    
                    # Deploy TO prod
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} "unzip -o /tmp/myapp.zip -d /tmp/app/ && \
                        . /tmp/app/venv/bin/activate && \
                        sudo service flaskapp restart"
                    '''
                }
            }
        }
    }
}
