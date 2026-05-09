# 🚀 Insure-Me DevSecOps CI/CD Pipeline Project

## 📌 Project Overview

This project demonstrates a complete end-to-end DevSecOps CI/CD pipeline implementation using:

* Jenkins
* SonarQube
* Docker
* Trivy
* AWS EC2
* AWS S3
* DockerHub
* Maven
* GitHub

The pipeline automates:

* Source Code Pull
* Build Automation
* Static Code Analysis
* Quality Gate Validation
* Artifact Storage
* Docker Image Build
* DockerHub Push
* Vulnerability Scanning
* Container Deployment

---

# 🏗️ Project Architecture

```text
GitHub Repository
        ↓
Jenkins Pipeline
        ↓
Maven Build
        ↓
SonarQube Analysis
        ↓
Quality Gate Validation
        ↓
AWS S3 Artifact Upload
        ↓
Docker Image Build
        ↓
DockerHub Push
        ↓
Trivy Vulnerability Scan
        ↓
Docker Container Deployment
        ↓
AWS EC2 Hosted Application
```

---

# ⚙️ Tech Stack

| Tool      | Purpose                |
| --------- | ---------------------- |
| Jenkins   | CI/CD Automation       |
| SonarQube | Static Code Analysis   |
| Docker    | Containerization       |
| Trivy     | Vulnerability Scanning |
| AWS EC2   | Cloud Hosting          |
| AWS S3    | Artifact Storage       |
| DockerHub | Docker Image Registry  |
| Maven     | Build Tool             |
| GitHub    | Source Code Management |

---

# ☁️ AWS EC2 Configuration

A larger EC2 instance was selected to support Jenkins, SonarQube, Docker, and Trivy simultaneously.

## Instance Configuration

| Configuration | Value          |
| ------------- | -------------- |
| Instance Type | m7i-flex.large |
| vCPU          | 2              |
| RAM           | 8 GB           |
| Storage       | 20 GB          |
| OS            | Ubuntu         |

## Why Larger Instance Was Required?

SonarQube and Jenkins consume significant memory.

Using 8 GB RAM avoided:

* Jenkins crashes
* SonarQube container failures
* Docker memory issues
* Pipeline slowdowns

---

# 🔐 Security Group Configuration

The following inbound ports were enabled:

| Port | Purpose                |
| ---- | ---------------------- |
| 22   | SSH Access             |
| 8080 | Jenkins                |
| 9000 | SonarQube              |
| 8089 | Application Deployment |

---

# 📸 Screenshot Usage Guide

Add screenshots for every major setup step using the following markdown format:

```markdown
![Screenshot Description](screenshots/file-name.png)
```

Store all screenshots inside:

```text
screenshots/
```

---

# 🖥️ Jenkins Installation

![Jenkins Installation](screenshots/jenkins-installation.png)

## Install Java

![Java Installation](screenshots/java-installation.png)

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

## Install Jenkins LTS

![Jenkins LTS Installation](screenshots/jenkins-lts-installation.png)

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins
```

## Access Jenkins

![Access Jenkins](screenshots/access-jenkins.png)

```text
http://<EC2-PUBLIC-IP>:8080
```

## Get Jenkins Initial Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 🐳 Docker Installation

![Docker Installation](screenshots/docker-installation.png)

```bash
sudo apt install docker.io -y
sudo systemctl start docker
```

## Give Jenkins Docker Access

![Docker Permission for Jenkins](screenshots/docker-jenkins-access.png)

```bash
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

## Restart Services

```bash
sudo systemctl restart docker
sudo systemctl restart jenkins
```

## Verify Docker

```bash
docker --version
docker ps
```

---

# 🔍 SonarQube Setup

![SonarQube Container](screenshots/sonarqube-container.png)

## Run SonarQube Container

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## Access SonarQube

![Access SonarQube](screenshots/access-sonarqube.png)

```text
http://<EC2-PUBLIC-IP>:9000
```

## Default Credentials

| Username | Password |
| -------- | -------- |
| admin    | admin    |

After login, change the default password.

---

# 📦 Install AWS CLI

![AWS CLI Installation](screenshots/aws-cli-installation.png)

```bash
sudo snap install aws-cli --classic
```

---

# 🔐 Install Jenkins Plugins

![Jenkins Plugins](screenshots/plugins-installation.png)

Installed the following plugins:

* Stage View
* Maven Integration
* SonarQube Scanner
* AWS Credentials
* S3 Publisher
* Docker

---

# 🛠️ Configure Jenkins Tools

![Jenkins Tools Configuration](screenshots/jenkins-tools.png)

Navigate:

```text
Dashboard → Manage Jenkins → Tools
```

Configured:

* Maven
* SonarQube Scanner
* Docker

---

# 🔑 Configure Jenkins Credentials

![Jenkins Credentials](screenshots/credentials-setup.png)

Navigate:

