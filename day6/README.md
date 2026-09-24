# Day 6: End-to-End GitHub Actions Production CI/CD Mini Project

## Class Goal

By the end of Day 6, students should understand how a real production-style GitHub Actions pipeline is designed from pull request to production deployment.

Students will learn:

- CI checks on pull request
- Build workflow on push to `main`
- Docker image build and tag strategy
- AWS authentication using OIDC IAM role
- Amazon ECR login and image push
- Staging and production environment strategy
- Manual approval before production
- Rollback planning
- What to store in secrets, variables, and AWS IAM

## 1. Production CI/CD Flow

A good production flow is not just one command. It is a controlled path from code change to release.

```text
Developer creates feature branch
        |
        v
Developer opens pull request
        |
        v
GitHub Actions runs CI checks
        |
        v
Reviewer approves pull request
        |
        v
Pull request is merged to main
        |
        v
GitHub Actions builds Docker image
        |
        v
Image is pushed to Amazon ECR
        |
        v
Application deploys to staging
        |
        v
Manual approval
        |
        v
Application deploys to production
```

## 2. Why We Separate Pull Request and Main Workflows

Pull request workflow answers:

```text
Is this code safe to merge?
```

Main branch workflow answers:

```text
The code is merged. Should we build and deploy it?
```

Common pattern:

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| CI workflow | `pull_request` | Run tests before merge |
| Build/deploy workflow | `push` to `main` | Build artifact and deploy |
| Manual workflow | `workflow_dispatch` | Run deployment manually if needed |

## 3. Required Repository Files

A simple application repository usually contains:

```text
app.py or source code
requirements.txt or package.json
Dockerfile
.github/workflows/ci.yml
.github/workflows/deploy.yml
README.md
```

This training repository already has a sample Python app:

```text
day3/projects/multibranch-ecr-demo/app.py
```

## 4. CI Workflow for Pull Requests

Example CI workflow:

```yaml
name: CI Checks

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r day3/projects/multibranch-ecr-demo/requirements.txt

      - name: Syntax check
        run: python -m py_compile day3/projects/multibranch-ecr-demo/app.py
```

Teaching point:

```text
Pull request workflow should be fast. It should catch errors before code enters main.
```

## 5. Build and Deploy Workflow for Main

Production deployment should run only from trusted branches, usually `main`.

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

This means:

```text
Merge to main -> automatic build/deploy
Manual run -> possible if team needs controlled redeployment
```

## 6. Docker Image Tag Strategy

A Docker image needs a tag.

Bad tag for production-only tracking:

```text
latest
```

Better tags:

```text
Commit SHA
Git tag
Release version
Build number
Date + SHA
```

Recommended for class:

```yaml
IMAGE_TAG: ${{ github.sha }}
```

Why commit SHA is useful:

```text
Every image maps to exact source code.
Rollback and troubleshooting become easier.
```

Example image:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app:5bcb48c...
```

## 7. AWS Authentication for Production

For production, prefer OIDC IAM Role.

Do not store long-term AWS keys if OIDC is available.

Flow:

```text
GitHub Actions requests OIDC token
AWS verifies repository and branch conditions
AWS allows workflow to assume IAM role
Workflow receives temporary credentials
Temporary credentials expire automatically
```

Workflow permissions:

```yaml
permissions:
  id-token: write
  contents: read
```

AWS configure step:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v6.1.0
  with:
    role-to-assume: arn:aws:iam::123456789012:role/github-actions-ecr-role
    aws-region: ap-south-1
```

## 8. AWS IAM Role Requirements

The IAM role should allow only required permissions.

For ECR push, the policy usually needs permissions like:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:CompleteLayerUpload
ecr:PutImage
```

Best practice:

```text
Use separate IAM roles for staging and production.
Give each role minimum permissions.
Restrict role trust policy to your GitHub repository and branch/environment.
```

## 9. ECR Login and Push

After AWS credentials are configured, login to ECR:

```yaml
- name: Login to Amazon ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2
```

Build and push:

```yaml
- name: Build and push Docker image
  env:
    REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    IMAGE_TAG: ${{ github.sha }}
  run: |
    docker build -t $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
    docker push $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

## 10. Full Production-Style ECR Workflow Template

Save this as a reference. Do not run until AWS role and ECR repository are ready.

```yaml
name: Production ECR Build

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
    environment: staging

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6.1.0
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-staging-role
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

## 11. GitHub Environments

GitHub Environments help protect deployments.

Create environments:

```text
staging
production
```

Path:

```text
Repository -> Settings -> Environments -> New environment
```

For production, configure:

```text
Required reviewers
Deployment branch rules
Environment secrets
Environment variables
```

Teaching point:

```text
Production secrets should be available only to production deployment jobs, not every workflow.
```

## 12. Staging and Production Separation

Recommended design:

| Environment | Trigger | Approval | AWS Role |
| --- | --- | --- | --- |
| Staging | Push to main | No or optional | staging IAM role |
| Production | Manual approval | Required | production IAM role |

Why separate roles?

```text
If staging workflow is compromised, production permissions are still protected.
```

## 13. Manual Approval for Production

A production job can use:

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production
```

