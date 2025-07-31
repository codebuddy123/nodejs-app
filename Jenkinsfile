pipeline {
    agent any 
    
    tools {
        nodejs 'NodeJS-20.19.4'
    }
    environment {
        NODEJS_DEPLOYMENT_SERVER_USER = "ec2-user"
        NODEJS_DEPLOYMENT_SERVER_IP = "172.31.38.104"
        NODEJS_DEPLOYMENT_SERVER_SSH_KEY = "nodejscred" // Ensure this credential ID matches your Jenkins credentials
        NODEJS_DEPLOYMENT_REMOTE_PATH = "/home/ec2-user/nodejs-app/"
    }
    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', credentialsId: 'ce4402e7-050c-46c0-b476-9b9e33e8db44', url: 'https://github.com/codebuddy123/nodejs-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Transfer the files into Deployment Server') {
            steps {
                sshagent([env.NODEJS_DEPLOYMENT_SERVER_SSH_KEY]) {
                    
                    // Ensure the remote directory exists
                    sh "ssh -o StrictHostKeyChecking=no  $NODEJS_DEPLOYMENT_SERVER_USER@$NODEJS_DEPLOYMENT_SERVER_IP mkdir -p ${env.NODEJS_DEPLOYMENT_REMOTE_PATH}"
                    // Use rsync to transfer files to the deployment server
                    sh "rsync -avz --exclude='.git' ./ $NODEJS_DEPLOYMENT_SERVER_USER@$NODEJS_DEPLOYMENT_SERVER_IP:$NODEJS_DEPLOYMENT_REMOTE_PATH"  


                }  
            }
        }
        
        stage('Deploy to Deployment Server') {
            steps {
                sshagent([env.NODEJS_DEPLOYMENT_SERVER_SSH_KEY]) {
                    // Connect to the remote server and run deployment commands
                     
                    sh '''                        
                        ssh -o StrictHostKeyChecking=no ${env.NODEJS_DEPLOYMENT_SERVER_USER}@${env.NODEJS_DEPLOYMENT_SERVER_IP}
                        cd ${env.NODEJS_DEPLOYMENT_REMOTE_PATH} &&
                        npx pm2 start app.js --name my-app --update-env || npx pm2 restart my-app
                       
                       '''
                }
                echo 'Application Deployed...'
            }
        }
    }
}
