# :rocket: Node.js Application Deployment using Jenkins CI/CD

This project demonstrates a **Jenkins CI/CD pipeline** to deploy a **Node.js application directly on an EC2 instance**  Jenkins pulls the code from GitHub, installs dependencies, and runs the Node.js app using a **Jenkinsfile**.

---

##  **Project Objective**

### To automate Node.js application deployment using:

* Jenkins Pipeline
* Jenkinsfile
* GitHub SCM (Git)
* AWS EC2 (Linux)

### After a successful build, the application is accessible via **EC2 Public IP**.

---

## Tools & Technologies Used

* **Node.js**
* **Jenkins**
* **Git & GitHub**
* **AWS EC2 (Linux)**


---

##  **Step 1: Launch EC2 Instance:-**

* Launch **Amazon Linux / Ubuntu EC2**
* Configure Security Group:

  * **22** – SSH
  * **8080** – Jenkins
  * **3000** – Node.js App

---

##  **Step 2: Install Node.js, Git & Jenkins on EC2**

### Install Node.js

```bash
sudo yum install nodejs npm -y
sudo yum install -g pm2
```

### 1) Use / Purpose:

<b>Installs Node.js → required to run your app.js

Installs npm → required to install project dependencies (npm install)</b>

### Why needed?

### Without this, Jenkins cannot run

---
### 2) PM2 = Production Process Manager for Node.js
---

### Install Git

```bash
sudo yum install git -y
git --version
```

### Install Jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install java-17-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

##  **Step 3: Access Jenkins Dashboard**

Open browser:

```text
http://<EC2-Public-IP>:8080
```

* Unlock Jenkins using initial admin password
* Install **Suggested Plugins**

📸 **Image:** Jenkins Dashboard

```md
![Jenkins Dashboard](images/jenkins-dashboard.png)
```

---

##  **Step 4: Install Required Jenkins Plugins**

Go to:

```text
Manage Jenkins → Plugins
```

### Install:

*  Pipeline
*  Git



##  **Step 5: Create Jenkins Pipeline Job**

1. Click **New Item**
2. Job Name: `node-app-deployment`
3. Select **Pipeline**
4. Click **OK**



---

##  **Step 6: Configure Pipeline (SCM – Git)**

In **Pipeline section**:

* Definition: `Pipeline script from SCM`

* SCM: `Git`

* Repository URL: `https://github.com/your-username/node-app.git`

![Pipeline SCM](/configure.png)

* Branch Specifier: `*/main`
* Script Path: `Jenkinsfile`


---
![Pipeline SCM](/configure-2.png)



##  **Step 7: Jenkinsfile**

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP      = '16.171.255.228'
        SSH_CREDENTIAL = 'node-app-key'
        REPO_URL       = 'https://github.com/techrohitx/node-js-app-CICD.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ec2-user'
        REMOTE_PATH    = '/home/ec2-user/node-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Install Dependencies & Start App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            npm install &&
                            pm2 start app.js --name node-app || pm2 restart node-app
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo ' Application deployed successfully!'
        }
        failure {
            echo ' Deployment failed.'
        }
    }
}


```

---

##  **Step 8: Build the Pipeline**

* Click **Build Now**
* Check **Console Output**
* Ensure all stages are successful

📸 **Image:** Build Success


![Build Success](/build_now.png)


---

##  **Step 9: Access Node.js Application**

Open browser:

```text
http://<EC2-Public-IP>:3000
```



![App Output](/node-cicd.png)

---

##  **Final Output :white_check_mark:**

* ### Jenkins pipeline executed successfully
* ### Node.js dependencies installed
* ### Application running on EC2
* ### App accessible using Public IP

---

##  **:small_red_triangle: CI/CD Benefits** 

* ### Fully automated deployment
* ### Simple and beginner-friendly pipeline
* ### Easy to understand Jenkinsfile

 ## **Project Summary** :red_circle: 

**In this project, I implemented a Jenkins CI/CD pipeline to deploy a Node.js application on an AWS EC2 instance without using Docker. Jenkins is configured using Pipeline as Code (Jenkinsfile) and integrated with GitHub SCM.**

**Whenever the pipeline is triggered, Jenkins automatically pulls the latest code from the main branch, installs required Node.js dependencies using npm, and deploys the application on EC2. The Node.js application is managed using PM2, which ensures the app runs in the background, restarts automatically on failure, and remains active even after the Jenkins job completes or the server restarts.**

**After a successful build, the application is accessible through the EC2 public IP and application port, demonstrating a complete end-to-end CI/CD workflow using Jenkins.**





