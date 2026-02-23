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

                echo "Building Backend"
                dir("${BACKEND_DIR}") {
                    sh "npm install"
                }

                echo "Building Frontend (dist)"
                dir("${FRONTEND_DIR}") {
                    sh "npm install"
                    sh "npm run build"
                }

                echo "Checking dist folder"
                sh "ls -l ${WORKSPACE}/${FRONTEND_DIR}"
            }
        }

        // -----------------------
        // DEPLOY
        // -----------------------
        stage('Deploy') {
            steps {

                sh """
                echo "Deploy Backend"

                sudo rm -rf ${DEPLOY_PATH}/${BACKEND_DIR}

                sudo cp -r ${WORKSPACE}/${BACKEND_DIR} ${DEPLOY_PATH}/

                cd ${DEPLOY_PATH}/${BACKEND_DIR}

                sudo npm install --production

                sudo systemctl restart ${SERVICE_NAME}


                echo "Deploy Frontend (dist)"

                sudo rm -rf ${NGINX_PATH}/*

                sudo cp -r ${WORKSPACE}/${FRONTEND_DIR}/dist/* ${NGINX_PATH}/

                sudo systemctl restart nginx

                echo "Deployment Successful"
                """
            }
        }

    }

    post {

        success {
            echo "SUCCESS: dist built and deployed"
        }

        failure {
            echo "FAILURE: Check build logs"
        }

    }

}