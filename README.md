# 🚀 SpringBoot CI/CD Pipeline Project

> **DevOps Practice Project** | Day 13–20 | Jenkins + Docker + GitHub Webhook

![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red?logo=jenkins)
![Docker](https://img.shields.io/badge/Docker-Container-blue?logo=docker)
![SpringBoot](https://img.shields.io/badge/SpringBoot-3.2-green?logo=springboot)
![Java](https://img.shields.io/badge/Java-17-orange?logo=java)

---

## 📌 Project Overview

This project demonstrates a **complete CI/CD pipeline** for a Spring Boot REST API using Jenkins and Docker. It covers all the key DevOps topics from Day 13 to Day 20.

### ✅ Topics Covered

| Day | Topic |
|-----|-------|
| Day 13 | Jenkins Installation & Setup |
| Day 14 | Jenkins Jobs, Pipelines & Plugins |
| Day 15 | Declarative Pipeline with Parameters & Environment Variables |
| Day 16 | GitHub Webhook Integration |
| Day 17 | CI/CD for Docker Project |
| Day 18 | Jenkins with Email Notification |
| Day 19 | Jenkins with Slack Notification |
| Day 20 | Complete CI/CD Project Pipeline |

---

## 🏗️ Project Structure

```
springboot-cicd-project/
├── src/
│   ├── main/java/com/devops/app/
│   │   ├── CicdApplication.java          # Main entry point
│   │   ├── controller/
│   │   │   ├── HealthController.java     # Health check API
│   │   │   └── StudentController.java    # Student CRUD API
│   │   ├── service/
│   │   │   └── StudentService.java       # Business logic
│   │   └── model/
│   │       └── Student.java              # Data model
│   └── test/java/com/devops/app/
│       └── CicdApplicationTests.java     # Unit + Integration tests
├── Dockerfile                            # Multi-stage Docker build
├── docker-compose.yml                    # Docker Compose config
├── Jenkinsfile                           # Declarative Jenkins Pipeline
├── pom.xml                               # Maven dependencies
└── README.md
```

---

## 🔧 Tech Stack

- **Backend:** Spring Boot 3.2, Java 17
- **Build Tool:** Maven
- **Containerization:** Docker (Multi-stage build)
- **CI/CD:** Jenkins Declarative Pipeline
- **Notifications:** Email (SMTP) + Slack
- **Webhook:** GitHub → Jenkins auto-trigger

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Application health status |
| GET | `/api/info` | App information |
| GET | `/api/students` | Get all students |
| GET | `/api/students/{id}` | Get student by ID |
| POST | `/api/students` | Create new student |
| DELETE | `/api/students/{id}` | Delete student |
| GET | `/actuator/health` | Spring Actuator health |

---

## 🐳 Docker Commands

```bash
# Build image
docker build -t springboot-cicd-app:latest .

# Run container
docker run -d -p 8080:8080 --name springboot-app springboot-cicd-app:latest

# Using Docker Compose
docker-compose up -d

# Check logs
docker logs springboot-app

# Stop container
docker-compose down
```

---

## ⚙️ Jenkins Pipeline Stages

```
Checkout → Build → Test → Package → Docker Build → Docker Push → Deploy → Clean Up
```

### Pipeline Features:
- ✅ **Parameterized Build** – Choose environment (dev/staging/prod)
- ✅ **GitHub Webhook** – Auto-trigger on `git push`
- ✅ **Automated Testing** – JUnit test reports published
- ✅ **Docker Multi-stage Build** – Optimized image size
- ✅ **Docker Hub Push** – Image versioned with build number
- ✅ **Health Check** – Verify app after deployment
- ✅ **Email Notification** – HTML email on success/failure
- ✅ **Slack Notification** – Real-time build alerts
- ✅ **Workspace Cleanup** – Auto clean after build

---

## 🔌 Jenkins Setup

### Required Plugins:
- Git Plugin
- Pipeline Plugin
- Docker Pipeline Plugin
- Email Extension Plugin
- Slack Notification Plugin

### Required Credentials (Jenkins → Manage Credentials):
| Credential ID | Type | Description |
|--------------|------|-------------|
| `dockerhub-credentials` | Username/Password | Docker Hub login |

### Environment Variables to Update in Jenkinsfile:
```groovy
DOCKER_HUB_USERNAME = 'your-dockerhub-username'   // ← Change this
EMAIL_RECIPIENT = 'your-email@gmail.com'           // ← Change this
SLACK_CHANNEL = '#your-channel'                    // ← Change this
```

---

## 🔗 GitHub Webhook Setup (Day 16)

1. Go to your GitHub repo → **Settings** → **Webhooks**
2. Click **Add webhook**
3. Payload URL: `http://<your-jenkins-ip>:8080/github-webhook/`
4. Content type: `application/json`
5. Select: **Just the push event**
6. In Jenkins job → **Build Triggers** → ✅ **GitHub hook trigger for GITScm polling**

---

## 📧 Email Notification Setup (Day 18)

In Jenkins → **Manage Jenkins** → **Configure System** → **Extended E-mail Notification**:
```
SMTP Server: smtp.gmail.com
SMTP Port: 465
Use SSL: ✅
Credentials: your Gmail App Password
```

---

## 💬 Slack Notification Setup (Day 19)

1. Create Slack App → Add **Incoming Webhooks**
2. Install **Slack Notification Plugin** in Jenkins
3. Jenkins → **Manage Jenkins** → **Configure System** → **Slack**:
   - Workspace: `your-workspace`
   - Credential: Add Slack Bot Token
   - Default Channel: `#devops-notifications`

---

## 🧪 Running Tests Locally

```bash
# Run all tests
mvn test

# Run with report
mvn test surefire-report:report

# View report
open target/site/surefire-report.html
```

---

## 🚀 Quick Start

```bash
# Clone repo
git clone https://github.com/your-username/springboot-cicd-project.git
cd springboot-cicd-project

# Run locally
mvn spring-boot:run

# Test API
curl http://localhost:8080/api/health
curl http://localhost:8080/api/students
```

---

## 👨‍💻 Author

**DevOps Student** | Jenkins + Docker + Spring Boot Practice  
📅 Project: Day 13–20 CI/CD Pipeline Implementation

---

## 📄 License

MIT License - Free to use for learning and practice.
