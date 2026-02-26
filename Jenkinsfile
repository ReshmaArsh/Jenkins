pipeline {

    agent { label 'built-in' }

    environment {
        FRONTEND_DIR = "frontend"
        BACKEND_DIR  = "backend"

        DEPLOY_PATH  = "/home/ec2-user"
        NGINX_PATH   = "/var/www/html"

        SERVICE_NAME = "backend"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
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
        // BUILD BACKEND
        // -----------------------
        stage('Build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh "npm install"
                }
            }
        }

        // -----------------------
        // BUILD FRONTEND
        // -----------------------
        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh """
                        npm install
                        npm run build
                    """
                }

                sh "ls -l ${WORKSPACE}/${FRONTEND_DIR}/dist"
            }
        }

        // -----------------------
        // DEPLOY BACKEND
        // -----------------------
        stage('Deploy Backend') {
            steps {
                sh """
                    echo "Deploying Backend"

                    sudo rm -rf ${DEPLOY_PATH}/${BACKEND_DIR}
                    sudo mkdir -p ${DEPLOY_PATH}/${BACKEND_DIR}

                    sudo cp -r ${WORKSPACE}/${BACKEND_DIR}/* ${DEPLOY_PATH}/${BACKEND_DIR}/

                    sudo chown -R ec2-user:ec2-user ${DEPLOY_PATH}/${BACKEND_DIR}

                    cd ${DEPLOY_PATH}/${BACKEND_DIR}
                    npm install --production

                    sudo systemctl restart ${SERVICE_NAME}
                """
            }
        }

        // -----------------------
        // DEPLOY FRONTEND
        // -----------------------
        stage('Deploy Frontend') {
            steps {
                sh """
                    echo "Deploying Frontend"

                    sudo rm -rf ${NGINX_PATH}/*
                    sudo cp -r ${WORKSPACE}/${FRONTEND_DIR}/dist/* ${NGINX_PATH}/

                    sudo systemctl restart nginx
                """
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Application deployed"
        }
        failure {
            echo "FAILURE: Check logs"
        }
    }
}