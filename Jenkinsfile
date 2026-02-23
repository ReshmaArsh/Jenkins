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

        // --------------------------------
        // STAGE 1: CHECKOUT
        // --------------------------------
        stage('Checkout') {
            steps {
                echo "Cleaning workspace..."
                cleanWs()

                echo "Cloning GitHub repository..."
                git branch: 'main',
                    url: 'https://github.com/ReshmaArsh/Jenkins.git'
            }
        }

        // --------------------------------
        // STAGE 2: BUILD
        // --------------------------------
        stage('Build') {
            steps {

                echo "Installing Backend Dependencies..."
                dir("${BACKEND_DIR}") {
                    sh 'npm install'
                }

                echo "Installing Frontend Dependencies..."
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                }

                echo "Build completed successfully!"
            }
        }

        // --------------------------------
        // STAGE 3: DEPLOY
        // --------------------------------
        stage('Deploy') {
            steps {

                sh """

                echo "=============================="
                echo "Deploying Backend"
                echo "=============================="

                sudo rm -rf ${DEPLOY_PATH}/backend

                sudo cp -r ${WORKSPACE}/${BACKEND_DIR} ${DEPLOY_PATH}/

                cd ${DEPLOY_PATH}/${BACKEND_DIR}

                sudo npm install --production

                sudo systemctl restart ${SERVICE_NAME}


                echo "=============================="
                echo "Deploying Frontend"
                echo "=============================="

                sudo rm -rf ${NGINX_PATH}/*

                sudo cp -r ${WORKSPACE}/${FRONTEND_DIR}/build/* ${NGINX_PATH}/

                sudo systemctl restart nginx


                echo "=============================="
                echo "Deployment Completed"
                echo "=============================="

                """
            }
        }

    }

    post {

        success {
            echo "✅ SUCCESS: Application deployed successfully!"
        }

        failure {
            echo "❌ ERROR: Deployment failed!"
        }

        always {
            echo "Pipeline finished."
        }

    }

}