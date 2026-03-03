<p align="center">
  <img src="https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white" alt="Jenkins">
  <img src="https://img.shields.io/badge/K3s-FFC61C?style=for-the-badge&logo=k3s&logoColor=black" alt="K3s">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram">
  <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub">
  <img src="https://img.shields.io/badge/Shell_Script-121011?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="Bash">
  <img src="https://img.shields.io/badge/SSH-000000?style=for-the-badge&logo=openssh&logoColor=white" alt="SSH">
</p>

<h1 align="center">🚀 Hybrid CI/CD Pipeline</h1>
<h3 align="center">Jenkins (K3s) ➜ Docker Host ➜ Production</h3>

<p align="center">
  <strong>🇬🇧 EN</strong> | Advanced, lightweight CI/CD architecture designed for single cloud servers
  <br/>
  <strong>🇸🇦 AR</strong> | معمارية متقدمة وخفيفة لعمليات التكامل والتسليم المستمر مخصصة للسيرفرات السحابية الفردية
</p>

<p align="center">
  <img src="https://img.shields.io/github/license/mmsal512/k3s-jenkins-hybrid-cicd?style=flat-square&color=blue" alt="License">
  <img src="https://img.shields.io/github/last-commit/mmsal512/k3s-jenkins-hybrid-cicd?style=flat-square&color=green" alt="Last Commit">
  <img src="https://img.shields.io/github/stars/mmsal512/k3s-jenkins-hybrid-cicd?style=flat-square&color=yellow" alt="Stars">
  <img src="https://img.shields.io/github/forks/mmsal512/k3s-jenkins-hybrid-cicd?style=flat-square&color=orange" alt="Forks">
</p>

---

<!-- ══════════════════════════════════════════════════════════════════ -->
<!-- 🇬🇧 ENGLISH SECTION                                              -->
<!-- ══════════════════════════════════════════════════════════════════ -->

# 🇬🇧 English

## 📖 Overview

This repository provides a **production-ready, fully automated CI/CD pipeline** that runs **Jenkins inside K3s (Kubernetes)** while performing all **Docker build and deploy operations on the host machine** via SSH. This hybrid approach combines Kubernetes self-healing capabilities with the simplicity and performance of native Docker operations.

### ✨ Key Highlights

- 🔄 **Fully Automated** — Triggered on every push to `main` via GitHub Webhooks (through smee.io)
- 🐳 **DooD Pattern** — Docker-outside-of-Docker via SSH for efficient builds
- 🔒 **Secure by Design** — All credentials managed through Jenkins Credentials Store
- 🔑 **Private Repo Support** — Authenticated Git cloning via `github-auth` credentials
- 📡 **Real-time Notifications** — Instant Telegram alerts on deploy success/failure
- 🏥 **Health Checks** — Automated post-deploy verification with configurable retries
- 🧹 **Self-cleaning** — Automatic Docker image and build cache cleanup
- ⏱️ **Execution Metrics** — Build duration tracking in notifications

---

## 🏛️ Architecture

```mermaid
flowchart TB
    subgraph INTERNET["☁️ Internet"]
        GH["🐙 GitHub<br/><small>Source Code</small>"]
        SMEE["🔀 smee.io<br/><small>Webhook Proxy</small>"]
        DHR["🐳 Docker Hub<br/><small>Container Registry</small>"]
    end

    subgraph SERVER["🖥️ Cloud Server"]
        subgraph K3S["☸️ K3s Cluster"]
            JK["🔧 Jenkins<br/><small>CI/CD Orchestrator</small>"]
            SC["📡 Smee Client<br/><small>Webhook Receiver</small>"]
        end

        subgraph HOST["🐋 Docker Host"]
            DB["🔨 Docker Build<br/><small>BuildKit + Cache</small>"]
            DC["📦 Docker Compose<br/><small>Application Stack</small>"]
        end
    end

    subgraph NOTIFY["📬 Notifications"]
        TG["✈️ Telegram Bot<br/><small>Deploy Alerts</small>"]
    end

    GH -- "1️⃣ Push Event" --> SMEE
    SMEE -- "2️⃣ Forward Webhook" --> SC
    SC -- "3️⃣ Trigger Pipeline" --> JK
    JK -- "4️⃣ SSH: Clone & Build" --> DB
    DB -- "5️⃣ Push Image" --> DHR
    JK -- "6️⃣ SSH: Pull & Deploy" --> DC
    DHR -- "7️⃣ Pull Image" --> DC
    JK -- "8️⃣ Notify Result" --> TG

    style INTERNET fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style SERVER fill:#0f3460,stroke:#533483,color:#e0e0e0
    style K3S fill:#1a1a4e,stroke:#FFC61C,color:#e0e0e0
    style HOST fill:#1a3a4e,stroke:#2496ED,color:#e0e0e0
    style NOTIFY fill:#1a2e3e,stroke:#26A5E4,color:#e0e0e0
```

### 🔁 Flow Summary

