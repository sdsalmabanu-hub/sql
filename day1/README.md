# Jenkins Day 1: Introduction, Architecture, and Installation

## Class Goal

By the end of Day 1, students should understand:

- Why Jenkins is needed
- What CI/CD means
- Jenkins architecture
- Jenkins installation using Docker
- Jenkins installation using Ubuntu apt
- First-time Jenkins setup
- Jenkins SSH key setup for GitHub/GitLab

## 1. Why Jenkins Is Needed

In a real company, developers push code many times in a day. After each code change, someone may need to:

1. Pull the latest code from GitHub or GitLab.
2. Install dependencies.
3. Run tests.
4. Build the application.
5. Package the application.
6. Deploy the application.

If these steps are done manually every time, problems can happen.

Common problems in manual deployment:

- Human mistakes
- Slow delivery
- Repeated manual commands
- No automatic testing
- No clear build history
- Difficult troubleshooting
- Deployment depends on one person

Jenkins solves this by automating repeated build, test, and deployment tasks.

## 2. What Is Jenkins?

Jenkins is an open-source automation server used to automate software build, test, package, and deployment processes.

Simple explanation:

> Jenkins is like a DevOps automation engine. When code changes, Jenkins can automatically pull the code, build it, test it, and deploy it.

Jenkins does not replace tools like Git, Docker, Maven, npm, Linux, or Kubernetes. Jenkins connects those tools and runs them in the correct order.

| Requirement | Tool | Jenkins Role |
| --- | --- | --- |
| Store source code | GitHub / GitLab | Pulls latest code |
| Build Java app | Maven / Gradle | Runs build commands |
| Build Node.js app | npm | Runs install/test commands |
| Package app | Docker | Builds Docker image |
| Deploy app | Docker / Kubernetes / SSH | Runs deployment commands |

## 3. What Is CI/CD?

### Continuous Integration

Continuous Integration means developers frequently push code to a shared repository, and every code change is automatically built and tested.

CI flow:

```text
Developer pushes code
        |
        v
GitHub / GitLab
        |
        v
Jenkins job starts
        |
        v
Jenkins checks out code
        |
        v
Jenkins builds application
        |
        v
Jenkins runs tests
        |
        v
Success or Failure
```

CI helps teams find errors early.

### Continuous Delivery

Continuous Delivery means code is automatically built, tested, and prepared for deployment, but production deployment usually needs manual approval.

### Continuous Deployment

Continuous Deployment means every successful code change is automatically deployed without manual approval.

| Concept | Meaning | Example |
| --- | --- | --- |
| CI | Automatically build and test code | Run tests after every push |
| Continuous Delivery | Keep code ready to deploy | Build Docker image and wait for approval |
| Continuous Deployment | Deploy automatically after success | Push to main deploys app |

## 4. Jenkins Architecture

Jenkins follows a controller-agent architecture.

```text
Developer
   |
   | git push
   v
GitHub / GitLab Repository
   |
   | webhook or manual build
   v
Jenkins Controller
   |
   +-- Plugins
   +-- Credentials
   +-- Build History
   +-- Job Configuration
   |
   v
Jenkins Job / Pipeline
   |
   v
Workspace
   |
   +-- Build tool: Maven / npm / Gradle
   +-- Test tool: JUnit / Pytest / npm test
   +-- Docker build
   +-- Deployment command

Optional:

Jenkins Controller ---> Jenkins Agent ---> Build/Test/Deploy
```

### Jenkins Controller

The Jenkins Controller is the main Jenkins server.

It manages:

- Jenkins web UI
- Jobs
- Plugins
- Credentials
- Build history
- Build scheduling
- Agent connections

Simple analogy:

> Jenkins Controller is the brain of Jenkins.

### Jenkins Agent

A Jenkins Agent is another machine where Jenkins can run jobs.

For Day 1, we can run Jenkins as a single server. Later, agents are used for larger projects.

### Jenkins Job

A Jenkins job is a task configured in Jenkins.

Examples:

- Pull code from Git
- Run shell commands
- Build application
- Run tests
- Build Docker image
- Deploy application

### Jenkins Workspace

Workspace is the folder where Jenkins stores files for a job.

Example for Docker Jenkins:

```text
/var/jenkins_home/workspace/job-name
```

Example for Ubuntu Jenkins:

```text
/var/lib/jenkins/workspace/job-name
```

### Jenkins Plugins

Plugins add extra features to Jenkins.

Common plugins:

| Plugin | Purpose |
| --- | --- |
| Git Plugin | Pull source code from Git repositories |
| Pipeline Plugin | Create Jenkinsfile-based pipelines |
| Credentials Plugin | Store secrets safely |
| Docker Plugin | Work with Docker workflows |
| GitHub / GitLab Plugin | Repository and webhook integration |

## 5. Jenkins Installation Method 1: Docker

This is the recommended practice method for students.

### Create Persistent Jenkins Volume

```bash
docker volume create jenkins_home
```

