pipeline {
    agent any
    tools {
        go 'go1.25.5'
        nodejs 'node25'
    }
    environment {
        // Project Settings
        BINARY_NAME = 'gitea'
        
        // Remote Server Details
        REMOTE_USER = 'oliren'
        REMOTE_HOST = 'ubuntu22.lan'
        REMOTE_DIR  = '/usr/local/bin/'
        
        // Jenkins Credential ID for SSH Key
        SSH_CREDS_ID = 'ssh-key' 
    }
    stages {
        stage('Build') {
            steps {
                // build binary
                sh 'TAGS="bindata" make build'
            }
        }
        stage('Deploy') {
            steps {
                // Using sshagent to load the SSH key for SCP and SSH commands
                sshagent(credentials: [SSH_CREDS_ID]) {
                    sh """
                        # 1. Secure Copy (SCP) the binary to the remote server
                        scp -o StrictHostKeyChecking=no ${BINARY_NAME} ${REMOTE_USER}@${REMOTE_HOST}:/tmp/${BINARY_NAME}
                        
                        # 2. Make the binary executable and restart service
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'sudo mv /tmp/${BINARY_NAME} ${REMOTE_DIR}/${BINARY_NAME} && chmod +x ${REMOTE_DIR}/${BINARY_NAME} && sudo systemctl restart gitea.service'
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace to save disk space
            cleanWs()
        }
        success {
            echo 'Build and Deployment Successful!'
        }
        failure {
            echo 'Pipeline Failed.'
        }
    }
}