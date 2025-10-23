# 🚀 Jenkins CI/CD Pipeline for Flask Application

This project demonstrates how to set up a complete **Jenkins CI/CD pipeline** for a simple **Python Flask application**, including automated builds, testing, deployment, and email notifications.

---

## 🧩 1. Setup and Install Jenkins on AWS EC2

### Step 1: Launch Virtual Machine
- Launch an **Ubuntu t3.micro** instance on **AWS EC2**.

### Step 2: Install Java
```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version
openjdk version "21.0.3" 2024-04-16
OpenJDK Runtime Environment (build 21.0.3+11-Debian-2)
OpenJDK 64-Bit Server VM (build 21.0.3+11-Debian-2, mixed mode, sharing)

Step 3: Install Jenkins

bash
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
Access Jenkins at:
👉 http://<YOUR-EC2-IP>:8080

Step 4: Install Required Plugins
In Jenkins Dashboard → Manage Jenkins → Manage Plugins → Install:

Email Extension Plugin

Mailer Plugin

🐍 2. Configure Python
Install Python and dependencies on Jenkins Node:

bash
Copy code
sudo apt install python3 python3-venv python3-pip -y
📦 3. Source Code
Fork the sample Flask repository:
👉 Sample Flask App

Clone your forked repository:

bash
Copy code
git clone https://github.com/vishal-user/flask-hello-world.git
cd flask-hello-world

⚙️ 4. Jenkins Pipeline Configuration
Create a Jenkinsfile in your repository root:

groovy
Copy code
pipeline {
  agent any

  environment {
    VENV = 'venv'
    APP_FILE = 'app.py'
    PID_FILE = 'app.pid'
    LOG_FILE = 'app.log'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'master', url: 'https://github.com/vishal-user/flask-hello-world.git'
      }
    }

    stage('Setup Python Environment') {
      steps {
        sh '''
          set -e
          if [ ! -d "${VENV}" ]; then
            python3 -m venv ${VENV}
          fi
          . ${VENV}/bin/activate
          python -m pip install --upgrade pip setuptools wheel
          if [ -f requirements.txt ]; then
            pip install -r requirements.txt
          else
            pip install flask
          fi
        '''
      }
    }

    stage('Run Flask App') {
      steps {
        sh '''
          . ${VENV}/bin/activate
          nohup python ${APP_FILE} > ${LOG_FILE} 2>&1 &
          echo $! > ${PID_FILE}
          echo "Started ${APP_FILE} (PID $(cat ${PID_FILE}))"
        '''
      }
    }
  }

  post {
    always {
      sh '''
        if [ -f ${PID_FILE} ]; then
          kill $(cat ${PID_FILE}) >/dev/null 2>&1 || true
          rm -f ${PID_FILE}
          echo "Stopped flask app"
        fi
      '''
      cleanWs()
    }

    failure {
      echo "Build failed — check ${LOG_FILE} for app output"
    }
  }
}
🔁 5. Configure Automatic Build Triggers
Enable builds on code pushes to main branch:

In Jenkins:

Go to your pipeline job → Configure → Build Triggers

Check ✅ GitHub hook trigger for GITScm polling

In GitHub:

Settings → Webhooks → Add Webhook

Payload URL: http://<your-jenkins-server-ip>:8080/github-webhook/

Content type: application/json

Events: “Just the push event”

Save ✅

Now Jenkins will trigger automatically on every new push.

📧 6. Email Notifications
SMTP Configuration
Go to Manage Jenkins → Configure System → E-mail Notification

Configure:

SMTP Server: smtp.gmail.com

Port: 587

Credentials: Use Gmail App Password

Test email sending to verify.

Email Extension Plugin
Install “Email Extension Plugin”

Add recipients in your pipeline using:

groovy
Copy code
post {
  success {
    emailext to: 'your-email@example.com',
             subject: "Jenkins Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "Good news! Your build completed successfully."
  }
  failure {
    emailext to: 'your-email@example.com',
             subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "The build failed. Check Jenkins logs for details."
  }
}
🧪 7. Run the Pipeline
Commit and push the Jenkinsfile to your GitHub repo.

In Jenkins:

Click New Item → Pipeline

Under Pipeline Definition, select Pipeline script from SCM

Set:

SCM: Git

Repository URL: https://github.com/vishal-user/flask-hello-world.git

Branch: master

Click Build Now

Jenkins will:
Clone the repo

Build the Python environment

Run tests

Deploy Flask app

Send email notifications

✅ 8. Example Output
csharp
Copy code
[Pipeline] Start of Pipeline
[Stage] Checkout
[Stage] Build
[Stage] Test
[Stage] Deploy
[Pipeline] End of Pipeline
Finished: SUCCESS
📂 Folder Structure
Copy code
flask-hello-world/
│
├── app.py
├── requirements.txt
├── tests/
│   └── test_app.py
├── Jenkinsfile
└── README.md
🧰 9. Tools Used
Tool	Purpose
Jenkins	Continuous Integration / Continuous Deployment
Python (Flask)	Web Framework
GitHub	Source Code Management
pytest	Unit Testing
Email Extension Plugin	Build Notifications

👨‍💻 Author
Vishal Dogra
💡 DevOps & Cloud Enthusiast
📧 Email: team@example.com
🌐 GitHub: vishal-user

