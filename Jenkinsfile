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
                    def changeLog = sh(script: "git diff HEAD~1 HEAD", returnStdout: true)
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
                    def tagsJson = sh(
                        script: "curl -s https://registry.hub.docker.com/v2/repositories/${env.IMAGE_NAME}/tags?page_size=100",
                        returnStdout: true
                    ).trim()
                
                    def tags = new groovy.json.JsonSlurperClassic().parseText(tagsJson).results*.name
                    def versionTags = tags.findAll { it.startsWith("v") }
                
                    def latestVersion = versionTags ?
                        versionTags.collect { it.replace("v", "").toInteger() }.max() :
                        0
                
                    def nextVersion = latestVersion + 1
                    env.NEXT_TAG = "v${nextVersion}"
                
                    echo "Latest version: v${latestVersion}"
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
