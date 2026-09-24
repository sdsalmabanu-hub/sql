# GitHub Actions Day 4: CI/CD Automation, Triggers, Secrets, and AWS ECR Login

## Class Goal

By the end of this class, students should understand what GitHub Actions is, how workflows run on push/merge/manual trigger, how GitHub and AWS authentication works, where secrets are stored, and how a Docker image can be pushed to Amazon ECR.

## 1. What Is GitHub Actions?

GitHub Actions is a CI/CD automation service built into GitHub.

```text
Developer pushes code
        |
        v
GitHub Actions starts workflow
        |
        v
Code is tested, built, scanned, packaged, or deployed
```

CI means Continuous Integration: test and build code whenever developers make changes.

CD means Continuous Delivery or Continuous Deployment: release or deploy the application after successful checks.

## 2. GitHub Actions vs Jenkins

| Point | Jenkins | GitHub Actions |
| --- | --- | --- |
| Platform | Separate CI/CD server | Built into GitHub |
| Pipeline file | `Jenkinsfile` | `.github/workflows/*.yml` |
| Trigger | Webhook/polling/manual | GitHub events directly |
| Runner/agent | Jenkins agent | GitHub runner |
| Secrets | Jenkins credentials | GitHub Secrets/Environments |
| Maintenance | User manages server/plugins | GitHub manages hosted runners |

Simple line:

```text
Jenkins is a separate automation server. GitHub Actions is automation inside GitHub.
```

## 3. Where Workflow Files Are Stored

Workflow files must be stored in:

```text
.github/workflows/
```

Example in this repository:

```text
.github/workflows/github-actions-practical.yml
```

GitHub automatically detects YAML files in this folder.

## 4. Basic Workflow Structure

```yaml
name: My First Workflow

on:
  push:
    branches:
      - main

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Print message
        run: echo "Hello from GitHub Actions"
```

| Keyword | Meaning |
| --- | --- |
| `name` | Workflow name shown in Actions tab |
| `on` | Event that starts the workflow |
| `jobs` | Work to perform |
| `runs-on` | Runner machine image |
| `steps` | Commands/actions inside a job |
| `uses` | Reuse an existing action |
| `run` | Execute shell commands |

## 5. Important Terms

| Term | Explanation |
| --- | --- |
| Workflow | Complete automation YAML file |
| Event | Trigger such as push, pull request, manual run |
| Job | Group of steps running on a runner |
| Step | Single command or reusable action |
| Runner | Machine where workflow runs |
| Action | Reusable task from GitHub Marketplace or repository |
| Secret | Sensitive value such as password or AWS key |

## 6. Automatic Trigger: Push

This runs when code is pushed to `main`:

```yaml
on:
  push:
    branches:
      - main
```

What happens:

```text
Developer commits code
Developer pushes to main
GitHub detects push event
Workflow starts automatically
```

If a pull request is merged into `main`, GitHub creates a new commit on `main`. That is also a push event, so the workflow runs automatically.

| Action | Workflow runs? |
| --- | --- |
| Direct commit to `main` | Yes |
| Pull request merged into `main` | Yes |
| Push to `dev` | No, unless configured |
| Push to `feature/login` | No, unless configured |

Multiple branches:

```yaml
on:
  push:
    branches:
      - main
      - dev
      - feature/**
```

## 7. Pull Request Trigger

Use this when you want to validate code before merging:

```yaml
on:
  pull_request:
    branches:
      - main
```

Simple difference:

```text
pull_request = check before merge
push         = run after commit or merge
```

Common company flow:

```text
Feature branch -> Pull request -> Tests run -> Review -> Merge -> Deploy from main
```

## 8. Manual Trigger: workflow_dispatch

If you do not want automatic trigger, use manual trigger:

```yaml
on:
  workflow_dispatch:
```

GitHub shows a Run workflow button:

```text
Repository -> Actions -> Select workflow -> Run workflow
```

Manual trigger with input:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose environment"
        required: true
        default: "dev"
```

Use input:

```yaml
run: echo "Deploying to ${{ inputs.environment }}"
```

Both automatic and manual:

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

## 9. Practical Workflow in This Repository

This repository contains a safe class workflow:

```text
.github/workflows/github-actions-practical.yml
```

It runs on:

```text
push to main
pull request to main
manual workflow_dispatch
```

It performs:

```text
1. Checkout repository
2. Print event, branch, commit SHA, and actor
3. Verify README files exist
4. Compile the Day 3 Python sample
5. Explain the next production AWS/ECR step
```

This workflow does not require AWS credentials, so it is safe for a classroom push demo.

## 10. Practical: Automatic Run

Make any small code or README change:

```bash
git add README.md
git commit -m "update training notes"
git push origin main
```

Then open:

```text
GitHub repository -> Actions -> GitHub Actions Practical
```

Expected result:

```text
Workflow starts automatically because push happened on main.
```

## 11. Practical: Manual Run

Open GitHub:

```text
Repository -> Actions
GitHub Actions Practical
Run workflow
Enter student name
Run workflow
```

Expected result:

```text
Workflow runs without any new commit.
Manual input is printed in the logs.
```

## 12. Why Checkout Is Needed

The runner starts as a fresh machine. It does not automatically have your repository files.

Use:

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

Without checkout, files like `README.md`, `Dockerfile`, and application code are not available to the job.

## 13. GitHub Login in GitHub Actions

GitHub automatically creates a temporary token for each workflow run:

```text
GITHUB_TOKEN
```

Use cases:

```text
Read repository contents
Create a release
Comment on pull request
Create issues
Push tags
```

Control token permissions:

```yaml
permissions:
  contents: read
