# Day 5: Jenkins vs GitHub Actions for Real Projects and Production

## Class Goal

By the end of Day 5, students should understand:

- The difference between Jenkins and GitHub Actions
- When to use Jenkins
- When to use GitHub Actions
- Why GitHub Actions is usually preferred for modern production projects when code is hosted in GitHub
- How to choose a CI/CD tool based on project type, team size, security, cloud usage, and maintenance effort
- How production CI/CD pipelines are designed with approvals, secrets, environments, and rollback planning

## 1. Quick Revision

Jenkins and GitHub Actions both solve the same basic problem:

```text
Take code from Git
Run checks
Build application
Package artifact
Deploy to an environment
```

But they solve it differently.

```text
Jenkins = external automation server
GitHub Actions = automation built into GitHub
```

## 2. Jenkins in Simple Words

Jenkins is an open-source automation server.

A team installs Jenkins on a server, configures plugins, creates jobs, stores credentials, connects agents, and runs pipelines.

Typical Jenkins setup:

```text
GitHub/GitLab/Bitbucket
        |
        | webhook
        v
Jenkins Controller
        |
        v
Jenkins Agent
        |
        v
Build / Test / Docker / Deploy
```

Jenkins is powerful, but the team owns the platform.

That means the team must manage:

- Jenkins server
- Plugin updates
- Security patches
- Backups
- Credentials
- Agents
- Disk space
- Java version
- Jenkins upgrades
- Network access
- Build queue and scaling

## 3. GitHub Actions in Simple Words

GitHub Actions is CI/CD built into GitHub.

A team creates workflow YAML files inside the repository:

```text
.github/workflows/*.yml
```

Typical GitHub Actions flow:

```text
Developer pushes code to GitHub
        |
        v
GitHub detects event
        |
        v
GitHub Actions starts runner
        |
        v
Workflow runs build/test/deploy steps
```

If using GitHub-hosted runners, GitHub manages the runner machine.

The team mainly manages:

- Workflow YAML
- Secrets and variables
- Environment approvals
- Cloud IAM permissions
- Branch protection rules
- Deployment strategy

## 4. Main Difference Table

| Area | Jenkins | GitHub Actions |
| --- | --- | --- |
| Platform | Separate CI/CD server | Built into GitHub |
| Setup | Install and configure Jenkins | Add YAML file in repository |
| Pipeline file | `Jenkinsfile` | `.github/workflows/*.yml` |
| Trigger | Webhook, polling, manual build | GitHub events directly |
| Runner | Jenkins agent | GitHub-hosted or self-hosted runner |
| Plugins | Jenkins plugins | GitHub Marketplace actions |
| Secrets | Jenkins credentials store | GitHub Secrets, Variables, Environments |
| Cloud auth | IAM user/role on agent, credentials plugin | OIDC IAM role or secrets |
| Maintenance | High | Low for GitHub-hosted runners |
| Best fit | Legacy/custom enterprise CI/CD | GitHub-based modern CI/CD |

## 5. When Should We Use Jenkins?

Use Jenkins when the project needs heavy customization or already has Jenkins infrastructure.

Good Jenkins use cases:

### 1. Existing company already uses Jenkins

If a company already has many Jenkins pipelines, agents, credentials, and approvals, it may continue using Jenkins.

Migration may take time and cost.

### 2. Code is not mainly in GitHub

Jenkins works well with many source control systems:

```text
GitHub
GitLab
Bitbucket
Azure Repos
Self-hosted Git
SVN
```

If the organization is not GitHub-centered, Jenkins may be more flexible.

### 3. Highly customized build agents

Some projects need special build machines:

```text
Large memory servers
GPU machines
Special operating systems
Internal-only network access
Licensed tools
Private build hardware
```

Jenkins agents can be customized deeply.

### 4. Complex legacy pipelines

Old enterprise pipelines may include:

```text
Manual approval plugins
Internal deployment scripts
Custom Groovy shared libraries
Legacy artifact repositories
Internal change-management systems
```

Jenkins can handle these, especially if already implemented.

### 5. Internal network deployments

If deployment targets are only inside private corporate networks, Jenkins running inside that network may be easier.

Example:

```text
Private data center
Internal servers
No public internet access
On-prem Kubernetes
Internal artifact server
```

### 6. Full control is required

Some teams want full control over:

```text
CI/CD server version
Installed plugins
Network routing
Agent images
Disk layout
Build cache
Security scanning tools
```

Jenkins gives that control, but with more maintenance.

## 6. When Should We Use GitHub Actions?

