# Glossary — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Project-Specific Terms

| Term | Definition |
|------|-----------|
| Atomic Workflow | A workflow that performs a single, focused task |
| Orchestration Workflow | A workflow that chains multiple atomic workflows |
| Service Ownership | Pattern where services own their deployment configuration (appspec, etc.) |
| Reusable Workflow | A workflow with `workflow_call` trigger that can be called from other repos |

---

## GitHub Actions Terms

| Term | Definition |
|------|-----------|
| Action | A reusable unit of code that performs a specific task |
| Artifact | Files produced by a workflow (e.g., build outputs, logs) |
| Composite Action | An action composed of multiple steps |
| Job | A set of steps that execute on the same runner |
| Matrix | Strategy to run a job with different configurations |
| Runner | A server that runs workflow jobs (GitHub-hosted or self-hosted) |
| Step | An individual task within a job |
| Workflow | An automated process defined in YAML |
| workflow_call | Trigger type for reusable workflows |
| workflow_dispatch | Trigger type for manual execution |

---

## AWS Terms

| Term | Definition |
|------|-----------|
| Blue/Green Deployment | Deployment strategy with two environments for zero-downtime |
| CodeDeploy | AWS service for automated deployments |
| ECR | Elastic Container Registry - Docker image storage |
| ECS | Elastic Container Service - container orchestration |
| IAM | Identity and Access Management |
| OIDC | OpenID Connect - authentication protocol for GitHub→AWS |
| Task Definition | ECS blueprint for running containers |
| Target Group | ALB routing destination for ECS tasks |

---

## CI/CD Terms

| Term | Definition |
|------|-----------|
| CI | Continuous Integration - automated testing on every commit |
| CD | Continuous Deployment - automated deployment to production |
| Pipeline | Sequence of stages from code to production |
| Semantic Versioning | Version format: MAJOR.MINOR.PATCH |
| Conventional Commits | Commit message format: `type(scope): description` |

---

## Abbreviations

| Abbr. | Meaning |
|-------|---------|
| ALB | Application Load Balancer |
| ARN | Amazon Resource Name |
| CI | Continuous Integration |
| CD | Continuous Deployment |
| ECR | Elastic Container Registry |
| ECS | Elastic Container Service |
| IAM | Identity and Access Management |
| OIDC | OpenID Connect |
| PR | Pull Request |
| SHA | Secure Hash Algorithm (commit identifier) |

---

## Commit Type Prefixes

| Prefix | Purpose | Version Impact |
|--------|---------|----------------|
| `feat:` | New feature | Minor bump |
| `fix:` | Bug fix | Patch bump |
| `perf:` | Performance improvement | Patch bump |
| `docs:` | Documentation only | No bump |
| `style:` | Code style (formatting) | No bump |
| `refactor:` | Code refactoring | No bump |
| `test:` | Adding tests | No bump |
| `chore:` | Maintenance tasks | No bump |
| `build:` | Build system changes | No bump |
| `ci:` | CI configuration | No bump |
| `BREAKING CHANGE:` | Breaking API change | Major bump |

---

## Further Documentation

[Overview](01-overview.md) | [Architecture](02-architecture.md) | [Tech Stack](03-tech-stack.md)
