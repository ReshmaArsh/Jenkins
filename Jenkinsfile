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

        // -------------------------
        // STAGE 1: CHECKOUT
        // -------------------------
        stage('Checkout') {
            steps {
                cleanWs()

                git branch: 'main',
                    url: 'https://github.com/ReshmaArsh/Jenkins.git'
            }
        }

        // -------------------------
        // STAGE 2: BUILD
        // -------------------------
        stage('Build') {
            steps {

                echo "Building Backend..."
                dir("${BACKEND_DIR}") {
                    sh 'npm install'
                }

                echo "Building Frontend..."
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        // -------------------------
        // STAGE 3: DEPLOY
        // -------------------------
        stage('Deploy') {
            steps {

                sh """
                echo "Deploying Backend..."

                sudo rm -rf ${DEPLOY_PATH}/backend
                sudo cp -r ${BACKEND_DIR} ${DEPLOY_PATH}/

                cd ${DEPLOY_PATH}/backend
                sudo npm install --production

                sudo systemctl restart ${SERVICE_NAME}


                echo "Deploying Frontend..."

                sudo rm -rf ${NGINX_PATH}/*
                sudo cp -r ${FRONTEND_DIR}/build/* ${NGINX_PATH}/

                sudo systemctl restart nginx


                echo "Verifying Services..."

                sudo systemctl status ${SERVICE_NAME} --no-pager
                sudo systemctl status nginx --no-pager
                """
            }
        }

    }

    post {

        success {
            echo "✅ Deployment Successful!"
        }

        failure {
            echo "❌ Deployment Failed!"
        }

    }

}