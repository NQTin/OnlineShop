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

        stage('Build Frontend') {
            steps {
                script {
                    echo "Building Frontend..."
                    dir(env.FRONTEND_DIR) {
                        sh 'npm install'
                        sh 'npm start'
                    }
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                script {
                    echo "Deploy Frontend..."
                    dir(env.FRONTEND_DIR) {
                        sh "pm2 start 'serve -s build' --name onlineshop-frontend"
                    }
                }
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    echo 'Building Backend...'
                    dir(env.BACKEND_DIR) {
                        sh "dotnet restore"
                        sh "dotnet build --configuration Release "
                    }
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                script {
                    echo "Deploy Backend"
                    dir(env.BACKEND_DIR) {
                        sh " nohup dotnet run > backend-log 2>&1 &"
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
