This project demonstrates how to set up a **Jenkins CI/CD pipeline** to automate the test, build  and deployment of a simple Python hello-flask-world (e.g., Flask).

## 🚀 1. Setup

### 🖥️ Option A: Install Jenkins on a Virtual AWS Machine
1. Launch an Ubuntu  virtual machine (t3.micro).
2. Install java version : openjdk version "21.0.3" 2024-04-16
   sudo apt update
   sudo apt install fontconfig openjdk-21-jre
   java -version
   openjdk version "21.0.3" 2024-04-16
   OpenJDK Runtime Environment (build 21.0.3+11-Debian-2)
   OpenJDK 64-Bit Server VM (build 21.0.3+11-Debian-2, mixed mode, sharing)
 
3. Install Jenkins:
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins -y
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
Access Jenkins at:
👉 http://<3.91.223.187>:8080

Install required plugins:

Email Extension Plugin
Mailer Plugin

Jenkins on AWS EC2

🐍 Configure Python
Install Python and necessary libraries on your Jenkins node:
bash
Copy code
sudo apt install python3 python3-venv python3-pip -y
📦  Source Code
Fork the sample Python Flask repository:
👉 Sample Python Flask App

Clone your forked repository into the Jenkins server:

bash
Copy code
git clone https://github.com/vishal-user/flask-hello-world.git
cd flask-hello-world

⚙️ 4. Jenkins Pipeline Configuration
Create a file named Jenkinsfile in the root of your repository with the following content:

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
          # create venv if missing
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
          # run app in background, save PID and redirect logs
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
        # stop app if pid file exists
        if [ -f ${PID_FILE} ]; then
          kill $(cat ${PID_FILE}) >/dev/null 2>&1 || true
          rm -f ${PID_FILE}
          echo "Stopped flask app"
        fi
      '''
      // Optional: keep workspace or cleanup as needed
      cleanWs()
    }

    failure {
      echo "Build failed — check ${LOG_FILE} for app output"
    }
  }
}

🔁 5. Triggers (Automatic Build on Code Push)
Enable automatic builds on new commits to the master branch.

In Jenkins, open pipeline job → Configure → Build Triggers.

Select GitHub hook trigger for GITScm polling.

In  GitHub repo → Settings → Webhooks:

Payload URL: http://<jenkins-server-ip>:8080/github-webhook/

Content type: application/json

Events: “Just the push event”

Save.

Now Jenkins will automatically trigger builds on every new push to the main branch.

📧 6. Notifications (Email Alerts)
Set up email notifications for build success or failure.

SMTP Configuration:
Go to Manage Jenkins → Configure System → E-mail Notification.

Enter SMTP settings (vddogra96@gmail.com):

SMTP Server: smtp.gmail.com

Port: 587

Credentials: Add Gmail App Password via Jenkins credentials.

Test email sending to ensure setup works.

Email-ext Configuration:
Install the Email Extension Plugin.

Add recipients in your pipeline (emailext to: line).

Jenkins will send emails after each build with results and console logs.

🧪 7. Run the Pipeline
Commit and push the Jenkinsfile to your repository.

In Jenkins:

Create a New Item → Pipeline.

Under Pipeline Definition, select Pipeline script from SCM.

Enter your GitHub repo URL.

Set the branch to main (or master).

Click Build Now.

Jenkins will:

Clone your repository

Build your Python environment

Run tests

Deploy the app (on success)

Send notification emails

✅ 8. Example Output
stages in Jenkins like this:

csharp
Copy code
[Pipeline] Start of Pipeline
[Stage] Checkout
[Stage] Build
[Stage] Test
[Stage] Deploy
[Pipeline] End of Pipeline
Finished: SUCCESS
📂 Folder Structure Example
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
Email Extension Plugin	Notification System

🧑‍💻 Author
Vishal Dogra
💡 DevOps & Cloud Enthusiast
📧 Contact: team@example.com
🌐 GitHub: vishal-user











