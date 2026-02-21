pipeline {
    agent any
    
    // إعدادات حماية السيرفر من التعليق أو الضغط المضاعف
    options {
        timeout(time: 15, unit: 'MINUTES') 
        disableConcurrentBuilds() 
    }
    
    environment {
        // [تعديل] إعدادات الصورة والمستودعات
        DOCKER_IMAGE = 'your-dockerhub-username/your-app-name'
        GIT_REPO = 'https://github.com/your-username/your-repo.git'
        
        // [تعديل] معرفات بيانات الاعتماد المحفوظة في جينكنز
        DOCKER_CREDS = 'docker-hub-creds'
        SERVER_CREDS = 'server-ssh-key'
        
        // [تعديل] بيانات الاتصال بالسيرفر المضيف (Host)
        SERVER_IP = '100.x.x.x' // يمكن استخدام IP الخاص بـ Tailscale أو السيرفر
        SSH_PORT = '22' // أو البورت المخصص لديك
        
        // [تعديل] مسارات المجلدات في السيرفر
        BUILD_DIR = '/home/user/jenkins_build_workspace'
        LIVE_DIR = '/home/user/your-app-stack'
    }
    
    stages {
        stage('Production Deployment') {
            when {
                // التنفيذ فقط عند التحديث على الفرع الرئيسي
                branch 'main'
            }
            stages {
                stage('Prepare Source Code on Server') {
                    steps {
                        sshagent([SERVER_CREDS]) {
                            sh '''
                                set -e
                                ssh -o StrictHostKeyChecking=no -p $SSH_PORT user@$SERVER_IP << EOF
                                set -e
                                echo "📂 Preparing Workspace..."
                                mkdir -p $BUILD_DIR
                                cd $BUILD_DIR
                                if [ -d .git ]; then
                                    echo "🔄 Pulling latest changes from GitHub..."
                                    git reset --hard
                                    git pull origin main
                                else
                                    echo "⬇️ Cloning repository..."
                                    git clone $GIT_REPO .
                                fi
EOF
                            '''
                        }
                    }
                }
                stage('Build & Push on Server') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: DOCKER_CREDS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sshagent([SERVER_CREDS]) {
                                sh '''
                                    set -e
                                    ssh -o StrictHostKeyChecking=no -p $SSH_PORT user@$SERVER_IP << EOF
                                    set -e
                                    cd $BUILD_DIR
                                    
                                    echo "🐳 Logging into Docker Hub..."
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                    
                                    echo "🔨 Building Docker Image with BuildKit..."
                                    export DOCKER_BUILDKIT=1
                                    docker build -t $DOCKER_IMAGE:latest -t $DOCKER_IMAGE:$BUILD_NUMBER .
                                    
                                    echo "🚀 Pushing to Docker Hub..."
                                    docker push $DOCKER_IMAGE:latest
                                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                                    
                                    echo "👋 Logging out..."
                                    docker logout
EOF
                                '''
                            }
                        }
                    }
                }
                stage('Deploy Live (Docker Compose)') {
                    steps {
                        sshagent([SERVER_CREDS]) {
                            sh '''
                                set -e
                                ssh -o StrictHostKeyChecking=no -p $SSH_PORT user@$SERVER_IP << EOF
                                set -e
                                echo "🚀 Starting Deployment..."
                                cd $LIVE_DIR
                                docker compose pull your-app-service
                                docker compose up -d --no-deps your-app-service
                                docker image prune -f
                                echo "✅ Deployment Successful!"
EOF
                            '''
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '🎉 Pipeline completed successfully! Your app is live.'
        }
        failure {
            echo '❌ Pipeline failed! Please check the logs.'
        }
    }
}
