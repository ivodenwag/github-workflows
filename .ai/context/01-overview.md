# Project Overview — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Purpose

Centralized, reusable GitHub Actions workflows for all tec42 microservices. This repository standardizes CI/CD processes including testing, releasing, and deployment.

---

## What This Repository Provides

| Capability | Workflow | Status |
|------------|----------|--------|
| CI Testing | `reusable-ci-docker.yml` | ✅ Production |
| Release + ECR Push | `reusable-release-ecr.yml` | ✅ Production |
| Terraform Deploy | `reusable-terraform-deploy.yml` | ✅ Production |
| ECS CodeDeploy | `reusable-ecs-codedeploy.yml` | ✅ Production |
| Full Pipeline | `reusable-service-deployment.yml` | ✅ Production |

---

## Technologies

| Technology | Purpose |
|------------|---------|
| GitHub Actions | CI/CD platform |
| AWS OIDC | Secure authentication (no static credentials) |
| release-it | Semantic versioning |
| Docker | Container builds |
| Terraform | Infrastructure deployment |
| CodeDeploy | Blue/Green ECS deployments |

---

## Structure

```
github-workflows/
├── .github/workflows/
│   ├── reusable-ci-docker.yml           # CI testing
│   ├── reusable-release-ecr.yml         # Release + ECR
│   ├── reusable-terraform-deploy.yml    # Terraform
│   ├── reusable-ecs-codedeploy.yml      # ECS deployment
│   └── reusable-service-deployment.yml  # Orchestration
├── shared/
│   └── .release-it.json                 # Release config
├── examples/
│   ├── deploy-ecr-with-release.md
│   └── deploy-ecs-with-codedeploy.md
├── .ai/                                 # Documentation
├── README.md
└── CHANGELOG.md
```

---

## Workflow Coverage

```
┌─────────────────────────────────────────────────────────┐
│  Atomic Workflows (Building Blocks)                     │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ CI Testing   │  │ Release+ECR  │  │ Terraform    │  │
│  │ lint         │  │ release-it   │  │ init         │  │
│  │ type-check   │  │ docker build │  │ plan         │  │
│  │ build        │  │ ECR push     │  │ apply        │  │
│  │ test         │  │              │  │              │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  ┌──────────────┐                                      │
│  │ ECS Deploy   │                                      │
│  │ task def     │                                      │
│  │ appspec      │                                      │
│  │ codedeploy   │                                      │
│  └──────────────┘                                      │
└─────────────────────────────────────────────────────────┘
                          │
                          ↓
┌─────────────────────────────────────────────────────────┐
│  Orchestration (Master Workflow)                        │
│                                                         │
│  reusable-service-deployment.yml                        │
│  ┌─────────┐ → ┌───────────┐ → ┌─────────────┐         │
│  │ Release │   │ Terraform │   │ ECS Deploy  │         │
│  │ + ECR   │   │ (optional)│   │ (CodeDeploy)│         │
│  └─────────┘   └───────────┘   └─────────────┘         │
└─────────────────────────────────────────────────────────┘
```

---

## Further Documentation

[Quick Start](00-quick-start.md) | [Architecture](02-architecture.md) | [Tech Stack](03-tech-stack.md)

**Purpose**: Blue/Green ECS deployments

**Features**:
- Task Definition update
- AppSpec generation
- CodeDeploy deployment trigger
- Health check monitoring
- Auto-rollback on failure

**Usage** (in service repository):
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-ecs-codedeploy.yml@main
with:
  cluster_name: tec42-cluster
  service_name: tec42-identity-service
  image_tag: ${{ needs.release-ecr.outputs.version }}
  codedeploy_application: tec42-identity-app
  codedeploy_deployment_group: tec42-identity-dg
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

**What it does**:
- Downloads current ECS Task Definition
- Creates new Task Definition revision with new image
- Generates appspec.yaml
- Triggers CodeDeploy Blue/Green deployment
- Monitors health checks

