# cicd-pipeline-jenkins-trivy-

# 🚀 DevSecOps CI/CD Pipeline Project using Jenkins, SonarQube, Docker, Trivy & AWS

## 📌 Project Overview

This project demonstrates a complete DevSecOps CI/CD pipeline built using:

* Jenkins
* SonarQube
* Docker
* Trivy
* AWS EC2
* Amazon S3
* Maven
* GitHub

The pipeline automatically:

1. Pulls source code from GitHub
2. Builds the Java Spring Boot application using Maven
3. Runs unit tests
4. Performs static code analysis using SonarQube
5. Applies Quality Gate checks
6. Uploads build artifacts to Amazon S3
7. Builds Docker image
8. Pushes image to DockerHub
9. Performs vulnerability scanning using Trivy

---

# 🏗️ Architecture

```text
GitHub → Jenkins → Maven Build → SonarQube Analysis → Quality Gate
       → S3 Upload → Docker Build → DockerHub Push → Trivy Scan
```

---

# 🛠️ Technologies Used

| Tool             | Purpose                          |
| ---------------- | -------------------------------- |
| Jenkins          | CI/CD Automation                 |
| SonarQube        | Code Quality & Security Analysis |
| Docker           | Containerization                 |
| Trivy            | Vulnerability Scanning           |
| AWS EC2          | Hosting Jenkins & SonarQube      |
| AWS S3           | Artifact Storage                 |
| Maven            | Build Tool                       |
| GitHub           | Source Code Management           |
| Java Spring Boot | Application Framework            |

---

# ☁️ AWS Infrastructure Setup

## Step 1: Launch EC2 Instance

### Recommended Instance Type

```text
t3.medium
```

### Security Group Ports

| Port | Purpose     |
| ---- | ----------- |
| 22   | SSH         |
| 8080 | Jenkins     |
| 9000 | SonarQube   |
| 80   | Application |

---

# 🔧 Jenkins Installation

## Install Java

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

## Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

 echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```

## Start Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

## Access Jenkins

```text
http://<EC2-PUBLIC-IP>:8080
```

## Get Jenkins Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 🐳 Docker Installation

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

## Give Jenkins Docker Access

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
```

## Verify Docker

```bash
docker --version
docker ps
```

---

# 🔍 SonarQube Setup

## Pull SonarQube Image

```bash
docker pull sonarqube:lts-community
```

## Run SonarQube Container

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## Verify Container

```bash
docker ps
```

## Access SonarQube

```text
http://<EC2-PUBLIC-IP>:9000
```

## Default Credentials

```text
Username: admin
Password: admin
```

---

# 🔐 SonarQube Token Setup

## Generate Token

Go to:

```text
Administration → Security → Users → Tokens
```

Create token:

```text
jenkins-token
```

Copy the generated token.

---

# ⚙️ Jenkins SonarQube Configuration

## Install Plugins

Install:

* SonarQube Scanner
* Pipeline
* Docker Pipeline
* AWS Credentials

---

## Configure SonarQube Server

Go to:

```text
Manage Jenkins → System → SonarQube servers
```

Add:

| Field | Value                       |
| ----- | --------------------------- |
| Name  | sonar-server                |
| URL   | http://<EC2-PUBLIC-IP>:9000 |
| Token | sonar-cred                  |

---

## Add Sonar Token in Jenkins

Go to:

```text
Manage Jenkins → Credentials
```

Add:

| Field  | Value           |
| ------ | --------------- |
| Kind   | Secret text     |
| ID     | sonar-cred      |
| Secret | SonarQube Token |

---

# 🔎 Install Sonar Scanner

Go to:

```text
Manage Jenkins → Tools
```

Add:

```text
Name: sonar-scanner
```

Enable:

```text
Install Automatically
```

---

# 🛡️ Trivy Installation

## Install Trivy

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb generic main" | \
sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install trivy -y
```

## Verify Trivy

```bash
trivy --version
```

---

# ☁️ AWS S3 Setup

## Create S3 Bucket

Example:

```text
grambalti
```

---

## Add AWS Credentials in Jenkins

Go to:

```text
Manage Jenkins → Credentials
```

Add:

| Field | Value           |
| ----- | --------------- |
| Kind  | AWS Credentials |
| ID    | aws-cred        |

---

# 🐳 DockerHub Setup

## Create DockerHub Credentials

Go to:

```text
Manage Jenkins → Credentials
```

Add:

| Field | Value                  |
| ----- | ---------------------- |
| Kind  | Username with password |
| ID    | docker-cred            |

---

# 📄 Jenkinsfile

```groovy
pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "grambalti"
        REGION = "ap-south-1"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
        DOCKER_IMAGE = "rupesh121203/insureme"
    }

    stages {

        stage('code-pull') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git']])
            }
        }

        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('code-test') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=InsureMe \
                    -Dsonar.projectName=InsureMe \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('code-test-quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }

        stage('code-push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
                }
            }
        }

        stage('docker-image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('image-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push ${DOCKER_IMAGE}:latest'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest | tee trivy-report.txt
                '''
            }
        }
    }
}
```

---

# 📊 Trivy Report

## View Trivy Report on EC2

```bash
cd /var/lib/jenkins/workspace/job1
cat trivy-report.txt
```

---

# 🧠 Problems Faced During Setup

## Issues Solved

* SonarQube URL missing http://
* SonarQube server name mismatch
* Docker permission denied
* SonarQube container stopped
* Public IP changed after EC2 restart
* Trivy image tag mismatch
* Quality Gate waiting issue
* Docker daemon access issue

---

# 📈 Final Outcome

✅ CI/CD Pipeline Successfully Built

✅ SonarQube Security Scanning

✅ Docker Image Build & Push

✅ Trivy Vulnerability Scan

✅ AWS S3 Artifact Upload

✅ End-to-End DevSecOps Workflow

---

# 📸 Recommended Screenshots for GitHub

Add screenshots for:

1. Jenkins Pipeline Success
2. SonarQube Dashboard
3. DockerHub Image
4. Trivy Scan Report
5. AWS EC2 Dashboard
6. S3 Bucket Artifact

---

# 🚀 Future Improvements

* Kubernetes Deployment
* Terraform Infrastructure
* Nginx Reverse Proxy
* HTTPS Setup
* Email Notifications
* Slack Integration
* Monitoring using Prometheus & Grafana
* Blue-Green Deployment
* GitHub Webhooks

---

# 👨‍💻 Author

## Rupesh Sonawane

Aspiring DevOps & Cloud Engineer passionate about:

* DevOps
* Cloud Computing
* CI/CD Automation
* Security Scanning
* Infrastructure Automation

---
