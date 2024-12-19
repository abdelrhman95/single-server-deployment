pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Setup') {
            steps {
                // Install Python dependencies in a virtual environment
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    source venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Package code') {
            steps {
                // Exclude unnecessary files while creating the ZIP
                sh '''
                    zip -r myapp.zip ./* -x "*.git*" "venv/*"
                    ls -lart
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                        # Copy the ZIP file to the production server
                        scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ubuntu/

                        # Connect to the server and deploy the application
                        ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                            # Unzip the package
                            unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app/
                            
                            # Activate virtual environment and install dependencies
                            python3 -m venv /home/ubuntu/app/venv
                            source /home/ubuntu/app/venv/bin/activate
                            pip install --upgrade pip
                            pip install -r /home/ubuntu/app/requirements.txt

                            # Restart the application service
                            sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
    }
}

