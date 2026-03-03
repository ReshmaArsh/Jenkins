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
}pipeline {
    agent any

    environment {
        FRONTEND_DIR = "frontend"
        BACKEND_DIR  = "backend"
        DEPLOY_PATH  = "/home/ec2-user"
        NGINX_PATH   = "/var/www/html"
        SERVICE_NAME = "backend"
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

        stage('Install and build Frontend Dependencies') {
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

                    # Ensure backend folder exists in workspace
                    if [ ! -d "${BACKEND_DIR}" ]; then
                        echo "Backend folder missing in workspace!"
                        exit 1
                    fi

                    # Remove old backend (file or directory)
                    sudo rm -rf ${DEPLOY_PATH}/${BACKEND_DIR}

                    # Copy full backend folder
                    sudo cp -r ${BACKEND_DIR} ${DEPLOY_PATH}

                    # Fix ownership
                    sudo chown -R ec2-user:ec2-user ${DEPLOY_PATH}/${BACKEND_DIR}

                    # Install production dependencies as ec2-user
                    sudo -u ec2-user bash -c "cd ${DEPLOY_PATH}/${BACKEND_DIR} && npm install --production"

                    # Restart backend service
                    sudo systemctl restart backend
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh '''
                    echo "===== Deploying Frontend ====="

                    echo "Checking frontend folder contents:"
                    ls -la frontend

                    # Determine output directory
                    if [ -d "frontend/build" ]; then
                        OUTPUT_DIR="frontend/build"
                    elif [ -d "frontend/dist" ]; then
                          OUTPUT_DIR="frontend/dist"
                    else
                    echo "No build or dist folder found!"
                    exit 1
                    fi

                    echo "Using output directory: $OUTPUT_DIR"

		    # Ensure nginx path exists and is a directory
            	    if [ ! -d "${NGINX_PATH}" ]; then
                 	echo "Nginx path not found. Creating ${NGINX_PATH}"
                	sudo mkdir -p ${NGINX_PATH}
            	    fi

            	    # Clear nginx html
                    sudo rm -rf ${NGINX_PATH}/*

                    # Copy frontend files
                    sudo cp -r $OUTPUT_DIR/* ${NGINX_PATH}

                    # Fix permissions
                    sudo chown -R nginx:nginx ${NGINX_PATH}

                    # Restart nginx
                    sudo systemctl restart nginx
               '''
            }
        }

        stage('Verify Services') {
            steps {
                sh '''
                    echo "===== Backend Status ====="
                    sudo systemctl is-active backend

                    echo "===== Nginx Status ====="
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