| Step | Action | Component |
|:---:|---|---|
| 1️⃣ | Developer pushes code to `main` branch | GitHub |
| 2️⃣ | GitHub sends webhook to smee.io proxy | smee.io |
| 3️⃣ | Smee client forwards webhook to Jenkins inside K3s | K3s / Jenkins |
| 4️⃣ | Jenkins SSHs into Docker Host, clones repo & builds image | Docker Host |
| 5️⃣ | Built image is pushed to Docker Hub with multiple tags | Docker Hub |
| 6️⃣ | Jenkins SSHs again to pull & deploy via Docker Compose | Docker Host |
| 7️⃣ | Docker Compose pulls the latest image from registry | Docker Hub |
| 8️⃣ | Jenkins sends success/failure notification via Telegram | Telegram Bot |

---

## 🔄 Pipeline Stages

| # | Stage | Description | Tools |
|:---:|:---:|---|---|
| 1 | 🔍 **Pre-flight Checks** | Validates Docker, Git, Docker Compose availability and disk space on the remote server | `ssh`, `docker`, `git`, `df` |
| 2 | 📂 **Prepare Source Code** | Clones or updates the repository on the server using authenticated GitHub credentials | `git clone`, `git fetch`, `git reset` |
| 3 | 🔨 **Build & Push** | Builds Docker image with BuildKit and pushes 3 tags (`latest`, `build-N`, `git-sha`) | `docker build`, `docker push` |
| 4 | 🚀 **Deploy to Production** | Pulls latest image and recreates the container with zero-downtime approach | `docker compose pull`, `docker compose up` |
| 5 | 🏥 **Health Check** | Verifies application health via HTTP with configurable retries (default: 6 × 10s) | `curl` |
| 6 | 🧹 **Cleanup** | Removes dangling images and build cache older than 7 days | `docker image prune`, `docker builder prune` |
| ✈️ | 📬 **Notification** | Sends instant Telegram notification with build result, duration & link to logs | `curl` → Telegram API |

---

## 🏗️ Design Decisions

<details>
<summary><b>1️⃣ Why does Jenkins run inside K3s?</b></summary>

- **Self-healing:** If Jenkins crashes, K3s automatically restarts it
- **Resource management:** K3s enforces resource limits to prevent Jenkins from consuming all server resources
- **Isolation:** Management tools remain isolated from host applications and files
- **Easy updates:** Updating Jenkins is as simple as updating a Kubernetes manifest

</details>

<details>
<summary><b>2️⃣ Why build images on the host instead of inside K3s?</b></summary>

- **K3s & Docker incompatibility:** K3s uses `containerd` and doesn't have a Docker engine
- **Resource overhead:** Building images inside K3s (via Kaniko or ephemeral pods) consumes excessive memory and CPU
- **Solution — DooD via SSH:** Jenkins connects to the host via SSH and uses the installed Docker engine
- **Docker Cache:** Leveraging Docker's native build cache on the host dramatically speeds up builds

</details>

<details>
<summary><b>3️⃣ Why use smee.io?</b></summary>

