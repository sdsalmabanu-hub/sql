# Jenkins Day 3: Plugins, Multibranch Pipeline, and Real CI Stages

## Class Goal

By the end of Day 3, students should understand:

- What Jenkins plugins are and why they are important
- Which plugins are commonly used in real DevOps projects
- What a multibranch pipeline is
- How Jenkins discovers branches automatically
- How real CI stages work in a production-style pipeline
- How code moves from Git checkout to Docker image build, tag, and push to AWS ECR
- How to create a multibranch pipeline project using a `Jenkinsfile`

## 1. Day 2 Recap

Ask students:

- What is Matrix-based security?
- What are Jenkins credentials?
- What is a freestyle project?
- Where are Jenkins build logs stored?
- What is the difference between freestyle and pipeline?

Then say:

> Yesterday we created Jenkins jobs manually. Today we will move closer to real company practice using plugins, Jenkinsfile, and multibranch pipeline.

## 2. What Are Jenkins Plugins?

Jenkins plugins are add-ons that extend Jenkins functionality.

Simple explanation:

> Jenkins core gives the basic automation server. Plugins give Jenkins the power to connect with GitHub, Docker, AWS, Kubernetes, Slack, email, Maven, Gradle, and many other tools.

Without plugins, Jenkins is limited. With plugins, Jenkins can integrate with almost every DevOps tool.

## 3. Why Plugins Are Needed

Example real project:

```text
Developer pushes code to GitHub
        |
        v
Jenkins pulls code
        |
        v
Jenkins builds Docker image
        |
        v
Jenkins pushes image to AWS ECR
        |
        v
Jenkins deploys to server or Kubernetes
```

To do this smoothly, Jenkins may need plugins for:

- Git integration
- Pipeline syntax
- Credentials
- Docker
- AWS
- Blue Ocean or better UI
- Notifications

## 4. Common Jenkins Plugins

| Plugin | Purpose | Example Use |
| --- | --- | --- |
| Git Plugin | Pull code from Git repositories | Checkout code from GitHub |
| GitHub Plugin | GitHub integration | GitHub hooks and repository connection |
| Pipeline Plugin | Run Jenkinsfile pipelines | Build/test/deploy as code |
| Multibranch Pipeline Plugin | Discover branches automatically | Build `main`, `dev`, and feature branches |
| Credentials Plugin | Store secrets safely | GitHub token, AWS keys, DockerHub password |
| Credentials Binding Plugin | Use credentials inside pipeline | Inject AWS keys into environment |
| Docker Pipeline Plugin | Docker operations in pipeline | Build Docker images |
| AWS Credentials Plugin | Store AWS credentials | Access AWS services securely |
| Blue Ocean Plugin | Better pipeline visualization | View pipeline stages clearly |
| Email Extension Plugin | Email notifications | Send build failure email |
| Slack Notification Plugin | Slack alerts | Notify team when build fails |

## 5. How to Install Plugins

UI path:

```text
Jenkins Dashboard
  -> Manage Jenkins
  -> Plugins
  -> Available plugins
  -> Search plugin name
  -> Select plugin
  -> Install
  -> Restart Jenkins if required
```

Recommended plugins for Day 3:

```text
Git
GitHub
Pipeline
Multibranch Pipeline
Credentials
Credentials Binding
Docker Pipeline
AWS Credentials
Blue Ocean
```

Teaching point:

> Do not install random plugins in production. Every plugin increases maintenance and security responsibility. Install only what the project needs.

## 6. What Is a Pipeline?

A Jenkins pipeline is CI/CD logic written as code inside a file called `Jenkinsfile`.

Simple flow:

```text
Checkout
  -> Build
  -> Test
  -> Docker Build
  -> Docker Tag
  -> Push Image
  -> Deploy
```

Pipeline is better than freestyle because:

- Pipeline code is stored in Git
- Changes are version controlled
- Team can review pipeline changes
- Same pipeline can run again and again
- It supports multiple stages clearly

## 7. What Is a Multibranch Pipeline?

A multibranch pipeline automatically discovers branches in a Git repository and runs the `Jenkinsfile` from each branch.

Example repository:

