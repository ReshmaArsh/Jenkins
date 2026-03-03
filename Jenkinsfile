pipeline {
    agent any

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

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gowthamb8799-maker/thin07.git'
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'npm install'
                }
            }
        }

        stage('Install and Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    echo "===== Deploying Backend ====="

                    sudo rm -rf ${DEPLOY_PATH}/${BACKEND_DIR}
                    sudo cp -r ${BACKEND_DIR} ${DEPLOY_PATH}

                    sudo chown -R ec2-user:ec2-user ${DEPLOY_PATH}/${BACKEND_DIR}

                    sudo -u ec2-user bash -c "cd ${DEPLOY_PATH}/${BACKEND_DIR} && npm install --production"

                    sudo systemctl restart backend
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                    echo "===== Deploying Frontend ====="

                    if [ -d "frontend/build" ]; then
                        OUTPUT_DIR="frontend/build"
                    elif [ -d "frontend/dist" ]; then
                        OUTPUT_DIR="frontend/dist"
                    else
                        echo "No build or dist folder found!"
                        exit 1
                    fi

                    sudo mkdir -p ${NGINX_PATH}
                    sudo rm -rf ${NGINX_PATH}/*
                    sudo cp -r $OUTPUT_DIR/* ${NGINX_PATH}
                    sudo chown -R nginx:nginx ${NGINX_PATH}

                    sudo systemctl restart nginx
                '''
            }
        }

        stage('Verify Services') {
            steps {
                sh '''
                    echo "Backend Status:"
                    sudo systemctl is-active backend

                    echo "Nginx Status:"
                    sudo systemctl is-active nginx
                '''
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