Use GitHub Actions when the code is in GitHub and the team wants a simple, secure, cloud-friendly CI/CD system.

Good GitHub Actions use cases:

### 1. Code is hosted in GitHub

This is the strongest reason.

GitHub Actions directly understands GitHub events:

```text
push
pull_request
merge
release
tag
issue
schedule
manual run
```

No separate webhook setup is required for basic workflows.

### 2. Modern cloud deployments

GitHub Actions works very well with cloud platforms:

```text
AWS
Azure
GCP
Docker Hub
Kubernetes
Terraform
ECR
ECS
EKS
Lambda
S3
CloudFront
```

For AWS, GitHub Actions can use OIDC IAM roles, which avoids long-term AWS keys.

### 3. Production deployment from GitHub

For production, GitHub Actions gives strong built-in controls:

```text
Branch protection
Required status checks
Environment approvals
Deployment history
Secrets per environment
OIDC authentication
Audit logs
Manual workflow_dispatch
Rollback workflows
```

### 4. Less maintenance

With GitHub-hosted runners, teams do not need to maintain CI servers.

No need to manage:

```text
Jenkins server upgrades
Plugin compatibility
Java issues
Agent connectivity
Controller disk cleanup
Backup of Jenkins home
```

### 5. Faster onboarding

A student or new developer can understand GitHub Actions quickly because workflow files live inside the repo.

Example:

```text
Open repository
Open .github/workflows
Read YAML
Understand CI/CD steps
```

### 6. Pull request checks are natural

GitHub Actions integrates directly with pull requests.

Example:

```text
Developer opens PR
Tests run automatically
GitHub blocks merge if tests fail
Reviewer approves
Merge to main
Deployment workflow starts
```

This is very useful in production teams.

## 7. Why GitHub Actions Is Mainly Preferred for Modern Production

For modern projects where the repository is in GitHub, GitHub Actions is usually preferred for production because it reduces platform maintenance and improves security integration.

### Reason 1: It is built into the source code platform

Production deployments start from source code changes.

Since code, pull requests, reviews, branch protection, releases, and Actions are all inside GitHub, the workflow is simpler.

```text
Code review and CI/CD live in the same platform.
```

### Reason 2: No separate Jenkins server to maintain

In production, maintaining Jenkins is real work.

A Jenkins admin must handle:

```text
Server patching
Jenkins upgrades
Plugin upgrades
Backup
Agent scaling
Security hardening
Credential cleanup
Disk cleanup
```

GitHub-hosted runners remove much of this operational burden.

### Reason 3: Better cloud security with OIDC

With GitHub Actions and AWS OIDC:

```text
No permanent AWS access key is stored in GitHub.
GitHub receives temporary AWS credentials only during workflow execution.
AWS IAM controls exactly which repo/branch/environment can assume the role.
```

This is better than storing long-term access keys in Jenkins or GitHub secrets.

### Reason 4: Environment protection is built in

GitHub Environments can protect production deployments.

Example:

```text
main branch workflow completes build
Deploy to staging automatically
Deploy to production waits for approval
Only selected reviewers can approve production
Production secrets are available only after approval
```

This is very useful in real companies.

### Reason 5: Branch protection and required checks

Production teams can block bad code from entering `main`.

Example branch rule:

```text
Require pull request before merge
Require status checks to pass
Require approval from reviewer
Require conversation resolution
Block force push
```

GitHub Actions becomes part of the merge gate.

### Reason 6: Workflow is version-controlled with code

The workflow YAML is stored in Git.

Benefits:

```text
Pipeline changes are reviewed in pull requests
History is visible
Rollback is easy
Team can discuss CI/CD changes
```

### Reason 7: Easy audit and visibility

GitHub shows:

```text
Who triggered workflow
Which commit was deployed
Which environment was deployed
Who approved production
What logs were generated
```

This helps production troubleshooting.

## 8. Important Production Note

Do not say GitHub Actions is always better than Jenkins.

Better statement:

```text
For modern GitHub-based cloud projects, GitHub Actions is usually preferred for production.
For legacy, highly customized, or non-GitHub enterprise setups, Jenkins can still be the better choice.
```

## 9. Project-Based Decision Guide