```text
main branch      -> Jenkinsfile
dev branch       -> Jenkinsfile
feature-login    -> Jenkinsfile
feature-payment  -> Jenkinsfile
```

Jenkins creates branch jobs automatically:

```text
Multibranch-Project
  -> main
  -> dev
  -> feature-login
  -> feature-payment
```

Why companies use multibranch pipeline:

- Real projects have many branches
- Each branch should be tested separately
- Feature branch code can be validated before merging
- Pull request workflows become easier
- Jenkins automatically discovers new branches

## 8. Freestyle vs Pipeline vs Multibranch

| Feature | Freestyle | Pipeline | Multibranch Pipeline |
| --- | --- | --- | --- |
| Configuration | Jenkins UI | Jenkinsfile | Jenkinsfile from each branch |
| Stored in Git | No | Yes | Yes |
| Branch handling | Manual | Usually one branch | Automatic branch discovery |
| Best for | Beginners/simple jobs | Real CI/CD | Large projects with branches |
| Industry usage | Low/medium | High | Very high |

Simple explanation:

```text
Freestyle:
Commands are configured in Jenkins UI.

Pipeline:
Commands are written in Jenkinsfile.

Multibranch:
Jenkins finds branches automatically and runs Jenkinsfile from each branch.
```

## 9. Real-Time Jenkins Stages Explained

In real projects, Jenkins pipelines are divided into stages.

Example real CI flow:

```text
Stage 1: Git Checkout
Stage 2: Unit Test
Stage 3: Docker Build
Stage 4: Docker Tag
Stage 5: Login to AWS ECR
Stage 6: Push Docker Image to ECR
Stage 7: Deploy
```

## 10. Stage 1: Git Checkout

Purpose:

> Jenkins pulls the latest source code from GitHub/GitLab into the Jenkins workspace.

Example:

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

What happens internally:

```text
Jenkins connects to Git repository
Jenkins downloads branch code
Code is stored in Jenkins workspace
Pipeline starts using that source code
```

Workspace example in Docker Jenkins:

```text
/var/jenkins_home/workspace/<job-name>
```

Workspace example in Ubuntu Jenkins:

```text
/var/lib/jenkins/workspace/<job-name>
```

## 11. Stage 2: Unit Test

Purpose:

> Jenkins checks whether the application code is working before building an image.

Example for Python:

```groovy
stage('Test') {
    steps {
        sh 'python3 --version'
        sh 'python3 -m py_compile app.py'
    }
}
```

Example for Node.js:

```groovy
stage('Test') {
    steps {
        sh 'npm install'
        sh 'npm test'
    }
}
```

Example for Java:

```groovy
stage('Test') {
    steps {
        sh 'mvn test'
    }
}
```

Teaching point:

> If tests fail, Jenkins should stop. We should not build and push a bad application image.

## 12. Stage 3: Docker Build

Purpose:

> Jenkins builds a Docker image from the application source code and Dockerfile.

Example:

```groovy
stage('Docker Build') {
    steps {
        sh 'docker build -t my-app:${BUILD_NUMBER} .'
    }
}
```

What is `BUILD_NUMBER`?

`BUILD_NUMBER` is a Jenkins environment variable. Every Jenkins build gets a number.

Example:

```text
Build #1 -> image tag 1
Build #2 -> image tag 2
Build #3 -> image tag 3
```

Why this is useful:

- Every image version is unique
- Easy rollback
- Easy troubleshooting
- We can identify which Jenkins build created which image

## 13. Stage 4: Docker Tag

Purpose:

> Docker tag gives the image the correct registry name so it can be pushed to AWS ECR.

Local image:

```text
my-app:5
```

ECR image format:

