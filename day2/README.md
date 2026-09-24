# Jenkins Day 2: Security, Credentials, Logs, and Freestyle Job

## Class Goal

By the end of Day 2, students should understand:

- How to secure Jenkins using Matrix-based security
- How to create Jenkins users
- How to assign permissions
- How Jenkins credentials work
- Where Jenkins build logs are stored
- How to create a freestyle Git job
- How to run a complete practical project named `Project1`

## 1. Why Do We Secure Jenkins?

Jenkins can:

- Run shell commands
- Access source code
- Store secrets
- Build applications
- Deploy applications
- Trigger infrastructure actions

Because Jenkins has this much power, it must be secured.

Security has two parts:

| Part | Meaning | Example |
| --- | --- | --- |
| Authentication | Who are you? | Login with username/password |
| Authorization | What can you do? | Build job, configure job, administer Jenkins |

## 2. Jenkins Security Terms

| Term | Explanation |
| --- | --- |
| Security Realm | Controls where Jenkins users come from |
| Jenkins own user database | Jenkins stores users and passwords internally |
| Authorization | Controls user permissions after login |
| Matrix-based security | Permission table with usernames and checkboxes |
| Anonymous user | Any user who opens Jenkins without logging in |
| Authenticated users | All logged-in users |

## 3. Secure Jenkins Using Matrix-Based Security

Matrix-based security allows us to add usernames and select checkboxes for what each user can do.

Important:

> Before saving permissions, always give at least one admin user `Overall/Administer`. If admin permission is removed accidentally, you may lock yourself out of Jenkins.

### UI Navigation

```text
Jenkins Dashboard
  -> Manage Jenkins
  -> Security
  -> Enable Security
  -> Security Realm
  -> Jenkins own user database
  -> Authorization
  -> Matrix-based security
  -> Add users
  -> Select permissions using checkboxes
  -> Save
```

### If Matrix-Based Security Is Not Visible

Install the Matrix Authorization Strategy plugin:

```text
Manage Jenkins
  -> Plugins
  -> Available plugins
  -> Search: Matrix Authorization Strategy
  -> Install
  -> Restart Jenkins if required
```

### Create Jenkins Users

```text
Manage Jenkins
  -> Users
  -> Create User
```

Example users:

```text
admin      -> Jenkins administrator
student1   -> Developer/student user
student2   -> Developer/student user
readonly   -> Read-only user
```

### Recommended Permission Example

| User | Recommended Permissions | Use Case |
| --- | --- | --- |
| `admin` | `Overall/Administer` | Full control of Jenkins |
| `student1` | `Overall/Read`, `Job/Read`, `Job/Build`, `Job/Workspace` | Can view and run jobs |
| `student2` | `Overall/Read`, `Job/Read`, `Job/Build` | Can view and trigger builds |
| `readonly` | `Overall/Read`, `Job/Read`, `View/Read` | Can only view jobs and build status |
| `anonymous` | No permissions | Prevents unauthenticated access |

Best practice:

> Give users the minimum permissions they need. This is called the principle of least privilege.

## 4. Credentials in Jenkins

Credentials are used to store secrets safely in Jenkins. Instead of writing passwords or tokens directly inside a job, we store them in Jenkins Credentials and select them from the job configuration.

UI path:

```text
Jenkins Dashboard
  -> Manage Jenkins
  -> Credentials
  -> System
  -> Global credentials (unrestricted)
  -> Add Credentials
```

Common credential types:

| Credential Kind | Used For | Example |
| --- | --- | --- |
| Username with password | HTTPS Git repositories, Docker Hub login | username + personal access token |
| Secret text | API tokens | Slack token, GitHub token |
| SSH Username with private key | SSH Git clone | `git` + private key |
| Secret file | Config files | kubeconfig, service account JSON |

### Create Git Credentials Using HTTPS

For GitHub/GitLab HTTPS repository access, use a personal access token where required.

```text
Kind: Username with password
Scope: Global
Username: your-git-username
Password: personal access token
ID: git-https-token
Description: Git HTTPS token for Jenkins jobs
```

### Create Git Credentials Using SSH Key

```text
Kind: SSH Username with private key
Scope: Global
ID: git-ssh-key
Username: git
Private Key: Enter directly or use Jenkins user private key
Description: Git SSH key for Jenkins jobs
```

