pipeline {

    agent { label 'built-in' }

    environment {
        FRONTEND_DIR = "frontend"
        BACKEND_DIR  = "backend"
        DEPLOY_PATH  = "/home/ec2-user"
        NGINX_PATH   = "/usr/share/nginx/html"
        SERVICE_NAME = "backend"
    }

    stages {

        // -----------------------
        // CHECKOUT
        // -----------------------
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main',
                url: 'https://github.com/ReshmaArsh/Jenkins.git'
            }
        }

        // -----------------------
        // BUILD
        // -----------------------
        stage('Build') {
            steps {

                echo "Building Backend..."
                dir("${BACKEND_DIR}") {
                    sh "pwd"
                    sh "npm install"
                }

                echo "Building Frontend..."
                dir("${FRONTEND_DIR}") {
                    sh "pwd"
                    sh "npm install"
                    sh "npm run build"
                }

            }
        }

        // -----------------------
        // DEPLOY
        // -----------------------
        stage('Deploy') {
            steps {

                sh '''
                echo "Workspace location:"
                pwd
                ls -l

                echo "Checking backend exists:"
                ls -l backend

                echo "Removing old backend"
                sudo rm -rf /home/ec2-user/backend

                echo "Copying backend"
                sudo cp -r backend /home/ec2-user/

                echo "Verifying copy"
                ls -l /home/ec2-user/

                echo "Entering backend folder"
                cd /home/ec2-user/backend

                echo "Installing production dependencies"
                sudo npm install --production

                echo "Restart backend service"
                sudo systemctl restart backend


                echo "Deploying frontend"
                sudo rm -rf /usr/share/nginx/html/*

                sudo cp -r frontend/build/* /usr/share/nginx/html/

                echo "Restart nginx"
                sudo systemctl restart nginx

                echo "Deployment completed successfully"
                '''
            }
        }

    }

    post {

        success {
            echo "SUCCESS: Deployment completed"
        }

        failure {
            echo "FAILURE: Deployment failed"
        }

    }

}