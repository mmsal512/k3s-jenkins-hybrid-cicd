// ============================================================================
// 🚀 Hybrid CI/CD Pipeline: Jenkins (K3s) ➜ Docker Host
// ============================================================================
// Architecture: Jenkins orchestrates inside K3s, builds & deploys via SSH
//               on the Docker Host (DooD pattern).
// Trigger:      Automatically on push to 'main' branch (via smee.io webhook).
// ============================================================================

pipeline {
    agent any

    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    environment {
        // ──────────────────────────────────────────────────────────
        // [تعديل] إعدادات الصورة والمستودعات
        // Image & Repository Settings
        // ──────────────────────────────────────────────────────────
        DOCKER_IMAGE    = 'your-dockerhub-username/your-app-name'
        GIT_REPO        = 'https://github.com/your-username/your-repo.git'
        GIT_BRANCH      = 'main'

        // ──────────────────────────────────────────────────────────
        // [تعديل] معرّفات بيانات الاعتماد المحفوظة في Jenkins
        // Jenkins Credentials IDs
        // ──────────────────────────────────────────────────────────
        DOCKER_CREDS    = 'docker-hub-creds'
        SERVER_CREDS    = 'server-ssh-key'

        // ──────────────────────────────────────────────────────────
        // [تعديل] بيانات الاتصال بالسيرفر المضيف (Host)
        // Docker Host Connection Details
        // ──────────────────────────────────────────────────────────
        SERVER_USER     = 'your_server_user'
        SERVER_IP       = 'your_server_ip'       // Tailscale IP or Public IP
        SSH_PORT        = '22'                    // Your SSH port

        // ──────────────────────────────────────────────────────────
        // [تعديل] مسارات المجلدات على السيرفر
        // Server Directory Paths
        // ──────────────────────────────────────────────────────────
        BUILD_DIR       = '/home/your_server_user/jenkins_build_workspace'
        LIVE_DIR        = '/home/your_server_user/your-app-stack'
        APP_SERVICE     = 'your-app-service-name' // Service name in docker-compose.yml

        // ──────────────────────────────────────────────────────────
        // [تعديل] Health Check (فحص صحة التطبيق بعد النشر)
        // Post-Deploy Health Check Settings
        // ──────────────────────────────────────────────────────────
        HEALTH_URL      = 'https://your-domain.com'
        HEALTH_RETRIES  = '6'
        HEALTH_INTERVAL = '10'

        // ──────────────────────────────────────────────────────────
        // [تعديل] إشعارات التلجرام
        // Telegram Notifications
        // ──────────────────────────────────────────────────────────
        TELEGRAM_TOKEN  = credentials('telegram-bot-token') // Store token as Secret Text in Jenkins
        TELEGRAM_CHAT   = 'YOUR_TELEGRAM_CHAT_ID'
    }

    stages {

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 1 — التحقق المبدئي (Pre-flight Checks)              ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Pre-flight Checks') {
            steps {
                script {
                    env.DEPLOY_START = sh(script: 'date +%s', returnStdout: true).trim()
                    currentBuild.description = "🔨 #${BUILD_NUMBER} → ${GIT_BRANCH}"
                }
                sshagent([SERVER_CREDS]) {
                    sh """
                        set -e
                        ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'PREFLIGHT'
                        set -e
                        echo "══════════════════════════════════════════"
                        echo "🔍 Pre-flight Checks"
                        echo "══════════════════════════════════════════"

                        if ! command -v docker &>/dev/null; then
                            echo "❌ Docker is not installed!"
                            exit 1
                        fi
                        echo "✅ Docker: \$(docker --version)"

                        if ! docker compose version &>/dev/null; then
                            echo "❌ Docker Compose plugin is not available!"
                            exit 1
                        fi
                        echo "✅ Docker Compose: \$(docker compose version --short)"

                        if ! command -v git &>/dev/null; then
                            echo "❌ Git is not installed!"
                            exit 1
                        fi
                        echo "✅ Git: \$(git --version)"

                        AVAILABLE_KB=\$(df / | tail -1 | awk '{print \$4}')
                        AVAILABLE_GB=\$(echo "scale=1; \$AVAILABLE_KB / 1048576" | bc)
                        echo "💾 Disk Available: \${AVAILABLE_GB} GB"
                        if [ "\$AVAILABLE_KB" -lt 2097152 ]; then
                            echo "⚠️  Warning: Low disk space!"
                        fi

                        echo "══════════════════════════════════════════"
                        echo "✅ All pre-flight checks passed!"
                        echo "══════════════════════════════════════════"
PREFLIGHT
                    """
                }
            }
        }

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 2 — تحضير الكود المصدري على السيرفر                 ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Prepare Source Code') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-auth',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sshagent([SERVER_CREDS]) {
                        sh """
                            set -e
                            ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'PREPARE'
                            set -e
                            echo "══════════════════════════════════════════"
                            echo "📂 Preparing Source Code..."
                            echo "══════════════════════════════════════════"

                            mkdir -p ${BUILD_DIR}
                            cd ${BUILD_DIR}

                            AUTH_REPO="https://${GIT_USER}:${GIT_TOKEN}@github.com/your-username/your-repo.git"

                            if [ -d .git ]; then
                                echo "🔄 Repository exists — pulling latest changes..."
                                git remote set-url origin "\$AUTH_REPO"
                                git fetch --all --prune
                                git reset --hard origin/${GIT_BRANCH}
                                git clean -fd
                            else
                                echo "⬇️  Cloning repository..."
                                git clone --branch ${GIT_BRANCH} --single-branch "\$AUTH_REPO" .
                            fi

                            COMMIT_SHA=\$(git rev-parse --short HEAD)
                            COMMIT_MSG=\$(git log -1 --pretty=%s)
                            echo "══════════════════════════════════════════"
                            echo "✅ Source ready | Commit: \${COMMIT_SHA}"
                            echo "📝 Message: \${COMMIT_MSG}"
                            echo "══════════════════════════════════════════"
PREPARE
                        """
                    }
                }
            }
        }

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 3 — بناء ورفع صورة Docker                           ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sshagent([SERVER_CREDS]) {
                        sh """
                            set -e
                            ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'BUILDPUSH'
                            set -e
                            cd ${BUILD_DIR}

                            echo "══════════════════════════════════════════"
                            echo "🐳 Docker Build & Push"
                            echo "══════════════════════════════════════════"

                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            echo "✅ Logged into Docker Hub"

                            GIT_SHA=\$(git rev-parse --short HEAD)

                            echo "🔨 Building image..."
                            export DOCKER_BUILDKIT=1
                            docker build \\
                                --label "org.opencontainers.image.revision=\${GIT_SHA}" \\
                                --label "org.opencontainers.image.created=\$(date -u +%Y-%m-%dT%H:%M:%SZ)" \\
                                -t ${DOCKER_IMAGE}:latest \\
                                -t ${DOCKER_IMAGE}:build-${BUILD_NUMBER} \\
                                -t ${DOCKER_IMAGE}:\${GIT_SHA} \\
                                .
                            echo "✅ Image built successfully"

                            echo "🚀 Pushing images to registry..."
                            docker push ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:build-${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:\${GIT_SHA}
                            echo "✅ All tags pushed"

                            docker logout
                            echo "✅ Docker logout complete"

                            echo "══════════════════════════════════════════"
BUILDPUSH
                        """
                    }
                }
            }
        }

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 4 — النشر المباشر عبر Docker Compose                ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Deploy to Production') {
            steps {
                sshagent([SERVER_CREDS]) {
                    sh """
                        set -e
                        ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'DEPLOY'
                        set -e
                        echo "══════════════════════════════════════════"
                        echo "🚀 Deploying to Production..."
                        echo "══════════════════════════════════════════"

                        cd ${LIVE_DIR}

                        OLD_IMAGE=\$(docker compose images ${APP_SERVICE} --format json 2>/dev/null | head -1 | grep -oP '"Tag":"\\K[^"]+' || echo "none")
                        echo "📌 Current image tag: \${OLD_IMAGE}"

                        echo "⬇️  Pulling latest image..."
                        docker compose pull ${APP_SERVICE}

                        echo "♻️  Recreating service..."
                        docker compose up -d --no-deps --force-recreate ${APP_SERVICE}

                        echo "⏳ Waiting for container to start..."
                        sleep 5

                        CONTAINER_STATUS=\$(docker compose ps ${APP_SERVICE} --format '{{.Status}}' 2>/dev/null || echo "unknown")
                        if echo "\${CONTAINER_STATUS}" | grep -qi "up"; then
                            echo "✅ Container is running: \${CONTAINER_STATUS}"
                        else
                            echo "❌ Container is NOT healthy: \${CONTAINER_STATUS}"
                            echo "📋 Logs:"
                            docker compose logs --tail=30 ${APP_SERVICE}
                            exit 1
                        fi

                        echo "══════════════════════════════════════════"
                        echo "✅ Deployment Complete!"
                        echo "══════════════════════════════════════════"
DEPLOY
                    """
                }
            }
        }

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 5 — فحص صحة التطبيق (Health Check)                  ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Health Check') {
            steps {
                sshagent([SERVER_CREDS]) {
                    sh """
                        set -e
                        ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'HEALTH'
                        set -e
                        echo "══════════════════════════════════════════"
                        echo "🏥 Running Health Check..."
                        echo "══════════════════════════════════════════"

                        RETRIES=${HEALTH_RETRIES}
                        INTERVAL=${HEALTH_INTERVAL}
                        URL="${HEALTH_URL}"
                        SUCCESS=0

                        for i in \$(seq 1 \$RETRIES); do
                            HTTP_CODE=\$(curl -s -o /dev/null -w "%{http_code}" "\$URL" 2>/dev/null || echo "000")
                            if [ "\$HTTP_CODE" -ge 200 ] && [ "\$HTTP_CODE" -lt 400 ]; then
                                echo "✅ Health check passed! (HTTP \$HTTP_CODE) — Attempt \$i/\$RETRIES"
                                SUCCESS=1
                                break
                            fi
                            echo "⏳ Attempt \$i/\$RETRIES — HTTP \$HTTP_CODE — retrying in \${INTERVAL}s..."
                            sleep \$INTERVAL
                        done

                        if [ "\$SUCCESS" -eq 0 ]; then
                            echo "❌ Health check failed after \$RETRIES attempts!"
                            echo "📋 Container logs:"
                            cd ${LIVE_DIR}
                            docker compose logs --tail=50 ${APP_SERVICE}
                            exit 1
                        fi
HEALTH
                    """
                }
            }
        }

        // ╔════════════════════════════════════════════════════════════╗
        // ║  Stage 6 — تنظيف الموارد (Cleanup)                       ║
        // ╚════════════════════════════════════════════════════════════╝
        stage('Cleanup') {
            steps {
                sshagent([SERVER_CREDS]) {
                    sh """
                        set -e
                        ssh -o StrictHostKeyChecking=no -p ${SSH_PORT} ${SERVER_USER}@${SERVER_IP} << 'CLEANUP'
                        set -e
                        echo "══════════════════════════════════════════"
                        echo "🧹 Cleaning up resources..."
                        echo "══════════════════════════════════════════"

                        docker image prune -f
                        echo "✅ Dangling images removed"

                        docker builder prune --filter "until=168h" -f 2>/dev/null || true
                        echo "✅ Old build cache pruned"

                        echo "══════════════════════════════════════════"
                        echo "🧹 Cleanup Complete!"
                        echo "══════════════════════════════════════════"
CLEANUP
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                def duration = ''
                try {
                    def start = env.DEPLOY_START?.toLong() ?: 0
                    def end = (sh(script: 'date +%s', returnStdout: true).trim()).toLong()
                    def elapsed = end - start
                    def mins = (elapsed / 60) as int
                    def secs = (elapsed % 60) as int
                    duration = "${mins}m ${secs}s"
                } catch (e) {
                    duration = 'N/A'
                }

                echo """
╔════════════════════════════════════════════════════════════╗
║  🎉 PIPELINE SUCCEEDED                                     ║
╠════════════════════════════════════════════════════════════╣
║  📦 Image:    ${DOCKER_IMAGE}:build-${BUILD_NUMBER}
║  🌿 Branch:   ${GIT_BRANCH}
║  🔢 Build:    #${BUILD_NUMBER}
║  ⏱️  Duration: ${duration}
╚════════════════════════════════════════════════════════════╝
"""
                currentBuild.description = "✅ #${BUILD_NUMBER} deployed (${duration})"

                sh """
                    curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" -d chat_id="${TELEGRAM_CHAT}" -d parse_mode="Markdown" -d text="✅ *Deploy Succeeded*%0A📦 ${DOCKER_IMAGE}:build-${BUILD_NUMBER}%0A⏱️ Duration: ${duration}"
                """
            }
        }

        failure {
            script {
                echo """
╔════════════════════════════════════════════════════════════╗
║  ❌ PIPELINE FAILED                                        ║
╠════════════════════════════════════════════════════════════╣
║  🌿 Branch:   ${GIT_BRANCH}
║  🔢 Build:    #${BUILD_NUMBER}
║  📋 Logs:     ${env.BUILD_URL}console
╚════════════════════════════════════════════════════════════╝
"""
                currentBuild.description = "❌ #${BUILD_NUMBER} failed"

                sh """
                    curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" -d chat_id="${TELEGRAM_CHAT}" -d parse_mode="Markdown" -d text="❌ *Deploy FAILED*%0A🔢 Build: #${BUILD_NUMBER}%0A📋 [View Logs](${env.BUILD_URL}console)"
                """
            }
        }

        always {
            cleanWs()
        }
    }
}