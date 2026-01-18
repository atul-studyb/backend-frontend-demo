pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"

        BACKEND_DIR = "backend"
        FRONTEND_DIR = "frontend"

        JAR_NAME = "ems-backend-0.0.1-SNAPSHOT.jar"
        DEPLOY_DIR = "/opt/myapp"
        NGINX_DIR = "/var/www/html"
        SERVICE_NAME = "myappbackend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend (Maven)') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend (React)') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                        rm -rf node_modules package-lock.json
                        npm cache clean --force
                        npm install --verbose
                        npm install vite --verbose
                        npm run build
                    '''
                }
            }
        }

        stage('Deploy React to Nginx') {
            steps {
                sh '''
                    sudo rm -rf ${NGINX_DIR}/*
                    sudo cp -r ${FRONTEND_DIR}/dist/* ${NGINX_DIR}/
                '''
            }
        }

        stage('Deploy Backend JAR') {
            steps {
                sh '''
                    sudo mkdir -p ${DEPLOY_DIR}
                    sudo cp ${BACKEND_DIR}/target/${JAR_NAME} ${DEPLOY_DIR}/backend.jar
                '''
            }
        }

        stage('Restart Backend Service') {
            steps {
                sh '''
                    sudo systemctl daemon-reload
                    sudo systemctl restart ${SERVICE_NAME}
                    sudo systemctl status ${SERVICE_NAME} --no-pager
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}
