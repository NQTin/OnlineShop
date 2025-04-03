pipeline {
    agent { label 'build-server' }

    environment {
        USER_PROJECTS = 'onlineshop'
        FRONTEND_DIR = 'frontend'
        BACKEND_DIR = 'backend'
        BUILD_TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Clean Old Services') {
            steps {
                script {
                    echo "Stopping and removing old services..."
                    sh "pm2 delete my-frontend || true"
                    sh "pm2 delete my-backend || true"
                    sh "rm -rf frontend/build backend/bin backend/obj"
                }
            }
        }

        stage('Build and Deploy Frontend') {
            steps {
                script {
                    echo "Building Frontend..."
                    dir(env.FRONTEND_DIR) {
                        sh "su jenkins -c 'npm install'"
                        sh "su jenkins -c 'npm start'"
                        sh "pm2 start 'serve -s build' --name onlineshop-frontend"
                    }
                }
            }
        }

        stage('Build and Deploy Backend') {
            steps {
                script {
                    echo 'Building Backend...'
                    dir(env.BACKEND_DIR) {
                        sh 'dotnet restore'
                        sh 'dotnet build --configuration Release'
                        sh 'nohup dotnet run > backend-log 2>&1 &'
                    }
                }
            }
        }
    }
    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
