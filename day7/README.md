# Day 7: Revision, Troubleshooting, Interview Preparation, and Final CI/CD Checklist

## Class Goal

By the end of Day 7, students should be able to explain the complete CI/CD journey using Jenkins and GitHub Actions.

Students should be able to answer:

- What problem CI/CD solves
- How Jenkins works
- How GitHub Actions works
- When to use Jenkins
- When to use GitHub Actions
- Why GitHub Actions is preferred for modern GitHub-based production projects
- How triggers work
- How secrets are stored
- How AWS and ECR login works
- How to troubleshoot common CI/CD failures

## 1. Full Course Revision

The course covered:

| Day | Topic |
| --- | --- |
| Day 1 | Jenkins introduction, CI/CD, architecture, installation |
| Day 2 | Jenkins security, credentials, logs, freestyle job |
| Day 3 | Jenkins plugins, pipeline, multibranch, Docker/ECR stages |
| Day 4 | GitHub Actions workflow syntax, triggers, secrets, AWS/ECR login |
| Day 5 | Jenkins vs GitHub Actions decision-making for projects and production |
| Day 6 | End-to-end GitHub Actions production-style CI/CD mini project |
| Day 7 | Revision, troubleshooting, interview preparation |

## 2. CI/CD Revision

CI/CD automates software delivery.

```text
Code change
  -> Test
  -> Build
  -> Package
  -> Deploy
  -> Verify
```

CI helps developers find errors early.

CD helps teams release faster and more safely.

## 3. Jenkins Revision

Jenkins is an open-source automation server.

Important Jenkins concepts:

```text
Controller
Agent
Job
Workspace
Plugin
Credential
Jenkinsfile
Pipeline
Multibranch pipeline
Console output
```

Jenkins is powerful because it can connect many tools.

Jenkins requires maintenance because the team owns the server and plugins.

## 4. GitHub Actions Revision

GitHub Actions is CI/CD built into GitHub.

Important GitHub Actions concepts:

```text
Workflow
Event
Job
Step
Runner
Action
Secret
Variable
Environment
GITHUB_TOKEN
OIDC
```

Workflow files live in:

```text
.github/workflows/
```

## 5. Trigger Revision

| Trigger | Meaning | Use Case |
| --- | --- | --- |
| `push` | Runs after code is pushed | Build/deploy from main |
| `pull_request` | Runs when PR is opened/updated | Test before merge |
| `workflow_dispatch` | Manual run button | Manual deployment or demo |
| `schedule` | Cron-based run | Nightly jobs or cleanup |
| `release` | Runs on release events | Production release pipeline |

Key teaching point:

```text
Merge to main creates a push event on main, so a push workflow runs automatically after merge.
```

## 6. Jenkins vs GitHub Actions Final Summary

| Question | Jenkins | GitHub Actions |
| --- | --- | --- |
| Is it built into GitHub? | No | Yes |
| Needs separate server? | Yes | No, if using hosted runners |
| Best for GitHub PR checks? | Possible | Excellent |
| Best for legacy custom systems? | Excellent | Possible with self-hosted runners |
| Maintenance effort | Higher | Lower |
| Production cloud OIDC support | Possible but custom | Strong built-in pattern |
| Beginner setup speed | Slower | Faster |

Final simple statement:

```text
Use GitHub Actions for modern GitHub-based production projects.
Use Jenkins when you need legacy support, deep customization, on-prem agents, or existing Jenkins investment.
```

## 7. Why GitHub Actions for Modern Production

GitHub Actions is commonly preferred for production when code is in GitHub because:

1. CI/CD config lives with code.
2. Pull request checks are built into GitHub.
3. Branch protection can require workflow success.
4. GitHub Environments can require production approval.
5. Secrets can be scoped by environment.
6. OIDC allows temporary AWS credentials.
7. GitHub-hosted runners reduce maintenance.
8. Deployment history is visible in GitHub.
9. Workflow logs connect directly to commits and pull requests.
10. Teams avoid managing a separate Jenkins controller.

## 8. When Jenkins Is Still Useful

Jenkins is still useful for:

```text
Legacy enterprise pipelines
Internal data center deployments
Custom build machines
Multi-SCM organizations
Existing Jenkins shared libraries
Complex plugin-based flows
Private networks without GitHub runner access
```

Do not teach Jenkins as outdated. Teach it as powerful but maintenance-heavy.

## 9. Credentials Revision

Never store secrets in code.

| Tool | Secret Store |
| --- | --- |
| Jenkins | Jenkins Credentials |
| GitHub Actions | GitHub Secrets, Variables, Environments |
| AWS | IAM roles, IAM policies, Secrets Manager, Parameter Store |

GitHub Actions automatic token:

```text
GITHUB_TOKEN
```

AWS production recommendation:

```text
Use OIDC IAM role instead of long-term AWS keys.
```

## 10. AWS and ECR Revision

ECR is Amazon Elastic Container Registry.

Docker image push flow:

```text
Configure AWS credentials
Login to ECR
Build Docker image
Tag Docker image
Push Docker image
Deploy image
```