If the production environment has required reviewers, GitHub pauses the job until approval.

Flow:

```text
Build completes
Staging deploy completes
Production job waits
Approver reviews
Approver clicks approve
Production deploy continues
```

## 14. What Should Be Stored Where?

| Value | Store in |
| --- | --- |
| AWS account ID | GitHub Variable or workflow `env` |
| AWS region | GitHub Variable or workflow `env` |
| ECR repository name | GitHub Variable or workflow `env` |
| AWS IAM role ARN | GitHub Environment variable or workflow `env` |
| AWS access key | Avoid in production; use OIDC |
| Database password | GitHub Environment Secret or cloud secret manager |
| Docker image tag | Generated by workflow, usually `github.sha` |
| Production approval | GitHub Environment protection rule |

## 15. Rollback Strategy

Production pipelines must have rollback planning.

Rollback means returning to a previous working version.

Docker image rollback example:

```text
Current production image: my-app:bad-sha
Previous good image: my-app:good-sha
Rollback deploys previous good image again
```

Students should remember:

```text
If every Docker image is tagged with commit SHA, rollback becomes easier.
```

## 16. Common Production Mistakes

| Mistake | Risk | Better Practice |
| --- | --- | --- |
| Using only `latest` tag | Hard to know deployed version | Use commit SHA or release version |
| Storing AWS keys in code | Secret leak | Use OIDC or GitHub Secrets |
| No production approval | Accidental deployment | Use GitHub Environments |
| Same AWS role for all envs | Too much access | Separate staging and production roles |
| No branch protection | Bad code can enter main | Require PR and checks |
| No rollback plan | Long outage | Keep previous image tags |

## 17. Practical Classroom Demo

Use the existing safe workflow first:

```text
.github/workflows/github-actions-practical.yml
```

Demo steps:

1. Open workflow file.
2. Explain `on.push`, `pull_request`, and `workflow_dispatch`.
3. Commit a README change.
4. Push to `main`.
5. Open Actions tab.
6. Show automatic run.
7. Run workflow manually.
8. Open logs and explain event, branch, SHA, actor.
9. Explain how AWS/ECR steps would be added for production.

## 18. Student Assignment

Students should create a diagram for this flow:

```text
PR -> CI checks -> Merge -> Build Docker image -> Push ECR -> Deploy staging -> Approval -> Deploy production
```

They should also answer:

1. Why do we use `pull_request`?
2. Why do we use `push` on `main`?
3. Why is `workflow_dispatch` useful?
4. Why is OIDC better than AWS access keys?
5. Why should production require approval?
6. Why should Docker images use commit SHA tags?

## 19. Interview Questions

### Q1. What is an end-to-end CI/CD pipeline?

It is an automated flow that takes code from source control, runs checks, builds an artifact, and deploys it to an environment.

### Q2. Why do we run tests before Docker build?

To avoid packaging broken code into a Docker image.

### Q3. Why do we push Docker images to ECR?

ECR stores container images in AWS so services like ECS, EKS, or EC2 can deploy them.

### Q4. What is GitHub Environment approval?

It is a protection rule that pauses deployment until an approved user allows it to continue.

### Q5. What is rollback?

Rollback means deploying a previous known-good version when the current version has a problem.

## 20. Real GitHub Actions to ECR Demo Added in This Repository

This repository now has a real GitHub Actions workflow that pushes a Docker image to Amazon ECR.

Workflow file:

```text
.github/workflows/github-actions-practical.yml
```

Day 6 Dockerfile:

```text
day6/Dockerfile
```

The workflow runs on:

```text
push to main
manual workflow_dispatch
pull_request to main for CI only
```

Important:

```text
The ECR push job does not run on pull_request.
It runs only on push to main or manual workflow_dispatch.
```

This prevents feature branch pull requests from pushing images to production-style registries.

## 21. Real AWS/ECR Values Used for Demo

AWS account:

```text
909969506392
```

AWS region:

```text
ap-south-1
```

ECR repository:

```text
github-actions-day6-demo
```

ECR image URI format:

```text
909969506392.dkr.ecr.ap-south-1.amazonaws.com/github-actions-day6-demo:<commit-sha>
```

GitHub Actions IAM role:

```text
arn:aws:iam::909969506392:role/GitHubActionsDay6EcrPushRole
```

## 22. What Was Created in AWS

For GitHub Actions to push to ECR, AWS must trust GitHub.

Created AWS OIDC provider:

```text
arn:aws:iam::909969506392:oidc-provider/token.actions.githubusercontent.com
```

Created IAM role:

```text
GitHubActionsDay6EcrPushRole
```

The role trust policy allows only this repository and branch:

```text
repo:bharisagar/Jenkins-Githubactions:ref:refs/heads/main
```

This means GitHub Actions from another repository or another branch cannot use this role.

## 23. Why We Use OIDC Instead of AWS Keys

Old method:

```text
Create AWS access key
Store AWS_ACCESS_KEY_ID in GitHub Secrets
Store AWS_SECRET_ACCESS_KEY in GitHub Secrets
Workflow uses long-term key
```

Production method:

```text
GitHub requests temporary OIDC token
AWS verifies repository and branch
AWS gives temporary credentials
Credentials expire automatically
No permanent AWS key is stored in GitHub
```

This is safer for production because there is no long-term AWS secret sitting inside GitHub.

## 24. Workflow Jobs Explained

The workflow has two jobs.

### Job 1: Basic CI Demo

```text
basic-ci
```

It does:

```text
Checkout repository
Print event, branch, SHA, actor
Verify README and Dockerfile
Compile Python sample
Explain next production step
```

This job runs for:

```text
push
pull_request
workflow_dispatch
```

### Job 2: Build, Tag, and Push Image to ECR

```text
ecr-build-and-push
```

It does:

```text
Checkout repository
Configure AWS credentials using OIDC
Login to Amazon ECR
Build Docker image from day6/Dockerfile
Tag image with GitHub commit SHA
Tag image as latest
Push both tags to ECR
Show ECR image details
```

This job is skipped for pull requests:

```yaml
if: github.event_name != 'pull_request'
```

## 25. Dockerfile Used by GitHub Actions

File:

```text
day6/Dockerfile
```

Content summary:

```text
Use python:3.12-slim
Set /app as workdir
Copy requirements.txt from Day 3 sample app
Install Flask dependency
Copy app.py from Day 3 sample app
Expose port 5000
Run python app.py
```

The workflow builds using repository root as context:

```bash
docker build -f day6/Dockerfile -t <ecr-image-uri>:<commit-sha> .
```

We use root context because the Dockerfile copies files from:

```text
day3/projects/multibranch-ecr-demo/
```

## 26. Build, Tag, and Push Commands in GitHub Actions

Inside the workflow, GitHub Actions runs:

```bash
docker build -f day6/Dockerfile -t $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
docker tag $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $REGISTRY/$ECR_REPOSITORY:latest
docker push $REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
docker push $REGISTRY/$ECR_REPOSITORY:latest
```

`IMAGE_TAG` comes from:

```yaml
IMAGE_TAG: ${{ github.sha }}
```

Why this is important:

```text
Every image tag points to the exact Git commit that built it.
This helps debugging, rollback, and audit.
```

## 27. How to Show Live Demo to Students

Step 1: Open workflow file.

```text
.github/workflows/github-actions-practical.yml
```

Step 2: Explain triggers.

```text
push to main -> CI plus ECR push
pull_request to main -> CI only
workflow_dispatch -> manual CI plus ECR push
```

Step 3: Push a small commit or run manually.

Manual path:

```text
GitHub repository
  -> Actions
  -> GitHub Actions Practical
  -> Run workflow
```

Push path:

```bash
git commit --allow-empty -m "demo github actions ecr push"
git push origin main
```

Step 4: Open Actions run and explain jobs.

```text
basic-ci
  -> proves code is checked and valid

ecr-build-and-push
  -> proves image is built and pushed to ECR
```

Step 5: Open ECR in AWS Console.

```text
AWS Console
  -> ECR
  -> Private repositories
  -> github-actions-day6-demo
  -> Images
```

Students should see:

```text
latest
<commit-sha>
```

## 28. What Students Should Understand from This Demo

This is the real production idea:

```text
GitHub commit becomes Docker image.
Docker image is stored in ECR.
Deployment tools can now deploy that image to ECS, EKS, EC2, or Kubernetes.
```

The key DevOps learning is not only Docker push. The key learning is traceability:

```text
Git commit SHA = Docker image tag = deployable artifact version
```

## 29. Important Production Notes

For classroom, pushing `latest` is useful to understand tags.

For production, do not rely only on `latest`.

Better production approach:

```text
Always push commit SHA tag.
Optionally push latest for convenience.
Deploy using commit SHA or release version.
```

Also use:

```text
Branch protection
Pull request checks
Environment approvals
Separate staging and production roles
Least-privilege IAM permissions
Rollback workflow
```

## 30. Final Day 6 Demo Summary

Use this explanation in class:

```text
Earlier we manually pushed a Docker image to ECR from our laptop.
Now GitHub Actions does the same from GitHub.

The workflow checks the code, logs in to AWS using OIDC, logs in to ECR, builds the Docker image, tags it with the commit SHA, pushes it to ECR, and prints image details.

This is how modern production CI/CD starts: code change creates a traceable container image.
```