- The server is behind a private network (VPN/Tailscale) and not publicly accessible
- This prevents GitHub from sending webhooks directly to Jenkins
- **Solution:** [smee.io](https://smee.io/) acts as a webhook relay — it receives events from GitHub, and the smee client pulls them internally to Jenkins without exposing any ports

</details>

<details>
<summary><b>4️⃣ Why Docker Compose for deployment instead of Kubernetes?</b></summary>

- Docker Compose is simpler and more appropriate for single-server applications
- **Easily extensible:** You can switch the deploy stage to use `kubectl apply` if you want to deploy as pods inside K3s
- Lower operational overhead compared to managing Kubernetes deployments for a single app

</details>

<details>
<summary><b>5️⃣ Why authenticated Git cloning (github-auth)?</b></summary>

- Supports **private repositories** without exposing credentials in the Jenkinsfile
- Uses Jenkins `withCredentials` to securely inject GitHub username and token at runtime
- The token is dynamically injected into the clone URL and never persisted on disk

</details>

---

## 🛠️ Prerequisites

| Requirement | Description | Link |
|---|---|---|
| 🖥️ **Cloud Server** | Ubuntu/Debian with Docker installed | [Install Docker](https://docs.docker.com/engine/install/) |
| ☸️ **K3s** | Installed on the server with Jenkins deployed | [Install K3s](https://k3s.io/) |
| 🔀 **smee.io** | Webhook relay channel with smee client running | [smee.io](https://smee.io/) |
| 🐳 **Docker Hub** | Account for pushing images (or any OCI-compatible registry) | [Docker Hub](https://hub.docker.com/) |
| 🔑 **SSH Key** | SSH key for Jenkins to connect to the host machine | — |
| 🐙 **GitHub Token** | Personal Access Token (PAT) with `repo` scope for private repos | [Create Token](https://github.com/settings/tokens) |
| ✈️ **Telegram Bot** | Telegram bot with your Chat ID for notifications | [Create Bot](https://core.telegram.org/bots#how-do-i-create-a-bot) |

---

## ⚙️ Setup Guide

### Step 1: Configure Jenkins Credentials

Navigate to **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

| Type | ID | Description |
|---|---|---|
| `Username with password` | `docker-hub-creds` | Docker Hub credentials (username + password or Access Token) |
| `SSH Username with private key` | `server-ssh-key` | SSH key for host machine access |
| `Username with password` | `github-auth` | GitHub username + Personal Access Token (for private repos) |
| `Secret text` | `telegram-bot-token` | Telegram bot token (stored securely, never exposed in code) |

> [!IMPORTANT]
> **Security Note:** The Telegram bot token is stored as a **Secret Text** in Jenkins Credentials and retrieved at runtime via `credentials('telegram-bot-token')`. It never appears in source code, build logs, or Git history. Jenkins automatically masks it with `****` in console output.

### Step 2: Customize Environment Variables

Open the `Jenkinsfile` and update the `environment` block to match your project:

```groovy
environment {
    // ─── Image & Repository Settings ───
    DOCKER_IMAGE    = 'your-dockerhub-username/your-app-name'
    GIT_REPO        = 'https://github.com/your-username/your-repo.git'
    GIT_BRANCH      = 'main'

    // ─── Jenkins Credentials IDs ───
    DOCKER_CREDS    = 'docker-hub-creds'
    SERVER_CREDS    = 'server-ssh-key'

    // ─── Docker Host Connection ───
    SERVER_USER     = 'your_server_user'
    SERVER_IP       = 'your_server_ip'       // Tailscale IP or Public IP
    SSH_PORT        = '22'                    // Your SSH port

    // ─── Server Directory Paths ───
    BUILD_DIR       = '/home/your_server_user/jenkins_build_workspace'
    LIVE_DIR        = '/home/your_server_user/your-app-stack'
    APP_SERVICE     = 'your-app-service-name'  // Service name in docker-compose.yml

    // ─── Health Check ───
    HEALTH_URL      = 'https://your-domain.com'
    HEALTH_RETRIES  = '6'
    HEALTH_INTERVAL = '10'

    // ─── Telegram Notifications ───
    TELEGRAM_TOKEN  = credentials('telegram-bot-token')
    TELEGRAM_CHAT   = 'YOUR_TELEGRAM_CHAT_ID'
}
```

> [!TIP]
> You also need to update the GitHub repo URL inside the `Prepare Source Code` stage where `AUTH_REPO` is constructed.

### Step 3: Create Pipeline in Jenkins

1. Go to **New Item** → Choose **Multibranch Pipeline** (or **Pipeline**)
2. Under **Branch Sources** → Add your GitHub repository URL
3. Under **Build Configuration** → Select **by Jenkinsfile**
4. Save and let smee.io handle incoming webhook events

### Step 4: Set Up smee.io Webhook Relay

```bash
# 1. Create a new channel at https://smee.io (copy the URL)

# 2. Configure GitHub Webhook:
#    → Repository Settings → Webhooks → Add Webhook
#    → Payload URL: https://smee.io/YOUR-CHANNEL-ID
#    → Content type: application/json
#    → Events: Just the push event

# 3. Run smee client on the server (as a K3s pod or systemd service):
npx smee-client -u https://smee.io/YOUR-CHANNEL-ID \
    -t http://JENKINS-INTERNAL-URL/github-webhook/
```

---

## ✈️ Telegram Notifications

The pipeline sends instant Telegram notifications after every deployment:

| Status | Notification Content |
|---|---|
| ✅ **Success** | Image name + Build number + Execution duration |
| ❌ **Failure** | Build number + Direct link to error logs in Jenkins |

> [!NOTE]
> **Security:** The token is automatically pulled from Jenkins Credentials via `credentials('telegram-bot-token')` and **never appears** in:
> - The `Jenkinsfile` (source code)
> - Build console output — Jenkins replaces it with `****`
> - Git repository history

### How to Get Your Telegram Chat ID

1. Send any message to your bot in Telegram
2. Open this URL in your browser (replace `YOUR_BOT_TOKEN` with your bot's token):
   ```
   https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates
   ```
3. Look for `"chat":{"id":` in the response — that's your Chat ID
4. For groups: Add the bot to the group, send a message, then repeat step 2 (group Chat IDs start with `-`)

---

## 🔒 Security Best Practices

This pipeline follows security best practices for public repositories:

| Practice | Implementation |
|---|---|
| 🔐 **No hardcoded secrets** | All sensitive data stored in Jenkins Credentials |
| 🔑 **Authenticated Git clone** | GitHub PAT injected via `withCredentials` at runtime |
| 🛡️ **SSH key authentication** | Jenkins agent uses SSH keys, never passwords |
| 🙈 **Masked console output** | Jenkins auto-masks credentials in build logs |
| 🔒 **Secure Telegram token** | Stored as `Secret text`, never in source code |
| 🧹 **Docker logout** | Explicit logout after push to prevent credential caching |
| 🗑️ **Workspace cleanup** | `cleanWs()` runs after every build to remove artifacts |

---

## 🔧 Troubleshooting

<details>
<summary><b>❌ SSH Connection Failed</b></summary>

```bash
# Verify the correct key is added in Jenkins Credentials
# Make sure the SSH port matches
ssh -p YOUR_PORT -i /path/to/key user@server-ip

# Ensure the user is in the docker group
sudo usermod -aG docker your-user

# Test SSH connectivity from Jenkins pod
kubectl exec -it <jenkins-pod> -- ssh -p YOUR_PORT user@server-ip
```
</details>

<details>
<summary><b>❌ Docker Build Failed</b></summary>

```bash
# Verify Dockerfile exists in the repository root
ls -la $BUILD_DIR/Dockerfile

# Check available disk space
df -h /

# Clean Docker cache to free space
docker system prune -a --force

# Check Docker daemon status
sudo systemctl status docker
```
</details>

<details>
<summary><b>❌ Health Check Failed</b></summary>

```bash
# Check if the container is running
docker compose ps

# View application logs
docker compose logs --tail=100 your-app-service-name

# Test the URL manually
curl -v https://your-domain.com

# Check if the port is exposed and accessible
ss -tlnp | grep <port>
```
</details>

<details>
<summary><b>❌ smee.io Not Working</b></summary>

```bash
# Ensure smee client is running
# You can run it as a systemd service or inside K3s

# Manual test:
npx smee-client -u https://smee.io/YOUR-CHANNEL \
    -t http://JENKINS-URL/github-webhook/

# Verify webhook delivery in GitHub:
# → Repository Settings → Webhooks → Recent Deliveries
```
</details>

<details>
<summary><b>❌ Telegram Notifications Not Working</b></summary>

```bash
# 1. Verify the token is stored in Jenkins Credentials with ID: telegram-bot-token
# 2. Verify the Chat ID is correct
# 3. Manual test:
curl -s -X POST "https://api.telegram.org/botYOUR_TOKEN/sendMessage" \
    -d chat_id="YOUR_CHAT_ID" \
    -d text="Test message from server"

# 4. For groups: ensure the bot is an Admin
# 5. Verify Jenkins can reach api.telegram.org (not blocked by firewall)
```
</details>

<details>
<summary><b>❌ GitHub Authentication Failed (Private Repos)</b></summary>

```bash
# 1. Verify 'github-auth' credentials exist in Jenkins
#    → Manage Jenkins → Credentials → System → Global
# 2. Ensure the GitHub PAT has 'repo' scope
# 3. Check if the PAT has expired
# 4. Manual test:
git clone https://USERNAME:TOKEN@github.com/your-username/your-repo.git
```
</details>

---

## 📁 Repository Structure

```
k3s-jenkins-hybrid-cicd/
│
├── 📄 Jenkinsfile          # Main CI/CD pipeline definition
├── 📄 README.md            # This documentation file
├── 📄 Dockerfile           # (in your app repo) Docker image build file
└── 📄 docker-compose.yml   # (on your server) Service orchestration file
```

---

## 🧰 Required Jenkins Credentials Summary

| Credential ID | Type | Purpose |
|---|---|---|
| `docker-hub-creds` | Username with password | Docker Hub authentication for push |
| `server-ssh-key` | SSH Username with private key | SSH access to Docker Host |
| `github-auth` | Username with password | GitHub authenticated clone (PAT) |
| `telegram-bot-token` | Secret text | Telegram bot token for notifications |

---

## 🤝 Contributing

Contributions are welcome! If you have ideas to improve this architecture:

1. Open an **Issue** to discuss your idea or report a problem
2. Create a **Fork** then submit a **Pull Request** with a clear description
3. Make sure your changes are tested and working before submitting

---

<br/>

<!-- ══════════════════════════════════════════════════════════════════ -->
<!-- 🇸🇦 ARABIC SECTION                                                -->
<!-- ══════════════════════════════════════════════════════════════════ -->

# 🇸🇦 العربية

## 📖 نظرة عامة

يقدم هذا المستودع حلاً **عملياً ومتكاملاً وجاهزاً للإنتاج** لبناء خط أنابيب CI/CD احترافي على سيرفر سحابي واحد. يعتمد الإعداد على تشغيل **Jenkins** داخل بيئة **K3s (Kubernetes)** للاستفادة من قدرات الاستشفاء الذاتي وإدارة الموارد، بينما تتم عمليات **بناء الصور ونشرها (Build & Deploy)** مباشرة على السيرفر المضيف (Host) عبر **Docker**، مع تخطي عوائق الشبكات الخاصة باستخدام **smee.io** كقناة Webhook آمنة.

### ✨ أبرز المميزات

- 🔄 **أتمتة كاملة** — يعمل تلقائياً عند كل Push إلى فرع `main` عبر GitHub Webhooks (من خلال smee.io)
- 🐳 **نمط DooD** — Docker-outside-of-Docker عبر SSH لبناء فعّال
- 🔒 **آمن بالتصميم** — جميع بيانات الاعتماد تُدار عبر Jenkins Credentials Store
- 🔑 **دعم المستودعات الخاصة** — استنساخ Git مُوثّق عبر بيانات اعتماد `github-auth`
- 📡 **إشعارات فورية** — تنبيهات Telegram فورية عند نجاح أو فشل النشر
- 🏥 **فحوصات صحة** — تحقق آلي بعد النشر مع محاولات قابلة للتخصيص
- 🧹 **تنظيف ذاتي** — تنظيف تلقائي لصور Docker وكاش البناء
- ⏱️ **مقاييس التنفيذ** — تتبع مدة البناء في الإشعارات

---

## 🏛️ المعمارية (Architecture)

```mermaid
flowchart TB
    subgraph INTERNET["☁️ الإنترنت"]
        GH["🐙 GitHub"]
        SMEE["🔀 smee.io<br/><small>وسيط الويب هوك</small>"]
        DHR["🐳 Docker Hub<br/><small>سجل الحاويات</small>"]
    end

    subgraph SERVER["🖥️ السيرفر السحابي"]
        subgraph K3S["☸️ K3s Cluster"]
            JK["🔧 Jenkins<br/><small>منسّق CI/CD</small>"]
            SC["📡 Smee Client<br/><small>مستقبل الويب هوك</small>"]
        end

        subgraph HOST["🐋 Docker Host"]
            DB["🔨 بناء Docker<br/><small>BuildKit + Cache</small>"]
            DC["📦 Docker Compose<br/><small>تطبيقات الإنتاج</small>"]
        end
    end

    subgraph NOTIFY["📬 الإشعارات"]
        TG["✈️ بوت تلجرام<br/><small>تنبيهات النشر</small>"]
    end

    GH -- "1️⃣ Push Event" --> SMEE
    SMEE -- "2️⃣ إعادة توجيه" --> SC
    SC -- "3️⃣ تشغيل Pipeline" --> JK
    JK -- "4️⃣ SSH: استنساخ وبناء" --> DB
    DB -- "5️⃣ رفع الصورة" --> DHR
    JK -- "6️⃣ SSH: سحب ونشر" --> DC
    DHR -- "7️⃣ سحب الصورة" --> DC
    JK -- "8️⃣ إرسال الإشعار" --> TG

    style INTERNET fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style SERVER fill:#0f3460,stroke:#533483,color:#e0e0e0
    style K3S fill:#1a1a4e,stroke:#FFC61C,color:#e0e0e0
    style HOST fill:#1a3a4e,stroke:#2496ED,color:#e0e0e0
    style NOTIFY fill:#1a2e3e,stroke:#26A5E4,color:#e0e0e0
```

---

## 🔄 مراحل الـ Pipeline

| # | المرحلة | الوصف | الأدوات |
|:---:|:---:|---|---|
| 1 | 🔍 **التحقق المبدئي** | التحقق من توفر Docker, Git, Docker Compose ومساحة القرص على السيرفر | `ssh`, `docker`, `git`, `df` |
| 2 | 📂 **تحضير الكود المصدري** | استنساخ أو تحديث المستودع على السيرفر باستخدام بيانات اعتماد GitHub المُوثّقة | `git clone`, `git fetch`, `git reset` |
| 3 | 🔨 **بناء ورفع الصورة** | بناء صورة Docker مع BuildKit ورفعها بـ 3 علامات (`latest`, `build-N`, `git-sha`) | `docker build`, `docker push` |
| 4 | 🚀 **النشر على الإنتاج** | سحب أحدث صورة وإعادة إنشاء الحاوية بدون توقف | `docker compose pull`, `docker compose up` |
| 5 | 🏥 **فحص الصحة** | التحقق من صحة التطبيق عبر HTTP مع إعادة المحاولة (افتراضي: 6 × 10 ثوانٍ) | `curl` |
| 6 | 🧹 **التنظيف** | إزالة الصور المعلقة وكاش البناء الأقدم من 7 أيام | `docker image prune`, `docker builder prune` |
| ✈️ | 📬 **الإشعار** | إرسال إشعار Telegram فوري بنتيجة البناء والمدة ورابط السجلات | `curl` → Telegram API |

---

## 🏗️ القرارات التصميمية

<details>
<summary><b>1️⃣ لماذا يتواجد Jenkins داخل K3s؟</b></summary>

- **الاستشفاء الذاتي (Self-healing):** إذا توقف Jenkins، يقوم K3s بإعادة تشغيله تلقائياً
- **إدارة الموارد:** K3s يضمن عدم استهلاك Jenkins لموارد أكثر مما هو مخصص
- **العزل (Isolation):** بقاء أدوات الإدارة معزولة عن ملفات وتطبيقات السيرفر المضيف
- **التحديثات السهلة:** تحديث Jenkins يتم عبر تحديث Manifest بسيط

</details>

<details>
<summary><b>2️⃣ لماذا نبني الصور على السيرفر وليس داخل K3s؟</b></summary>

- **مشكلة K3s و Docker:** بيئة K3s تعتمد على `containerd` ولا تحتوي على محرك Docker
- **استهلاك الموارد:** بناء الصور داخل K3s (عبر Kaniko أو Pods مؤقتة) يستهلك ذاكرة ومعالج بشكل مفرط
- **الحل — DooD عبر SSH:** Jenkins يتصل بالسيرفر عبر SSH ويستخدم Docker المثبت هناك
- **كاش Docker:** استخدام Docker Cache على السيرفر المضيف يُسرّع البناء بشكل مضاعف

</details>

<details>
<summary><b>3️⃣ لماذا نستخدم smee.io؟</b></summary>

- السيرفر محمي بشبكة خاصة (VPN/Tailscale) وغير متاح للإنترنت العام
- هذا يمنع GitHub من إرسال Webhooks مباشرة إلى Jenkins
- **الحل:** [smee.io](https://smee.io/) يعمل كقناة وسيطة — يستقبل الطلبات من GitHub، ويقوم عميل smee بسحبها وتمريرها داخلياً إلى Jenkins بدون فتح بورتات

</details>

<details>
<summary><b>4️⃣ لماذا Docker Compose وليس Kubernetes للنشر؟</b></summary>

- Docker Compose أسهل وأنسب للتطبيقات على سيرفر واحد
- **قابل للتعديل:** يمكنك بسهولة تحويل مرحلة النشر لاستخدام `kubectl apply` إذا أردت النشر كـ Pods داخل K3s
- عبء تشغيلي أقل مقارنة بإدارة Kubernetes deployments لتطبيق واحد

</details>

<details>
<summary><b>5️⃣ لماذا استنساخ Git المُوثّق (github-auth)؟</b></summary>

- دعم **المستودعات الخاصة** بدون كشف بيانات الاعتماد في ملف Jenkinsfile
- يستخدم `withCredentials` في Jenkins لحقن اسم المستخدم والتوكن بشكل آمن أثناء التشغيل
- التوكن يُحقن ديناميكياً في رابط الاستنساخ ولا يُحفظ أبداً على القرص

</details>

---

## 🛠️ المتطلبات (Prerequisites)

| المتطلب | الوصف | رابط |
|---|---|---|
| 🖥️ **سيرفر سحابي** | Ubuntu/Debian مع Docker مثبت | [Install Docker](https://docs.docker.com/engine/install/) |
| ☸️ **K3s** | مثبت على نفس السيرفر مع Jenkins | [Install K3s](https://k3s.io/) |
| 🔀 **smee.io** | قناة Webhook مع عميل smee مثبت | [smee.io](https://smee.io/) |
| 🐳 **Docker Hub** | حساب لرفع الصور (أو أي Registry متوافق) | [Docker Hub](https://hub.docker.com/) |
| 🔑 **SSH Key** | مفتاح SSH للاتصال بالسيرفر المضيف من Jenkins | — |
| 🐙 **GitHub Token** | توكن وصول شخصي (PAT) بصلاحية `repo` للمستودعات الخاصة | [إنشاء توكن](https://github.com/settings/tokens) |
| ✈️ **Telegram Bot** | بوت Telegram للإشعارات مع Chat ID | [إنشاء بوت](https://core.telegram.org/bots#how-do-i-create-a-bot) |

---

## ⚙️ دليل الإعداد

### الخطوة 1: إضافة بيانات الاعتماد في Jenkins

انتقل إلى **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

| النوع | المعرّف (ID) | الوصف |
|---|---|---|
| `Username with password` | `docker-hub-creds` | بيانات Docker Hub (اسم المستخدم + كلمة المرور أو Access Token) |
| `SSH Username with private key` | `server-ssh-key` | مفتاح SSH للدخول إلى السيرفر المضيف |
| `Username with password` | `github-auth` | اسم مستخدم GitHub + توكن وصول شخصي (PAT) للمستودعات الخاصة |
| `Secret text` | `telegram-bot-token` | توكن بوت Telegram (مُخزّن بأمان، لا يظهر في الكود) |

> [!IMPORTANT]
> **أمان التوكنات:** يتم تخزين توكن بوت Telegram كـ **Secret Text** في Jenkins Credentials ويُسحب أثناء التشغيل عبر `credentials('telegram-bot-token')`. لا يظهر أبداً في الكود المصدري أو سجلات البناء أو تاريخ Git. Jenkins يستبدله تلقائياً بـ `****` في مخرجات وحدة التحكم.

<details>
<summary><b>📋 خطوات إضافة توكن Telegram في Jenkins</b></summary>

1. اذهب إلى **Manage Jenkins → Credentials → System → Global credentials**
2. اضغط على **Add Credentials**
3. اختر **Kind:** `Secret text`
4. في حقل **Secret:** الصق توكن البوت (مثل `123456789:ABCdefGhIJKlmNoPQ...`)
5. في حقل **ID:** اكتب `telegram-bot-token` (يجب أن يتطابق مع ما في Jenkinsfile)
6. في حقل **Description:** اكتب وصفاً مثل `Telegram Bot Token for Deploy Notifications`
7. اضغط **Create**

</details>

<details>
<summary><b>📋 خطوات إضافة بيانات GitHub (github-auth) في Jenkins</b></summary>

1. اذهب إلى **Manage Jenkins → Credentials → System → Global credentials**
2. اضغط على **Add Credentials**
3. اختر **Kind:** `Username with password`
4. في حقل **Username:** اكتب اسم مستخدم GitHub الخاص بك
5. في حقل **Password:** الصق توكن الوصول الشخصي (PAT) — وليس كلمة المرور
6. في حقل **ID:** اكتب `github-auth` (يجب أن يتطابق مع ما في Jenkinsfile)
7. في حقل **Description:** اكتب وصفاً مثل `GitHub PAT for Private Repo Access`
8. اضغط **Create**

> **ملاحظة:** لإنشاء توكن وصول شخصي (PAT):
> 1. اذهب إلى GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
> 2. اضغط **Generate new token**
> 3. اختر صلاحية `repo` (كاملة)
> 4. انسخ التوكن فوراً (لن يظهر مرة أخرى)

</details>

<details>
<summary><b>📋 كيفية الحصول على Chat ID الخاص بالتلجرام</b></summary>

1. أرسل أي رسالة إلى البوت الخاص بك في Telegram
2. افتح الرابط التالي في المتصفح (استبدل `YOUR_BOT_TOKEN` بتوكن البوت):
   ```
   https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates
   ```
3. ابحث عن `"chat":{"id":` في النتيجة — هذا هو الـ Chat ID
4. للمجموعات: أضف البوت إلى المجموعة وأرسل رسالة، ثم كرر الخطوة 2 (Chat ID للمجموعات يبدأ بـ `-`)

</details>

### الخطوة 2: تعديل متغيرات البيئة

افتح ملف `Jenkinsfile` وعدّل قسم `environment` ليناسب مشروعك:

```groovy
environment {
    // ─── إعدادات الصورة والمستودع ───
    DOCKER_IMAGE    = 'your-dockerhub-username/your-app-name'
    GIT_REPO        = 'https://github.com/your-username/your-repo.git'
    GIT_BRANCH      = 'main'

    // ─── بيانات الاعتماد ───
    DOCKER_CREDS    = 'docker-hub-creds'
    SERVER_CREDS    = 'server-ssh-key'

    // ─── بيانات السيرفر ───
    SERVER_USER     = 'your_server_user'
    SERVER_IP       = 'your_server_ip'       // Tailscale IP أو عنوان IP عام
    SSH_PORT        = '22'                    // بورت SSH الخاص بك

    // ─── مسارات المجلدات ───
    BUILD_DIR       = '/home/your_server_user/jenkins_build_workspace'
    LIVE_DIR        = '/home/your_server_user/your-app-stack'
    APP_SERVICE     = 'your-app-service-name'  // اسم الخدمة في docker-compose.yml

    // ─── فحص الصحة ───
    HEALTH_URL      = 'https://your-domain.com'
    HEALTH_RETRIES  = '6'
    HEALTH_INTERVAL = '10'

    // ─── إشعارات Telegram ───
    TELEGRAM_TOKEN  = credentials('telegram-bot-token')
    TELEGRAM_CHAT   = 'YOUR_TELEGRAM_CHAT_ID'
}
```

> [!TIP]
> تحتاج أيضاً إلى تعديل رابط مستودع GitHub داخل مرحلة `Prepare Source Code` حيث يُبنى متغير `AUTH_REPO`.

### الخطوة 3: إنشاء Pipeline في Jenkins

1. اذهب إلى **New Item** → اختر **Multibranch Pipeline** (أو Pipeline)
2. في قسم **Branch Sources** → أضف رابط مستودع GitHub
3. في قسم **Build Configuration** → اختر **by Jenkinsfile**
4. احفظ واترك smee.io يتولى تلقي الأحداث

### الخطوة 4: إعداد smee.io لنقل الويب هوك

```bash
# 1. أنشئ قناة جديدة على https://smee.io (انسخ الرابط)

# 2. اضبط GitHub Webhook:
#    → إعدادات المستودع → Webhooks → Add Webhook
#    → Payload URL: https://smee.io/YOUR-CHANNEL-ID
#    → Content type: application/json
#    → Events: Just the push event

# 3. شغّل عميل smee على السيرفر (كـ K3s pod أو systemd service):
npx smee-client -u https://smee.io/YOUR-CHANNEL-ID \
    -t http://JENKINS-INTERNAL-URL/github-webhook/
```

---

## ✈️ إشعارات Telegram

يقوم الـ Pipeline بإرسال إشعار فوري إلى Telegram عند انتهاء كل عملية نشر:

| الحالة | محتوى الإشعار |
|---|---|
| ✅ **نجاح** | اسم الصورة + رقم البناء + مدة التنفيذ |
| ❌ **فشل** | رقم البناء + رابط مباشر لسجلات الأخطاء في Jenkins |

> **ملاحظة أمنية:** التوكن يُسحب تلقائياً من Jenkins Credentials عبر `credentials('telegram-bot-token')` ولا يظهر أبداً في:
> - ملف `Jenkinsfile` (الكود المصدري)
> - سجلات البناء (Build Console Output) — يستبدله Jenkins بـ `****`
> - مستودع Git

---

## 🔒 ممارسات الأمان

| الممارسة | التطبيق |
|---|---|
| 🔐 **لا أسرار مكتوبة في الكود** | جميع البيانات الحساسة تُخزّن في Jenkins Credentials |
| 🔑 **استنساخ Git مُوثّق** | توكن GitHub يُحقن عبر `withCredentials` أثناء التشغيل |
| 🛡️ **مفاتيح SSH** | Jenkins يستخدم مفاتيح SSH وليس كلمات مرور |
| 🙈 **إخفاء المخرجات** | Jenkins يُخفي بيانات الاعتماد تلقائياً في سجلات البناء |
| 🔒 **توكن Telegram آمن** | يُخزّن كـ `Secret text` ولا يظهر في الكود |
| 🧹 **تسجيل خروج Docker** | تسجيل خروج صريح بعد الرفع لمنع تخزين بيانات الاعتماد مؤقتاً |
| 🗑️ **تنظيف مساحة العمل** | `cleanWs()` يعمل بعد كل بناء لإزالة الملفات المؤقتة |

---

## 🔧 استكشاف الأخطاء وإصلاحها

<details>
<summary><b>❌ فشل الاتصال بـ SSH</b></summary>

```bash
# تأكد من أن المفتاح الصحيح مضاف في Jenkins Credentials
# تأكد من أن البورت صحيح
ssh -p YOUR_PORT -i /path/to/key user@server-ip

# تأكد من أن المستخدم مضاف لمجموعة docker
sudo usermod -aG docker your-user

# اختبر التوصيل من داخل pod Jenkins
kubectl exec -it <jenkins-pod> -- ssh -p YOUR_PORT user@server-ip
```
</details>

<details>
<summary><b>❌ فشل بناء الصورة (Docker Build)</b></summary>

```bash
# تأكد من وجود Dockerfile في المستودع
ls -la $BUILD_DIR/Dockerfile

# تأكد من مساحة القرص
df -h /

# تنظيف Docker cache
docker system prune -a --force

# تحقق من حالة Docker daemon
sudo systemctl status docker
```
</details>

<details>
<summary><b>❌ فشل Health Check</b></summary>

```bash
# تأكد من أن التطبيق يعمل
docker compose ps

# اعرض سجلات التطبيق
docker compose logs --tail=100 your-app-service-name

# اختبر الرابط يدوياً
curl -v https://your-domain.com

# تحقق من أن البورت مفتوح ومتاح
ss -tlnp | grep <port>
```
</details>

<details>
<summary><b>❌ smee.io لا يعمل</b></summary>

```bash
# تأكد من تشغيل عميل smee
# يمكنك تشغيله كـ systemd service أو داخل K3s

# اختبار يدوي:
npx smee-client -u https://smee.io/YOUR-CHANNEL \
    -t http://JENKINS-URL/github-webhook/

# تحقق من تسليم الـ webhook في GitHub:
# → إعدادات المستودع → Webhooks → Recent Deliveries
```
</details>

<details>
<summary><b>❌ إشعارات Telegram لا تعمل</b></summary>

```bash
# 1. تأكد من أن التوكن صحيح ومحفوظ في Jenkins Credentials بـ ID: telegram-bot-token
# 2. تأكد من أن الـ Chat ID صحيح
# 3. اختبر يدوياً:
curl -s -X POST "https://api.telegram.org/botYOUR_TOKEN/sendMessage" \
    -d chat_id="YOUR_CHAT_ID" \
    -d text="Test message from server"

# 4. إذا كنت تستخدم مجموعة، تأكد من أن البوت مشرف (Admin) في المجموعة
# 5. تأكد من أن Jenkins يمكنه الوصول إلى api.telegram.org (لا يحظره الفايروول)
```
</details>

<details>
<summary><b>❌ فشل التوثيق مع GitHub (المستودعات الخاصة)</b></summary>

```bash
# 1. تحقق من وجود بيانات اعتماد 'github-auth' في Jenkins
#    → Manage Jenkins → Credentials → System → Global
# 2. تأكد من أن توكن GitHub لديه صلاحية 'repo'
# 3. تحقق من عدم انتهاء صلاحية التوكن
# 4. اختبار يدوي:
git clone https://USERNAME:TOKEN@github.com/your-username/your-repo.git
```
</details>

---

## 📁 هيكل المستودع

```
k3s-jenkins-hybrid-cicd/
│
├── 📄 Jenkinsfile          # خط الأنابيب الرئيسي (CI/CD Pipeline)
├── 📄 README.md            # هذا الملف التوثيقي
├── 📄 Dockerfile           # (في مستودع تطبيقك) ملف بناء الصورة
└── 📄 docker-compose.yml   # (على السيرفر) ملف تشغيل الخدمات
```

---

## 🧰 ملخص بيانات الاعتماد المطلوبة في Jenkins

| معرّف بيانات الاعتماد | النوع | الغرض |
|---|---|---|
| `docker-hub-creds` | Username with password | توثيق Docker Hub للرفع |
| `server-ssh-key` | SSH Username with private key | الوصول عبر SSH إلى السيرفر المضيف |
| `github-auth` | Username with password | استنساخ Git المُوثّق (PAT) |
| `telegram-bot-token` | Secret text | توكن بوت Telegram للإشعارات |

---

## 🤝 المساهمة

المساهمات مرحب بها! إذا كانت لديك أفكار لتحسين هذه المعمارية:

1. افتح **Issue** لطرح فكرتك أو مشكلتك
2. أنشئ **Fork** ثم **Pull Request** مع وصف واضح
3. تأكد من أن التغييرات تعمل بشكل صحيح قبل الإرسال

---

## 📜 License / الترخيص

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT) — free to use in your projects.

هذا المشروع مرخص تحت رخصة [MIT](https://opensource.org/licenses/MIT) — استخدمه بحرية في مشاريعك.

---

<p align="center">
  <sub>Built with ❤️ for the best performance and highest security on single servers</sub>
  <br/>
  <sub>تم بناء هذه المعمارية بـ ❤️ لتوفير أفضل أداء وأعلى مستوى أمان للسيرفرات الفردية</sub>
  <br/><br/>
  <sub>💡 <em>If you find this useful, don't forget to ⭐ Star the repository!</em></sub>
  <br/>
  <sub>💡 <em>إذا أعجبك المستودع، لا تنسَ ⭐ النجمة!</em></sub>
</p>
