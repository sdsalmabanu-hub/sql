# Jenkins-Githubactions Course Notes

Beginner-friendly Jenkins and GitHub Actions notes with practical labs for DevOps students.

This repository is a complete 7-day CI/CD learning path. It starts with Jenkins basics, moves into Jenkins pipelines and ECR, then teaches GitHub Actions for modern production CI/CD.

## Repository Goal

Students should understand:

- What CI/CD means
- How Jenkins works
- How GitHub Actions works
- How push, merge, pull request, and manual triggers work
- How credentials and secrets are stored
- How AWS and ECR login works
- When to choose Jenkins
- When to choose GitHub Actions
- Why GitHub Actions is usually preferred for modern GitHub-based production projects

## Folder Structure

| Folder | Content |
| --- | --- |
| [day1](day1/README.md) | Jenkins introduction, CI/CD, architecture, installation, unlock setup, SSH setup |
| [day2](day2/README.md) | Jenkins security, Matrix-based security, credentials, logs, freestyle job, practical |
| [day2/practical/Project1](day2/practical/Project1) | Sample Day 2 Git project for Jenkins freestyle job |
| [day3](day3/README.md) | Jenkins plugins, multibranch pipeline, real CI stages, Docker build/tag/push to ECR |
| [day3/projects/multibranch-ecr-demo](day3/projects/multibranch-ecr-demo) | Sample multibranch pipeline project with Docker and ECR Jenkinsfile |
| [day4](day4/README.md) | GitHub Actions practical: push trigger, pull request trigger, manual trigger, secrets, AWS login, ECR login |
| [day5](day5/README.md) | Jenkins vs GitHub Actions: when to use each tool based on real project and production needs |
| [day6](day6/README.md) | End-to-end GitHub Actions production-style CI/CD mini project with AWS ECR design |
| [day7](day7/README.md) | Revision, troubleshooting, interview preparation, and final CI/CD checklist |
| [.github/workflows/github-actions-practical.yml](.github/workflows/github-actions-practical.yml) | GitHub Actions demo workflow that runs CI and pushes a Docker image to AWS ECR on push/manual trigger |

## 7-Day CI/CD Roadmap

| Day | Topic |
| --- | --- |
| Day 1 | Jenkins introduction, CI/CD, architecture, installation |
| Day 2 | Jenkins security, credentials, logs, freestyle Git job |
| Day 3 | Jenkins plugins, multibranch pipeline, Docker image build/tag/push to ECR |
| Day 4 | GitHub Actions workflow syntax, automatic triggers, manual triggers, secrets, AWS/ECR login |
| Day 5 | Jenkins vs GitHub Actions project-based and production-based decision guide |
| Day 6 | End-to-end GitHub Actions production CI/CD mini project |
| Day 7 | Revision, troubleshooting, interview preparation, final checklist |

## Current Lessons

- [Day 1 Notes](day1/README.md)
- [Day 2 Notes and Practical](day2/README.md)
- [Day 3 Notes and Multibranch Project](day3/README.md)
- [Day 4 GitHub Actions Notes and Practical](day4/README.md)
- [Day 5 Jenkins vs GitHub Actions Production Guide](day5/README.md)
- [Day 6 End-to-End GitHub Actions Mini Project](day6/README.md)
- [Day 7 Revision and Interview Preparation](day7/README.md)

## Current GitHub Actions Practical

This repository includes a classroom GitHub Actions workflow:

```text
.github/workflows/github-actions-practical.yml
```

It demonstrates:

- Automatic trigger on push to `main`
- Pull request validation for `main`
- Manual trigger using `workflow_dispatch`
- GitHub runner environment information
- Repository checkout using `actions/checkout`
- Basic Python validation of the Day 3 sample app
- Real Docker build, image tag, AWS OIDC login, ECR login, and ECR push on push/manual trigger

After this repository is pushed to GitHub, students can open the repository Actions tab and observe the workflow run automatically, then verify the pushed image in Amazon ECR.

## Jenkins or GitHub Actions?

Use Jenkins when the project needs deep customization, legacy pipelines, internal data-center deployment, special build agents, non-GitHub source control, or an existing Jenkins platform.

Use GitHub Actions when the code is hosted in GitHub and the team wants simple, secure, production-friendly CI/CD with pull request checks, branch protection, GitHub Environments, OIDC cloud login, and lower platform maintenance.

For modern production projects hosted in GitHub, GitHub Actions is usually preferred because CI/CD, code review, secrets, approvals, cloud authentication, and deployment visibility are available in one platform without maintaining a separate Jenkins controller.

## How Students Should Use This Repository

1. Open the folder for the current class day.
2. Read the notes before the practical.
3. Run the commands step by step.
4. Take screenshots of the required output.
5. Use the interview questions for revision.
6. Compare Jenkins and GitHub Actions based on real project requirements.