```text
Dashboard → Manage Jenkins → Credentials
```

Added:

## DockerHub Credentials

| Field | Value                  |
| ----- | ---------------------- |
| Type  | Username with Password |
| ID    | docker-cred            |

## AWS Credentials

| Field | Value           |
| ----- | --------------- |
| Type  | AWS Credentials |
| ID    | aws-cred        |

## SonarQube Token

| Field | Value       |
| ----- | ----------- |
| Type  | Secret Text |
| ID    | sonar-cred  |

---

# 🔗 Configure SonarQube in Jenkins

![SonarQube Jenkins Integration](screenshots/sonar-configuration.png)

Navigate:

```text
Dashboard → Manage Jenkins → System
```

## SonarQube Installation

| Field | Value                       |
| ----- | --------------------------- |
| Name  | sonar-server                |
| URL   | http://<EC2-PUBLIC-IP>:9000 |
| Token | sonar-cred                  |

---

# 🔔 Configure SonarQube Webhook

![SonarQube Webhook](screenshots/webhook-setup.png)

Navigate:

```text
SonarQube → Administration → Configuration → Webhooks
```

## Webhook URL

```text
http://<EC2-PUBLIC-IP>:8080/sonarqube-webhook/
```

---

# ☁️ Create AWS S3 Bucket

![AWS S3 Bucket](screenshots/s3-bucket.png)

An S3 bucket was created for storing build artifacts.

## Example Bucket

```text
grambalti
```

---

# 🛡️ Install Trivy

![Trivy Installation](screenshots/trivy-installation.png)

## Install Dependencies

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
```

## Add Trivy GPG Key

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

## Add Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb generic main" | \
sudo tee -a /etc/apt/sources.list.d/trivy.list
```

## Install Trivy

```bash
sudo apt-get update
sudo apt-get install trivy -y
```

## Verify Installation

```bash
trivy --version
```

---

# 🔨 Create Jenkins Pipeline Job

![Pipeline Job Creation](screenshots/pipeline-job.png)

Navigate:

```text
Dashboard → New Item
```

## Configure Pipeline

| Field    | Value    |
| -------- | -------- |
| Job Name | job1     |
| Type     | Pipeline |

---

# 📄 Jenkins Pipeline (Jenkinsfile)

![Pipeline Code](screenshots/pipeline-code.png)

```groovy
pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "bucket-name"
        REGION = "ap-south-1"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
        DOCKER_IMAGE = "dockerhubusername/insureme"
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

        stage("code-test") {
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

        stage("code-test-quality gate") {
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
                sh 'docker build -t dockerhubusername/insureme .'
            }
        }

        stage('image-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push dockerhubusername/insureme'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image ${DOCKER_IMAGE}:latest > trivy-report.txt
                '''
            }
        }

        stage('code-deploy') {
            steps {
                sh 'docker run -itd --name insure-me -p 8089:8081 dockerhubusername/insureme'
            }
        }
    }
}
```

---

# 🚀 Build Pipeline

![Successful Pipeline Execution](screenshots/successful-pipeline.png)

Click:

```text
Build Now
```

The pipeline automatically:

* builds the project
* scans the code
* pushes artifacts
* builds Docker image
* scans vulnerabilities
* deploys application

---

# 🌐 Access Deployed Application

![Final Application Deployment](screenshots/deployed-application.png)

```text
http://<EC2-PUBLIC-IP>:8089
```

Example:

```text
http://13.200.5.25:8089
```

---

# ✅ Final Pipeline Result

The complete DevSecOps pipeline successfully implemented:

* Continuous Integration
* Continuous Deployment
* Static Code Analysis
* Security Scanning
* Artifact Management
* Docker Containerization
* Cloud Deployment

---

# 📸 Screenshots

Create a `screenshots/` folder and add:

```text
screenshots/
│
├── ec2-instance.png
├── jenkins-installation.png
├── sonarqube-container.png
├── docker-installation.png
├── plugins-installation.png
├── credentials-setup.png
├── sonar-configuration.png
├── webhook-setup.png
├── s3-bucket.png
├── trivy-installation.png
├── pipeline-job.png
├── successful-pipeline.png
└── deployed-application.png
```

---

# 📚 Key Learnings

This project helped in understanding:

* CI/CD automation
* DevSecOps practices
* Jenkins pipelines
* Docker containerization
* SonarQube analysis
* Trivy vulnerability scanning
* AWS cloud deployment
* Artifact management
* Pipeline troubleshooting

---

# 👨‍💻 Author

## Rupesh Gorakhnath Sonawane

B.Tech Computer Science Engineering

DevOps | Cloud | CI/CD | AWS | Docker | Jenkins | DevSecOps

---

# ⭐ Conclusion

This project demonstrates a production-style DevSecOps CI/CD pipeline integrating automation, security scanning, containerization, and cloud deployment using modern DevOps tools and AWS infrastructure.