| Project Type | Recommended Tool | Reason |
| --- | --- | --- |
| Small GitHub project | GitHub Actions | Fast setup and no CI server maintenance |
| GitHub repo deploying to AWS | GitHub Actions | OIDC IAM role, ECR/ECS/EKS actions, branch protection |
| Startup web app on GitHub | GitHub Actions | Simple CI/CD, lower admin overhead |
| Microservices in GitHub | GitHub Actions | Matrix builds, reusable workflows, repo-level automation |
| Legacy enterprise CI/CD | Jenkins | Existing pipelines and plugins may already exist |
| On-prem data center deployment | Jenkins | Internal network access may be easier |
| Multi-SCM organization | Jenkins | Works across many repository systems |
| Heavy custom build hardware | Jenkins | Custom agents are easier to control |
| Highly regulated GitHub Enterprise project | GitHub Actions or Jenkins | Depends on compliance and runner strategy |

## 10. Example: Node.js Web App on GitHub

Recommended: GitHub Actions.

Reason:

```text
Code is in GitHub
PR checks are needed
Deployment can use GitHub Environments
No need to manage Jenkins server
```

Flow:

```text
pull_request -> npm test
push to main -> build Docker image -> deploy staging
manual approval -> deploy production
```

## 11. Example: Internal Banking App on Private Network

Possible recommendation: Jenkins.

Reason:

```text
Deployment target is inside private network
Company may already have Jenkins agents inside data center
Approval process may be integrated with internal tools
```

But if the company uses GitHub Enterprise with self-hosted runners inside the private network, GitHub Actions can also work.

## 12. Example: AWS ECR/ECS Production Deployment

Recommended: GitHub Actions if code is in GitHub.

Reason:

```text
GitHub Actions can use OIDC to assume AWS IAM role
No AWS keys need to be stored
ECR login action is available
Production deployment can require environment approval
```

Flow:

```text
PR opened -> test
PR merged to main -> build image
Login to ECR -> push image
Deploy to ECS staging
Approval -> deploy production
```

## 13. Production GitHub Actions Design

A production workflow should usually include:

```text
pull_request checks
push to main build
artifact or Docker image creation
security scan
environment-based secrets
manual approval for production
rollback option
clear logs
least-privilege permissions
```

Example production strategy:

```text
Feature branch
  -> Pull request
  -> Run tests
  -> Merge to main
  -> Build Docker image
  -> Push image to ECR
  -> Deploy to staging
  -> Manual approval
  -> Deploy to production
```

## 14. Production Security Best Practices

Use these rules:

1. Use OIDC instead of long-term AWS keys.
2. Give minimum IAM permissions.
3. Use GitHub Environments for production secrets.
4. Require approval before production deploy.
5. Protect the `main` branch.
6. Do not print secrets in logs.
7. Pin important action versions.
8. Review workflow changes like application code.
9. Separate staging and production roles.
10. Keep deploy and rollback commands clear.

## 15. Jenkins Production Best Practices

If using Jenkins in production:

1. Keep Jenkins updated.
2. Install only required plugins.
3. Use matrix authorization or role-based access.
4. Use credentials store, never hardcode secrets.
5. Use agents instead of building everything on controller.
6. Back up Jenkins home.
7. Monitor disk space and build logs.
8. Use pipeline as code with `Jenkinsfile`.
9. Restrict script approval carefully.
10. Use IAM roles where possible instead of static AWS keys.


```text
Jenkins is powerful but needs maintenance.
GitHub Actions is simpler when the project is already in GitHub.
For modern production GitHub projects, GitHub Actions is preferred because it gives CI/CD, pull request checks, secrets, approvals, OIDC cloud login, and deployment history in one place.
```

## 17. Interview Questions

### Q1. What is the main difference between Jenkins and GitHub Actions?

Jenkins is a separate automation server. GitHub Actions is CI/CD built into GitHub.

### Q2. When should we use Jenkins?

Use Jenkins for legacy pipelines, custom agents, on-prem environments, multi-SCM projects, or when the organization already has Jenkins infrastructure.

### Q3. When should we use GitHub Actions?

Use GitHub Actions when code is in GitHub, the team wants simple CI/CD, pull request checks, cloud deployment, environment approvals, and lower maintenance.

### Q4. Why is GitHub Actions preferred for modern production GitHub projects?

Because it integrates with pull requests, branch protection, secrets, environments, OIDC cloud authentication, deployment history, and hosted runners without requiring a separate Jenkins server.

### Q5. Is GitHub Actions always better than Jenkins?

No. Jenkins can still be better for legacy, on-prem, highly customized, or non-GitHub projects.

## 18. Homework

1. A small startup web app in GitHub: Jenkins or GitHub Actions? Why?
2. A private banking app deployed inside an internal data center: Jenkins or GitHub Actions? Why?
3. A Docker app deployed to AWS ECR/ECS from GitHub: Jenkins or GitHub Actions? Why?
4. Why is OIDC better than storing AWS access keys?
5. Why should production deployment require approval?