```text
AWS_ACCOUNT_ID.dkr.ecr.AWS_REGION.amazonaws.com/REPOSITORY_NAME:TAG
```

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/jenkins-demo:5
```

Command:

```bash
docker tag my-app:5 123456789012.dkr.ecr.ap-south-1.amazonaws.com/jenkins-demo:5
```

Pipeline example:

```groovy
stage('Docker Tag') {
    steps {
        sh '''
        docker tag ${APP_NAME}:${BUILD_NUMBER} \
        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}
        '''
    }
}
```

## 14. Stage 5: Login to AWS ECR

Purpose:

> Before Jenkins can push Docker images to ECR, Docker must authenticate with AWS ECR.

AWS CLI command:

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

Pipeline example:

```groovy
stage('Login to ECR') {
    steps {
        sh '''
        aws ecr get-login-password --region ${AWS_REGION} | \
        docker login --username AWS --password-stdin \
        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        '''
    }
}
```

Important:

> Jenkins needs AWS permission to push to ECR. In production, prefer IAM role on EC2/agent. For class, Jenkins credentials can be used.

## 15. Stage 6: Push Docker Image to ECR

Purpose:

> Jenkins uploads the Docker image to AWS Elastic Container Registry.

Command:

```bash
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/jenkins-demo:5
```

Pipeline example:

```groovy
stage('Push to ECR') {
    steps {
        sh '''
        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}
        '''
    }
}
```

After push, the image is available in ECR and can be deployed to:

- EC2
- ECS
- EKS
- Kubernetes
- Docker server

## 16. Stage 7: Deploy

Purpose:

> Jenkins deploys the new image to an environment.

For Day 3, deployment can be explained only. Actual deployment can be done in later classes.

Examples:

EC2 Docker deployment:

```bash
docker pull <ecr-image-url>
docker stop app || true
docker rm app || true
docker run -d --name app -p 80:5000 <ecr-image-url>
```

Kubernetes deployment:

```bash
kubectl set image deployment/my-app my-app=<ecr-image-url>
```

Teaching point:

> Build and push creates the artifact. Deployment runs that artifact in an environment.

## 17. Important Setup for Docker Jenkins

If Jenkins is running inside Docker and you want Jenkins to run Docker commands, Jenkins needs access to Docker.

For training, one common approach is to mount the host Docker socket:

```bash
docker run -d --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

Important warning:

> Mounting `/var/run/docker.sock` gives Jenkins powerful access to the Docker host. It is useful for lab practice, but in production use a dedicated Jenkins agent and proper security.

The default Jenkins image may not include Docker CLI or AWS CLI. For real Docker/ECR pipelines, use one of these options:

1. Use a Jenkins agent that already has Docker CLI and AWS CLI installed.
2. Build a custom Jenkins image with Docker CLI and AWS CLI.
3. Run the pipeline on an EC2 Jenkins agent with Docker and AWS CLI installed.

## 18. Day 3 Practical Project

Sample project folder:

[projects/multibranch-ecr-demo](projects/multibranch-ecr-demo)

Project files:

| File | Purpose |
| --- | --- |
| `app.py` | Simple Python web application |
| `requirements.txt` | Python dependency list |
| `Dockerfile` | Builds Docker image |
| `Jenkinsfile` | Multibranch pipeline with Docker/ECR stages |
| `.dockerignore` | Keeps Docker build context clean |

## 19. Create GitHub Repository for Day 3

Create a GitHub repository:

```text
jenkins-multibranch-ecr-demo
```

Copy files from:

```text
day3/projects/multibranch-ecr-demo
```

Push to GitHub:

```bash
git init
git add .
git commit -m "add multibranch ecr demo"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/jenkins-multibranch-ecr-demo.git
git push -u origin main
```

Create a dev branch:

```bash
git checkout -b dev
echo "dev branch update" >> README.md
git add .
git commit -m "update dev branch"
git push -u origin dev
```

Now GitHub has:

```text
main
dev
```

## 20. Create Jenkins Credentials for AWS

For class practice, create AWS credentials in Jenkins.

UI path:

```text
Jenkins Dashboard
  -> Manage Jenkins
  -> Credentials
  -> System
  -> Global credentials
  -> Add Credentials
```

Create two secret text credentials:

```text
Kind: Secret text
Secret: <AWS_ACCESS_KEY_ID>
ID: aws-access-key-id
Description: AWS access key id
```

```text
Kind: Secret text
Secret: <AWS_SECRET_ACCESS_KEY>
ID: aws-secret-access-key
Description: AWS secret access key
```

Important:

> In production, prefer IAM roles instead of storing AWS keys in Jenkins.

## 21. Create ECR Repository

In AWS Console:

```text
AWS Console
  -> ECR
  -> Private repositories
  -> Create repository
  -> Repository name: jenkins-demo
```