Why volume is important:

If Jenkins data is stored only inside the container, jobs and configuration may be lost when the container is removed. A Docker volume keeps Jenkins data persistent.

### Run Jenkins Container

```bash
docker run -d --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

PowerShell single-line command:

```powershell
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

Command explanation:

| Command Part | Meaning |
| --- | --- |
| `docker run` | Creates and starts a container |
| `-d` | Runs container in background |
| `--name jenkins` | Names the container `jenkins` |
| `-p 8080:8080` | Maps Jenkins web UI to localhost port 8080 |
| `-p 50000:50000` | Port used by Jenkins agents |
| `-v jenkins_home:/var/jenkins_home` | Stores Jenkins data in Docker volume |
| `jenkins/jenkins:lts` | Official Jenkins LTS image |

### Verify Jenkins Container

```bash
docker ps
```

Check logs:

```bash
docker logs jenkins
```

Open Jenkins:

```text
http://localhost:8080
```

### Get Initial Admin Password

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

If using Git Bash on Windows, use:

```bash
MSYS_NO_PATHCONV=1 docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Alternative Git Bash fix:

```bash
docker exec jenkins cat //var/jenkins_home/secrets/initialAdminPassword
```

PowerShell works normally:

```powershell
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## 6. Jenkins Installation Method 2: Ubuntu apt

Use this method on an Ubuntu server or VM.

### Install Java

```bash
sudo apt update
sudo apt install -y openjdk-21-jre
java -version
```

### Add Jenkins Repository

```bash
sudo apt install -y curl gnupg

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Install Jenkins

```bash
sudo apt-get update
sudo apt-get install -y jenkins
```

Important:

Use lowercase `jenkins`. Linux package names are usually lowercase.

### Start Jenkins Service

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

If firewall is enabled:

```bash
sudo ufw allow 8080
sudo ufw status
```

Open Jenkins:

```text
http://SERVER_PUBLIC_IP:8080
```

### Get Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 7. First-Time Jenkins Setup

After opening Jenkins in browser:

1. Paste the initial admin password.
2. Select **Install suggested plugins**.
3. Wait for plugin installation.
4. Create the first admin user.
5. Confirm Jenkins URL.
6. Open Jenkins dashboard.

Recommended admin example for lab:

```text
Username: admin
Password: Admin@123
Full Name: Jenkins Admin
Email: admin@example.com
```

For real projects, always use a strong password.

## 8. Jenkins SSH Key Setup

Jenkins may need SSH access to private Git repositories.

### Ubuntu apt Jenkins

```bash
sudo su - jenkins
ssh-keygen -t rsa -b 4096 -C "email@gmail.com"
cat /var/lib/jenkins/.ssh/id_rsa.pub
ssh -T git@gitlab.com
```

For GitHub:

```bash
ssh -T git@github.com
```

### Docker Jenkins

```bash
docker exec -it -u jenkins jenkins bash
ssh-keygen -t rsa -b 4096 -C "email@gmail.com"
cat /var/jenkins_home/.ssh/id_rsa.pub
ssh -T git@gitlab.com
```

Copy the public key and add it to:

- GitLab: User Settings -> SSH Keys
- GitHub: Settings -> SSH and GPG keys

Important:

- Public key can be shared with GitLab/GitHub.
- Private key must remain secret.
- Never commit private keys to Git.

## 9. Day 1 Practical Tasks

Students should complete:

1. Install Jenkins using Docker or Ubuntu apt.
2. Open Jenkins in browser.
3. Unlock Jenkins using initial admin password.
4. Install suggested plugins.
5. Create admin user.
6. Open Jenkins dashboard.
7. Generate Jenkins SSH key.
8. Add public key to GitLab or GitHub.
9. Test SSH connection.

## 10. Day 1 Troubleshooting

| Issue | Reason | Fix |
| --- | --- | --- |
| `localhost:8080` not opening | Jenkins not running | Run `docker ps` or `systemctl status jenkins` |
| Port 8080 already used | Another service uses port | Use `-p 8081:8080` |
| Password file not found in Git Bash | Path conversion issue | Use `MSYS_NO_PATHCONV=1` |
| `apt install Jenkins` fails | Wrong package name case | Use `sudo apt-get install -y jenkins` |
| SSH permission denied | Public key not added | Add Jenkins public key to Git provider |

## 11. Interview Questions

### Q1. What is Jenkins?

Jenkins is an open-source automation server used to automate build, test, package, and deployment tasks.

### Q2. What is CI?

Continuous Integration means automatically building and testing code whenever developers push changes.

### Q3. What is the Jenkins Controller?

The Jenkins Controller is the main Jenkins server that manages jobs, plugins, credentials, users, build history, and scheduling.

### Q4. What is Jenkins workspace?

Workspace is the folder where Jenkins stores files for a specific job.

### Q5. Why do we use plugins in Jenkins?

Plugins add extra features and integrations, such as Git, Docker, Pipeline, and Credentials support.
