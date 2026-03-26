# .ai Documentation — github-workflows

**Purpose**: Reusable CI/CD Workflows for all tec42 Services  
**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Quick Navigation

### Getting Started
- [Quick Start](context/00-quick-start.md) — Get started in 5 minutes
- [Overview](context/01-overview.md) — What are these workflows?
- [Architecture](context/02-architecture.md) — Workflow design & orchestration

### Core Documentation
- [Tech Stack](context/03-tech-stack.md) — GitHub Actions, AWS, Tools
- [Principles](context/04-principles.md) — Workflow best practices
- [Project Structure](context/06-project-structure.md) — File organization
- [Development](context/07-development.md) — How to create new workflows

### Reference
- [Troubleshooting](context/10-troubleshooting.md) — Common issues
- [Glossary](context/99-glossary.md) — Terms & Concepts

---

## Scenarios

### CI Testing
- [CI Scenarios](scenarios/ci-scenarios.md) — Test execution patterns

### Release & Deployment
- [Release ECR Scenarios](scenarios/release-ecr-scenarios.md) — Versioning & ECR push
- [Terraform Scenarios](scenarios/terraform-scenarios.md) — Infrastructure deployment
- [ECS CodeDeploy Scenarios](scenarios/ecs-codedeploy-scenarios.md) — Blue/Green deployments
- [Orchestration Scenarios](scenarios/orchestration-scenarios.md) — Full pipeline

---

## Workflows (Development Guides)

- [Add Workflow](workflows/add-workflow.md) — Create new reusable workflow
- [Add Composite Action](workflows/add-composite-action.md) — Create reusable action

---

## Quick Links

### For Service Teams
- **Deploy to ECR?** → [Release ECR Scenarios](scenarios/release-ecr-scenarios.md)
- **ECS CodeDeploy?** → [ECS CodeDeploy Scenarios](scenarios/ecs-codedeploy-scenarios.md)
- **Full pipeline?** → [Orchestration Scenarios](scenarios/orchestration-scenarios.md)

### For Workflow Developers
- **Create workflow?** → [Add Workflow](workflows/add-workflow.md)
- **Troubleshooting?** → [Troubleshooting](context/10-troubleshooting.md)

---

## File Structure

```
.ai/
├── README.md                           # This file
├── context/
│   ├── 00-quick-start.md              # Getting started
│   ├── 01-overview.md                 # Project overview
│   ├── 02-architecture.md             # System architecture
│   ├── 03-tech-stack.md               # Technologies
│   ├── 04-principles.md               # Best practices
│   ├── 06-project-structure.md        # File organization
│   ├── 07-development.md              # Development guide
│   ├── 10-troubleshooting.md          # Common issues
│   └── 99-glossary.md                 # Terminology
├── scenarios/
│   ├── ci-scenarios.md                # CI testing
│   ├── release-ecr-scenarios.md       # Release & ECR
│   ├── terraform-scenarios.md         # Infrastructure
│   ├── ecs-codedeploy-scenarios.md    # ECS deployment
│   └── orchestration-scenarios.md     # Full pipeline
└── workflows/
    ├── add-workflow.md                # Create workflow
    └── add-composite-action.md        # Create action
```
- ✅ Reusable GitHub Actions workflows
- ✅ CI/CD pipeline design
- ✅ Release automation (release-it)
- ✅ Docker build & ECR deployment
- ✅ ECS CodeDeploy Blue/Green
- ✅ Terraform infrastructure deployment

**NOT documented here**:
- ❌ Service-specific workflows (see `identity/.github/workflows/`, `render/.github/workflows/`)
- ❌ Infrastructure code (see `tf-aws-infra/`)
- ❌ Application code (see `identity/app/`, `render/app/`)

---

## 🚀 Available Workflows

### **1. CI Testing**
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-ci-docker.yml@main
```
**Purpose**: Lint, type-check, build, test with Docker Compose

---

### **2. Release & ECR Push**
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@main
```
**Purpose**: Semantic versioning, Docker build, ECR push

---

### **3. Terraform Deployment** (Coming Soon)
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-terraform-deploy.yml@main
```
**Purpose**: Deploy static infrastructure (RDS, Redis, ECS Service)

---

### **4. ECS CodeDeploy** (Coming Soon)
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-ecs-codedeploy.yml@main
```
**Purpose**: Blue/Green deployment via CodeDeploy

---

### **5. Master Orchestration** (Coming Soon)
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-service-deployment.yml@main
```
**Purpose**: Complete deployment pipeline (Release → Terraform → CodeDeploy)

---

## 🔗 Repository Structure

```
github-workflows/
├── .github/workflows/
│   ├── reusable-ci-docker.yml              ✅ CI Testing
│   ├── reusable-release-ecr.yml            ✅ Release & ECR
│   ├── reusable-terraform-deploy.yml       🚧 In Development
│   ├── reusable-ecs-codedeploy.yml         🚧 In Development
│   └── reusable-service-deployment.yml     🚧 In Development
├── .ai/                                     ✅ Documentation
├── shared/                                  ✅ Shared configs
├── examples/                                ✅ Usage examples
├── README.md
├── CHANGELOG.md
└── TASKS.md                                 ✅ Implementation tasks
```

---

## 🎓 Learning Path

### **Beginner**
1. Read [Quick Start](context/00-quick-start.md)
2. Read [Overview](context/01-overview.md)
3. Try [ECR Deployment Example](../examples/deploy-ecr-with-release.md)

### **Intermediate**
1. Read [Architecture](context/02-architecture.md)
2. Understand [Workflow Chaining](context/02-architecture.md#workflow-chaining)
3. Implement service deployment

### **Advanced**
1. Read [Development Guide](context/07-development.md)
2. Create custom workflows
3. Contribute to this repository

---

**Note**: This documentation follows the AI Documentation Templates standard from `ai-tools/templates/`.
