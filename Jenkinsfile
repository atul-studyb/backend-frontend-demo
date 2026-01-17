pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk"
        MAVEN_HOME = "/usr/share/maven"
        NODE_ENV = "production"

        BACKEND_DIR = "backend"
        FRONTEND_DIR = "frontend"

        JAR_NAME = "backend-app.jar"
        DEPLOY_DIR = "/opt/backend-app"
        NGINX_DIR = "/var/www/react-app"
        SERVICE_NAME = "backend-app"
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
                        npm install
                        npm run build
                    '''
                }
            }
        }

        stage('Deploy React to Nginx') {
            steps {
                sh '''
                    sudo rm -rf ${NGINX_DIR}/*
                    sudo cp -r ${FRONTEND_DIR}/build/* ${NGINX_DIR}/
                    sudo chown -R nginx:nginx ${NGINX_DIR}
                '''
            }
        }

        stage('Deploy Backend JAR') {
            steps {
                sh '''
                    sudo mkdir -p ${DEPLOY_DIR}
                    sudo cp ${BACKEND_DIR}/target/${JAR_NAME} ${DEPLOY_DIR}/
                    sudo chown -R appuser:appuser ${DEPLOY_DIR}
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