Using AWS CLI:

```bash
aws ecr create-repository --repository-name jenkins-demo --region ap-south-1
```

## 22. Create Multibranch Pipeline in Jenkins

UI path:

```text
Jenkins Dashboard
  -> New Item
  -> Name: Multibranch-ECR-Demo
  -> Select: Multibranch Pipeline
  -> OK
```

Configure:

```text
Branch Sources
  -> Add source
  -> Git
  -> Project Repository: https://github.com/YOUR_USERNAME/jenkins-multibranch-ecr-demo.git
  -> Credentials: GitHub credentials if private repository
```

Build configuration:

```text
Mode: by Jenkinsfile
Script Path: Jenkinsfile
```

Click:

```text
Save
Scan Multibranch Pipeline Now
```

Expected result:

```text
main branch discovered
dev branch discovered
```

Open each branch:

```text
Multibranch-ECR-Demo
  -> main
  -> Build Now

Multibranch-ECR-Demo
  -> dev
  -> Build Now
```

## 23. Jenkinsfile Used in Project

The sample project contains this real-time style pipeline:

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '123456789012'
        ECR_REPO = 'jenkins-demo'
        APP_NAME = 'jenkins-demo'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_IMAGE = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${APP_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Tag') {
            steps {
                sh 'docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_IMAGE}'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push ${ECR_IMAGE}'
            }
        }
    }
}
```

Before using it, replace:

```text
AWS_ACCOUNT_ID = '123456789012'
AWS_REGION = 'ap-south-1'
ECR_REPO = 'jenkins-demo'
```

with your real AWS values.

## 24. What to Explain During Practical

Use this explanation while building:

```text
Checkout:
Jenkins pulls code from the current branch.

Test:
Jenkins validates code before creating image.

Docker Build:
Jenkins creates Docker image from Dockerfile.

Docker Tag:
Jenkins gives image a proper ECR registry name.

Login to ECR:
Jenkins authenticates Docker with AWS ECR.

Push to ECR:
Jenkins uploads image to AWS ECR.
```

## 25. Common Errors and Fixes

| Error | Reason | Fix |
| --- | --- | --- |
| `docker: not found` | Docker CLI not installed in Jenkins agent | Install Docker CLI or use Docker-enabled agent |
| `permission denied /var/run/docker.sock` | Jenkins cannot access Docker socket | Add permission or use proper Docker agent |
| `aws: not found` | AWS CLI not installed | Install AWS CLI on Jenkins agent |
| `no basic auth credentials` | Docker not logged in to ECR | Run ECR login stage |
| `repository does not exist` | ECR repo missing | Create ECR repository first |
| `AccessDeniedException` | AWS credentials lack permission | Add ECR permissions to IAM user/role |
| Branch not discovered | Branch has no Jenkinsfile or scan not run | Add Jenkinsfile and scan multibranch pipeline |

## 26. Homework

Students should submit:

1. Screenshot of installed Jenkins plugins.
2. Screenshot of multibranch pipeline configuration.
3. Screenshot of branch scan showing `main` and `dev`.
4. Screenshot of pipeline stages.
5. Screenshot of Docker build stage output.
6. Screenshot of ECR image after push.
7. Short explanation of each stage: checkout, test, docker build, tag, ECR login, push.

## 27. Interview Questions

### Q1. What is a Jenkins plugin?

A Jenkins plugin is an add-on that extends Jenkins features or integrates Jenkins with external tools.

### Q2. Why do we use multibranch pipeline?

Multibranch pipeline automatically discovers repository branches and runs the Jenkinsfile from each branch.

### Q3. What is `checkout scm`?

`checkout scm` tells Jenkins to checkout the source code from the SCM configuration of the job or branch.

### Q4. Why do we tag Docker images?

Docker tags identify image versions and registry locations. Tags help with traceability and rollback.

### Q5. What is ECR?

Amazon Elastic Container Registry is an AWS service used to store Docker container images.

### Q6. Why should tests run before Docker push?

Tests should run before pushing so broken code does not become a published Docker image.

### Q7. What is the difference between build and deploy?

Build creates an artifact such as a Docker image. Deploy runs that artifact in an environment.