Credential safety rule:

> Never paste secrets directly inside Execute shell. Store them in Jenkins Credentials.

### Update or Delete Credentials

```text
Manage Jenkins
  -> Credentials
  -> Select credential store
  -> Click credential ID
  -> Update / Delete
```

## 5. Jenkins Build Logs

Every Jenkins build has console output. This is the first place to check when a job succeeds or fails.

From UI:

```text
Jenkins Dashboard
  -> Click job name
  -> Click build number, for example #2
  -> Console Output
```

Build logs are also stored on the Jenkins filesystem.

| Jenkins Setup | Build Log Location |
| --- | --- |
| Ubuntu apt installation | `/var/lib/jenkins/jobs/Project1/builds/2/log` |
| Docker Jenkins | `/var/jenkins_home/jobs/Project1/builds/2/log` inside container |
| Docker host with volume | `docker volume inspect jenkins_home` shows volume location |

Commands:

```bash
# Ubuntu apt Jenkins
sudo ls -la /var/lib/jenkins/jobs/Project1/builds/2
sudo cat /var/lib/jenkins/jobs/Project1/builds/2/log

# Docker Jenkins
docker exec jenkins ls -la /var/jenkins_home/jobs/Project1/builds/2
docker exec jenkins cat /var/jenkins_home/jobs/Project1/builds/2/log
```

## 6. Create a Jenkins Freestyle Job

A freestyle job is the simplest Jenkins project type. It is useful for beginners because Git, credentials, and shell commands can be configured from the UI.

### Create Job

```text
Jenkins Dashboard
  -> New Item
  -> Enter item name: Project1
  -> Select Freestyle project
  -> OK
```

### Configure Git SCM

```text
Project1 Configure Page
  -> Source Code Management
  -> Select Git
  -> Repository URL: https://github.com/USERNAME/Project1.git
  -> Credentials: select created credential
  -> Branch Specifier: */main
```

Repository URL examples:

```text
HTTPS: https://github.com/USERNAME/Project1.git
SSH: git@github.com:USERNAME/Project1.git
```

For HTTPS use username/token credentials. For SSH use SSH private key credentials.

### Add Execute Shell Commands

```text
Build Steps
  -> Add build step
  -> Execute shell
```

Example:

```bash
echo "Project1 build started"
echo "Current user:"
whoami

echo "Workspace path:"
pwd

echo "Files from Git repository:"
ls -la

echo "Build completed successfully"
```

Click:

```text
Save
Build Now
Console Output
```

Teaching point:

> A Jenkins freestyle job is simply a configured automation task. Jenkins checks out the code, enters the workspace, and runs the shell commands we provided.

## 7. Complete Day 2 Practical: Project1

The sample project is available here:

[Project1 sample files](practical/Project1)

Students can either create their own repository manually or use the sample files in this folder.

### Step 1: Create a Git Repository

Create a repository in GitHub named:

```text
Project1
```

Copy the files from:

```text
day2/practical/Project1
```

Or create them manually:

```bash
mkdir Project1
cd Project1

echo "# Project1 Jenkins Demo" > README.md

cat > hello.sh <<'EOF'
#!/bin/bash
echo "Hello from Jenkins Day 2 practical"
echo "This script is running inside Jenkins workspace"
echo "Current user: $(whoami)"
echo "Current directory: $(pwd)"
date
EOF

chmod +x hello.sh
```

Push to GitHub:

```bash
git init
git add .
git commit -m "initial Project1 files"
git branch -M main
git remote add origin https://github.com/USERNAME/Project1.git
git push -u origin main
```

### Step 2: Create Credentials

1. Open `Manage Jenkins -> Credentials`.
2. Open `System -> Global credentials`.
3. Click `Add Credentials`.
4. Select `Username with password` for HTTPS token or `SSH Username with private key` for SSH clone.
5. Give a clear ID such as `github-project1-token` or `github-ssh-key`.
6. Save the credential.

### Step 3: Create Freestyle Job

1. Open `Jenkins Dashboard -> New Item`.
2. Enter `Project1` as job name.
3. Select `Freestyle project`.
4. In `Source Code Management`, select `Git`.
5. Paste repository URL.
6. Select credentials.
7. Set branch as `*/main`.
8. Add `Execute shell` build step.
9. Save and click `Build Now`.

