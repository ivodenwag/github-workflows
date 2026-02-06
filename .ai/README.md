# .ai Documentation - GitHub Workflows

**Purpose**: Reusable CI/CD Workflows for all tec42 Services  
**Last Updated**: 6. Februar 2026  
**Status**: Active Development

---

## 📚 Navigation

### Start Here
- [Quick Start](context/00-quick-start.md) - Get started in 5 minutes
- [Overview](context/01-overview.md) - What are these workflows?
- [Architecture](context/02-architecture.md) - Workflow design & orchestration

### Core Documentation
- [Tech Stack](context/03-tech-stack.md) - GitHub Actions, AWS, Tools
- [Principles](context/04-principles.md) - Workflow best practices
- [Features](context/05-features.md) - What these workflows provide
- [Project Structure](context/06-project-structure.md) - File organization
- [Development](context/07-development.md) - How to create new workflows

### Reference
- [Troubleshooting](context/10-troubleshooting.md) - Common issues
- [Glossary](context/99-glossary.md) - Terms & Concepts

---

## 🎯 Quick Links

### For Developers
- **First time?** → [Quick Start](context/00-quick-start.md)
- **Understanding workflows?** → [Architecture](context/02-architecture.md)
- **Adding to a service?** → [Usage Examples](../examples/)

### For Service Teams
- **Deploy to ECR?** → [ECR Deployment Example](../examples/deploy-ecr-with-release.md)
- **ECS CodeDeploy?** → [ECS CodeDeploy Example](../examples/deploy-ecs-with-codedeploy.md)
- **Terraform?** → [Terraform Deployment](context/07-development.md#terraform-workflow)

### Related Documentation
- [Service Deployment Architecture](../../../Management/Architecture/Service-Deployment.md)
- [AWS Platform Architecture](../../../Management/Architecture/AWS-Architecture-v2.0-Final.md)
- [IaC Architecture](../../../Management/Architecture/IaC-Architecture.md)

---

## 📋 What is Documented Here?

This `.ai/` directory contains:
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
