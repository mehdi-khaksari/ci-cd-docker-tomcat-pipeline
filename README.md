#  Java WebApp CI/CD Pipeline with Git, Ansible and Docker

This repository demonstrates a lightweight CI/CD pipeline for a Java web application using:

-  **Git server** with a `post-receive` hook  
-  **Ansible server** to orchestrate deployment  
-  **Docker host** to run the final container

## 🗂️ Repository Structure

```txt
.
├── ansible
│   ├── cicd-docker.yml
│   ├── inventory
│   │   └── cicd
│   └── roles
│       └── cicd-docker
│           ├── files
│           │   └── my-webapp-1.0-SNAPSHOT.war
│           └── tasks
│              └── main.yml
└── git
    ├── app
    │   ├── pom.xml
    │   ├── src
    │   │   └── main
    │   │       └── java
    │   │          └── com
    │   │              └── example
    │   │                  └── App.java
    │   └── target
    │       └── my-webapp-1.0-SNAPSHOT.war
    └── hooks
        └── post-receive
```

## 🧱 Server Architecture

```txt
┌────────────┐        git push       
│ Developer  │ ──────────────────────────── |
└────────────┘                              |
                                            ▼
                                    ┌─────────────┐
                                    │ Git Server  │
                                    │ 10.10.10.20 │
                                    └──────┬──────┘
                                           │ post-receive hook
                                           ▼
                                    ┌────────────────┐
                                    │ Ansible Server │
                                    │ 10.10.10.17    │
                                    └──────┬─────────┘
                                           │ ansible-playbook
                                           ▼    
                                    ┌──────────────┐
                                    │ Docker Server│
                                    │ 10.10.10.12  │
                                    └──────────────┘
```
## 🔁 CI/CD Workflow
1. Developer edits the Java file:
```bash
vim src/main/java/com/example/App.java
```
2. Build, commit and push:
```bash
git add .
git commit -m "Update App.java"
git push origin master
```
3. Git server post-receive hook triggers automatically:

- Pulls latest code into working dir /home/app
- Runs Maven to build WAR file
- SSH to Ansible server to run playbook

4. Ansible Playbook performs:

- Removes old .war files (from Ansible + Docker server)
- Fetches new .war from Git server
- Copies .war to Docker host
- Removes old Docker container/image
- Builds new Docker image (tomcat10)
- Runs new Tomcat container with updated app

## 📩 post-receive script file in /srv/git/app.git/hooks/ 
```bash
#!/bin/bash

REPO_DIR="/srv/git/app.git"
WORK_DIR="/home/app"
ANSIBLE_SERVER="10.10.10.17"

echo "===> Pull latest code..."
cd "$WORK_DIR" || exit 1
git pull origin master

echo "===> Building project..."
mvn clean package

echo "===> Running Ansible playbook on $ANSIBLE_SERVER ..."
ssh root@$ANSIBLE_SERVER "cd /home/ansible/provision && ansible-playbook -i inventory/cicd cicd-docker.yml"
```

## 📡 Ansible Playbook Tasks Summary
File: roles/cicd-docker/tasks/main.yml
```yml
# On localhost (Ansible host)
- Remove old war
- Fetch new war from Git server

# On Docker server (10.10.10.12)
- Remove old war
- Copy new war to Tomcat build dir
- Remove old container and image
- Build new Docker image: tomcat10
- Run new container on port 80 → 8080
```


## 🐳 Docker Server (10.10.10.12)
- Builds image from /home/docker-file/tomcat/
- New WAR file copied to that directory
- New container tomcat runs with port mapping: 80:8080


# 📬 Contact
Feel free to fork, improve, and contribute!

Author: mehdi khaksari 

Email: mahdikhaksari36@gmail.com



