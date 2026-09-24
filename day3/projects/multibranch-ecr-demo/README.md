# Multibranch ECR Demo

This is the sample project for Jenkins Day 3.

It demonstrates a real-time Jenkins pipeline flow:

```text
Git Checkout
  -> Test
  -> Docker Build
  -> Docker Tag
  -> Login to AWS ECR
  -> Push Docker Image to ECR
```

## Files

| File | Purpose |
| --- | --- |
| `app.py` | Simple Python Flask application |
| `requirements.txt` | Python dependency list |
| `Dockerfile` | Builds the Docker image |
| `Jenkinsfile` | Jenkins multibranch pipeline |
| `.dockerignore` | Keeps Docker build context clean |

## Local Docker Test

```bash
docker build -t jenkins-demo:local .
docker run -p 5000:5000 jenkins-demo:local
```

Open:

```text
http://localhost:5000
```

## Jenkins Requirements

The Jenkins agent running this pipeline needs:

- Git
- Docker CLI
- Docker daemon access
- AWS CLI
- Jenkins credentials:
  - `aws-access-key-id`
  - `aws-secret-access-key`

Before running, update these values in `Jenkinsfile`:

```groovy
AWS_REGION = 'ap-south-1'
AWS_ACCOUNT_ID = '123456789012'
ECR_REPO = 'jenkins-demo'
```

## Create ECR Repository

```bash
aws ecr create-repository --repository-name jenkins-demo --region ap-south-1
```

## Multibranch Pipeline

Create branches:

```bash
git checkout -b dev
git push -u origin dev
```

In Jenkins:

```text
New Item
  -> Multibranch Pipeline
  -> Branch Sources
  -> Git
  -> Repository URL
  -> Script Path: Jenkinsfile
  -> Save
  -> Scan Multibranch Pipeline Now
```