GitHub Actions ECR flow:

```yaml
- uses: aws-actions/configure-aws-credentials@v6.1.0
- uses: aws-actions/amazon-ecr-login@v2
- run: docker build ...
- run: docker push ...
```

## 11. Production Checklist

Before a production pipeline is trusted, check:

| Check | Required? |
| --- | --- |
| Pull request checks | Yes |
| Branch protection on main | Yes |
| Required approvals | Yes |
| Environment separation | Yes |
| OIDC instead of static AWS keys | Recommended |
| Least-privilege IAM | Yes |
| Docker image version tags | Yes |
| Rollback plan | Yes |
| Logs available | Yes |
| Secrets hidden from logs | Yes |

## 12. Troubleshooting GitHub Actions

| Problem | Possible Reason | Fix |
| --- | --- | --- |
| Workflow not visible | YAML not inside `.github/workflows` | Move file to correct folder |
| Workflow not running | Branch filter mismatch | Check `on.push.branches` |
| Manual button missing | `workflow_dispatch` not configured | Add `workflow_dispatch` |
| Code files missing | Checkout step missing | Add `actions/checkout` |
| Secret empty | Secret not created or wrong name | Check GitHub Secrets |
| AWS auth failed | OIDC role/trust policy wrong | Fix IAM role trust policy |
| ECR push failed | Missing ECR permissions | Add required IAM permissions |
| Deployment blocked | Environment approval pending | Approver must approve |
| YAML error | Wrong indentation | Fix spacing and syntax |

## 13. Troubleshooting Jenkins

| Problem | Possible Reason | Fix |
| --- | --- | --- |
| Jenkins not opening | Service/container stopped | Start Jenkins |
| Git clone failed | Wrong credentials | Fix Jenkins credential |
| Build stuck | Agent offline | Check agent connection |
| Docker not found | Docker CLI missing | Install Docker on agent |
| AWS not found | AWS CLI missing | Install AWS CLI on agent |
| Permission denied Docker socket | Jenkins lacks Docker access | Fix agent permissions |
| Plugin missing | Required plugin not installed | Install plugin |
| User cannot build | Permission missing | Update security matrix |

## 14. Final Practical Review

Students should demonstrate:

1. Explain the root `README.md` course structure.
2. Open Day 4 GitHub Actions notes.
3. Open `.github/workflows/github-actions-practical.yml`.
4. Explain `push`, `pull_request`, and `workflow_dispatch`.
5. Push a small README change and show automatic workflow run.
6. Manually run the workflow.
7. Explain where AWS secrets should be stored.
8. Explain why OIDC is preferred.
9. Explain when Jenkins is useful.
10. Explain why GitHub Actions is preferred for modern production GitHub projects.

## 15. Final Interview Questions

### Q1. What is CI/CD?

CI/CD is the practice of automatically testing, building, packaging, and deploying code changes.

### Q2. What is Jenkins?

Jenkins is an open-source automation server used for build, test, and deployment pipelines.

### Q3. What is GitHub Actions?

GitHub Actions is GitHub's built-in CI/CD automation system.

### Q4. Where are GitHub Actions workflows stored?

Inside `.github/workflows` in the repository.

### Q5. What happens when code is merged into `main`?

A new commit is created on `main`, which triggers workflows configured with `push` on `main`.

### Q6. What is `workflow_dispatch`?

It is a manual trigger that shows a Run workflow button in the Actions tab.

### Q7. What is `GITHUB_TOKEN`?

It is a temporary token automatically created by GitHub for workflow runs.

### Q8. What is OIDC in GitHub Actions?

OIDC allows GitHub Actions to request temporary cloud credentials without storing long-term access keys.

### Q9. What is ECR?

Amazon ECR is a container registry for storing Docker images in AWS.

### Q10. When should we use Jenkins?

Use Jenkins for legacy CI/CD, custom agents, on-prem deployments, multi-SCM systems, or when Jenkins is already deeply used.

### Q11. When should we use GitHub Actions?

Use GitHub Actions for GitHub-hosted projects, pull request checks, cloud deployments, environment approvals, and lower-maintenance production workflows.

### Q12. Why is GitHub Actions preferred for modern production GitHub projects?

Because it integrates code, reviews, checks, environments, secrets, OIDC cloud authentication, and deployment logs in one platform while reducing CI/CD server maintenance.

## 16. Final Student Assignment

Students should submit:

1. Screenshot of GitHub Actions workflow run.
2. Screenshot of manual workflow run.
3. Explanation of Jenkins vs GitHub Actions.
4. Project-based recommendation: choose Jenkins or GitHub Actions for three example projects.
5. AWS/ECR login explanation.
6. Production checklist for a GitHub Actions pipeline.
7. Five troubleshooting examples and fixes.

## 17. Closing Summary

Use this final statement:

```text
CI/CD is not only about running commands automatically. A good production pipeline protects code quality, protects secrets, controls deployment, gives visibility, and makes rollback possible.

Jenkins gives deep control and flexibility.
GitHub Actions gives simple, integrated, production-friendly automation for GitHub-based projects.
```