### Step 4: Execute Shell for Project1

Use this in Jenkins Execute shell:

```bash
echo "Project1 build started"
echo "Workspace: $(pwd)"
ls -la
chmod +x hello.sh
./hello.sh
echo "Project1 build finished"
```

### Step 5: Verify Build Result

Expected output:

```text
Project1 build started
Workspace: /var/lib/jenkins/workspace/Project1
Hello from Jenkins Day 2 practical
This script is running inside Jenkins workspace
Project1 build finished
Finished: SUCCESS
```

For Docker Jenkins, workspace may be:

```text
/var/jenkins_home/workspace/Project1
```

### Step 6: Check Build Logs

Ubuntu Jenkins:

```bash
sudo cat /var/lib/jenkins/jobs/Project1/builds/1/log
```

Docker Jenkins:

```bash
docker exec jenkins cat /var/jenkins_home/jobs/Project1/builds/1/log
```

### Step 7: Create a Failed Build

Change Execute shell temporarily:

```bash
echo "Testing failed build"
wrongcommand
echo "This line may not execute"
```

Run `Build Now`.

Expected result:

```text
Finished: FAILURE
```

Explanation:

Jenkins marks the build as failed because one command failed. In real projects, failed tests, failed Docker builds, or failed deployments will also fail the Jenkins build.

## 8. Types of Jenkins Projects

| Project Type | Meaning | When to Use |
| --- | --- | --- |
| Freestyle | Basic UI-based job configuration | Beginners and simple jobs |
| Pipeline | CI/CD steps written as code using Groovy in a Jenkinsfile | Real CI/CD projects |
| Multibranch Pipeline | Jenkins discovers branches and runs each branch Jenkinsfile | Large projects with feature branches |

Simple summary:

- Freestyle = click-based job setup
- Pipeline = Jenkinsfile-based automation
- Multibranch Pipeline = Jenkinsfile automation for many branches

## 9. Common Errors and Fixes

| Error | Reason | Fix |
| --- | --- | --- |
| Authentication failed while cloning | Wrong credentials or token | Create correct Jenkins credentials and select them in SCM |
| Repository not found | Wrong URL or no access | Check repo URL and user permission |
| Permission denied publickey | SSH key missing in GitHub/GitLab | Add Jenkins public key to Git provider |
| No such file `hello.sh` | File not present in repo or wrong branch | Check branch and repository contents |
| Permission denied running script | Script is not executable | Run `chmod +x hello.sh` |
| User cannot see job | Matrix permission missing | Give `Overall/Read` and `Job/Read` |
| User cannot build job | `Job/Build` permission missing | Enable `Job/Build` for that user |

## 10. Homework

Students should submit:

1. Screenshot of Matrix-based security configuration.
2. Screenshot of credentials ID only. Do not expose the secret value.
3. Screenshot of `Project1` Git SCM configuration.
4. Screenshot of successful Console Output.
5. Screenshot of failed Console Output.
6. Build log path from server/container.
7. Short explanation of why the failed build failed.

## 11. Interview Questions

### Q1. What is Matrix-based security in Jenkins?

Matrix-based security is an authorization method where users or groups are added to a permission matrix and access is granted using checkboxes.

### Q2. What is the difference between authentication and authorization?

Authentication verifies who the user is. Authorization decides what the user is allowed to do after login.

### Q3. What is Jenkins own user database?

It is Jenkins' built-in user store where Jenkins manages usernames and passwords internally.

### Q4. Why are credentials used in Jenkins?

Credentials store secrets like tokens, passwords, and SSH keys safely so they are not exposed inside job commands or source code.

### Q5. Where are Jenkins build logs stored?

Ubuntu apt setup:

```text
/var/lib/jenkins/jobs/<job-name>/builds/<build-number>/log
```

Docker setup:

```text
/var/jenkins_home/jobs/<job-name>/builds/<build-number>/log
```

### Q6. What is a freestyle project?

A freestyle project is a basic Jenkins job configured from the UI. It can connect to Git and run build steps such as shell commands.

### Q7. What is a pipeline project?

A pipeline project defines CI/CD steps as code using Groovy syntax, usually in a Jenkinsfile stored in Git.

### Q8. What is a multibranch pipeline?

A multibranch pipeline automatically discovers branches in a repository and runs the Jenkinsfile from each branch.