**Status**: In Development

---

### **5. Master Orchestration (reusable-service-deployment.yml)** 🚧

**Purpose**: Complete deployment pipeline

**Features**:
- Sequential workflow chaining
- Output passing between jobs
- Conditional execution (skip terraform)
- Single-file service configuration

**Usage** (in service repository):
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-service-deployment.yml@main
with:
  service_name: identity
  ecr_repository: identity
  cluster_name: tec42-cluster
  ecs_service_name: tec42-identity-service
  task_family: tec42-identity-task
  codedeploy_application: tec42-identity-app
  codedeploy_deployment_group: tec42-identity-dg
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

**What it does**:
Orchestrates complete deployment by chaining:
1. `reusable-release-ecr.yml`
2. `reusable-terraform-deploy.yml` (optional)
3. `reusable-ecs-codedeploy.yml`

**Status**: In Development

---

## 🎯 Design Principles

### **1. DRY (Don't Repeat Yourself)**
- Write workflow logic ONCE
- Reuse across ALL services
- Update in ONE place

### **2. Standardization**
- Consistent deployment process
- Same versioning strategy
- Common troubleshooting

### **3. Simplicity**
- Service repos are simple (10 lines)
- Complexity hidden in reusable workflows
- Easy to onboard new services

### **4. Security**
- OIDC authentication (no access keys)
- Least privilege IAM roles
- Secret management via GitHub Secrets

### **5. Observability**
- GitHub Action summaries
- CloudWatch logs integration
- Deployment status tracking

---

## 🔄 Workflow Execution Flow

**Within this repository:**

```
reusable-release-ecr.yml
├─ Job: test
│  └─ Runs CI tests
├─ Job: release
│  └─ Creates version & git tag
└─ Job: deploy
   └─ Builds & pushes Docker image

Outputs: version, has_release
```

```
reusable-terraform-deploy.yml
├─ Job: terraform
│  ├─ terraform init
│  ├─ terraform plan
│  └─ terraform apply

Outputs: terraform_outputs
```

```
reusable-ecs-codedeploy.yml
├─ Job: deploy
│  ├─ Update Task Definition
│  ├─ Generate appspec.yaml
│  └─ Trigger CodeDeploy

Outputs: deployment_id, task_definition_arn
```

**Orchestration** (reusable-service-deployment.yml):
```
Job 1: release-ecr
   ↓ (needs: release-ecr)
Job 2: deploy-infrastructure
   ↓ (needs: [release-ecr, deploy-infrastructure])
Job 3: deploy-ecs
```

**Note**: How services trigger these workflows is documented in `examples/`

---

## 📊 Benefits

### **For This Repository**
- ✅ Single source of truth for CI/CD logic
- ✅ Tested once, reused everywhere
- ✅ Easy to maintain & update
- ✅ Versioned via git tags

### **For Service Teams**
- ✅ No workflow logic in service repos
- ✅ Just configuration (inputs)
- ✅ Automatic updates when workflows improve
- ✅ Consistent behavior across all services

---

## 🔗 Key Concepts

### **workflow_call**
GitHub Actions trigger type for reusable workflows:
```yaml
on:
  workflow_call:
    inputs:
      my_input:
        required: true
        type: string
    secrets:
      MY_SECRET:
        required: true
```

### **Outputs**
Pass data between workflows:
```yaml
jobs:
  my_job:
    outputs:
      version: ${{ steps.release.outputs.version }}
```

### **OIDC Authentication**
Temporary AWS credentials via OpenID Connect:
- No access keys in GitHub
- AWS IAM Role trusts GitHub
- Automatic credential rotation

---

## 📚 Next Steps

1. [Quick Start](00-quick-start.md) - Workflow inputs/outputs
2. [Architecture](02-architecture.md) - Design patterns
3. [Examples](../../examples/) - Service integration