```

Important:

```text
We do not manually create GITHUB_TOKEN. GitHub creates it automatically for the workflow run.
```

## 14. Secrets and Variables

Secrets store sensitive data.

Examples:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
DOCKERHUB_TOKEN
DATABASE_PASSWORD
```

Store secrets here:

```text
GitHub repository -> Settings -> Secrets and variables -> Actions -> New repository secret
```

Use a secret:

```yaml
run: echo "${{ secrets.MY_SECRET }}"
```

Do not print real secrets in logs.

Variables are for non-sensitive values:

```text
AWS_REGION
ECR_REPOSITORY
ENVIRONMENT_NAME
```

Example:

```yaml
env:
  AWS_REGION: ap-south-1
```

## 15. AWS Login Options

### Option 1: AWS Access Keys in GitHub Secrets

Store these in GitHub Secrets:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Workflow example:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v6.1.0
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-south-1
```

This is easy for learning, but not ideal for production because long-term keys are stored in GitHub.

### Option 2: OIDC IAM Role

Recommended production method:

```text
GitHub requests a temporary identity token
AWS trusts GitHub OIDC provider
Workflow assumes an AWS IAM role
AWS returns temporary credentials
No permanent AWS key is stored in GitHub
```

Workflow example:

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - name: Configure AWS credentials
    uses: aws-actions/configure-aws-credentials@v6.1.0
    with:
      role-to-assume: arn:aws:iam::123456789012:role/github-actions-ecr-role
      aws-region: ap-south-1
```

Best practice:

```text
Use OIDC IAM Role for real projects. Use access keys only for simple labs if required.
```

## 16. Amazon ECR Login

ECR means Amazon Elastic Container Registry. It stores Docker images in AWS.

Flow:

```text
Configure AWS credentials
Login to ECR
Build Docker image
Tag Docker image
Push Docker image to ECR
```

Login action:

```yaml
- name: Login to Amazon ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2
```

Manual equivalent:

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

## 17. Full AWS ECR Example

Use this after AWS IAM role or AWS secrets are configured:

```yaml
name: Build and Push Docker Image to ECR

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: github-actions-demo

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6.1.0
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ecr-role
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

Replace account ID, role name, region, and ECR repository with real values.

## 18. Where Credentials Are Stored

| Item | Where stored |
| --- | --- |
| `GITHUB_TOKEN` | Automatically created by GitHub |
| AWS IAM role | AWS IAM |
| AWS access key ID | GitHub Secrets, if using key method |
| AWS secret access key | GitHub Secrets, if using key method |
| AWS region | Workflow `env` or GitHub Variables |
| ECR repository name | Workflow `env` or GitHub Variables |
| DockerHub token | GitHub Secrets |
| Database password | GitHub Secrets or cloud secret manager |

Never store credentials in:

```text
README.md
Workflow YAML directly
Application source code
Dockerfile
Git commits
```

## 19. Common Errors and Fixes

| Error | Reason | Fix |
| --- | --- | --- |
| Workflow not running | YAML not inside `.github/workflows` | Move file to correct folder |
| Workflow not running on branch | Branch filter mismatch | Add correct branch under `on.push.branches` |
| Secret not found | Secret missing in GitHub | Add it in repository settings |
| AWS access denied | IAM policy missing permissions | Update role/policy |
| ECR login failed | AWS credentials not configured | Configure AWS before ECR login |
| Docker push denied | ECR repo missing or permission issue | Create repo and add ECR permissions |

## 20. Class Summary

```text
GitHub Actions is CI/CD inside GitHub.
Workflow files live in .github/workflows.
push runs after commit or merge.
pull_request validates before merge.
workflow_dispatch runs manually.
GITHUB_TOKEN is created automatically.
Secrets store sensitive values.
AWS login should use OIDC IAM Role in production.
ECR login happens after AWS credentials are configured.
```

## 21. Student Homework

1. Take a screenshot of the workflow YAML file in GitHub.
2. Take a screenshot of automatic workflow run after push.
3. Take a screenshot of manual workflow run.
4. Explain `push`, `pull_request`, and `workflow_dispatch`.
5. Explain where secrets are stored.
6. Explain AWS access keys vs OIDC IAM Role.
7. Explain ECR login flow.

## 22. Interview Questions

### Q1. What is GitHub Actions?

GitHub Actions is a CI/CD automation service built into GitHub.

### Q2. Where are workflow files stored?

Inside `.github/workflows`.

### Q3. What is the difference between `push` and `pull_request`?

`push` runs after code is pushed. `pull_request` runs before code is merged.

### Q4. How do we run a workflow manually?

Use `workflow_dispatch`.

### Q5. What is a runner?

A runner is the machine where GitHub Actions jobs execute.

### Q6. What is `GITHUB_TOKEN`?

It is an automatic temporary token created by GitHub for workflow runs.

### Q7. Where should AWS keys be stored?

In GitHub Secrets if using access keys. In production, prefer OIDC IAM Role.

### Q8. What is ECR?

Amazon ECR is a container registry used to store Docker images in AWS.

### Q9. Why use OIDC?

OIDC avoids storing long-term AWS access keys in GitHub.

### Q10. What happens when code is merged into `main`?

GitHub creates a new commit on `main`, so a workflow configured for push on `main` runs automatically.
