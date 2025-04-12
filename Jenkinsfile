pipeline {
    agent { label 'build-server' }

    environment {
        USER_PROJECTS = 'onlineshop'
        FRONTEND_DIR = 'frontend'
        BACKEND_DIR = 'backend'
        IMAGE_NAME = "tinnqforwork/shoeshop"
        MAX_VERSION = 5 
        BUILD_TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check for code changes') {
            steps {
                script {
                    def changeLog = git diff HEAD~1 HEAD
                    if (changeLog.trim() == "") {
                        echo "No code changes detected. Skipping build."
                        currentBuild.result = 'SUCCESS'
                        skipRemainingStages = true
                    } else {
                        echo "Code changes detected. Proceeding to build."
                    }
                }
            }
        }

        stage('Determine next version') {
            when {
                expression { return !binding.hasVariable('skipRemainingStages') }
            }
            steps {
                script {
                    def existingTags = sh(
                        script: "docker pull ${IMAGE_NAME} || true && docker images --format '{{.Tag}}' ${IMAGE_NAME} | grep '^v' | sort -V",
                        returnStdout: true
                    ).trim().split("\n")

                    def latestVersion = existingTags.collect { it.replace("v", "").toInteger() }.max() ?: 0
                    def nextVersion = latestVersion + 1
                    env.NEXT_TAG = "${nextVersion}"
                    echo "Next version tag: ${env.NEXT_TAG}"
                }
            }
        }
        stage('Build and Deploy Frontend') {
            steps {
                script {
                    echo "Building Frontend..."
                    dir(env.FRONTEND_DIR) {
                        sh "docker build -t ${env.IMAGE_NAME}:${env.NEXT_TAG} ."
                        
                        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        }
                        
                        sh "docker push ${env.IMAGE_NAME}:${env.NEXT_TAG}"
                        sh 'docker logout'
                        
                        sh "docker run -d -p 3000:3000 --name frontend_${env.NEXT_TAG} ${env.IMAGE_NAME}:${env.NEXT_TAG}"
                        // sh 'npm install'
                        // sh 'CI=false npm run build'
                        // sh 'ls -la && ls -la build' // check build tồn tại
                        // sh "pm2 start 'serve -s build -l 3000' --name onlineshop-frontend"
                        // // sh "pm2 start 'serve -s build' --name onlineshop-frontend"
                        // sh "pm2 list"
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
                        // sh 'nohup dotnet run > backend-log 2>&1 &'
                        sh "pm2 start 'dotnet run' --name onlineshop-backend"
                        sh 'pm2 list'
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
