# Overview - GitHub Workflows

**What**: Reusable CI/CD workflows for tec42 microservices  
**Why**: Standardize deployments across all services  
**How**: GitHub Actions with workflow_call pattern

---

## 🎯 Purpose

This repository provides **reusable GitHub Actions workflows** that can be called from any service repository to:
- ✅ Run CI tests (lint, type-check, build, test)
- ✅ Create semantic version releases
- ✅ Build & push Docker images to ECR
- ✅ Deploy infrastructure via Terraform
- ✅ Deploy services to ECS via CodeDeploy Blue/Green

---

## 🏗️ Workflow Architecture

```
┌─────────────────────────────────────────────────────────┐
│         github-workflows Repository                     │
│         (THIS REPO - Reusable workflows only)           │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Atomic Workflows (Individual tasks)             │  │
│  │                                                   │  │
│  │  reusable-ci-docker.yml                          │  │
│  │  reusable-release-ecr.yml                        │  │
│  │  reusable-terraform-deploy.yml                   │  │
│  │  reusable-ecs-codedeploy.yml                     │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Orchestration Workflow (Combines atomic)        │  │
│  │                                                   │  │
│  │  reusable-service-deployment.yml                 │  │
│  │  └─> Chains: release → terraform → codedeploy    │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Supporting Files                                │  │
│  │                                                   │  │
│  │  shared/.release-it.json                         │  │
│  │  shared/appspec-template.yaml                    │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Scope**: This documentation covers ONLY the workflows in this repository.  
**Service Integration**: See `examples/` directory for usage in service repos.

---

## 📦 Available Workflows

### **1. CI Testing (reusable-ci-docker.yml)**

**Purpose**: Run tests in Docker Compose environment

**Features**:
- Docker Compose based testing
- Parallel services (DB, Redis, etc.)
- Makefile-driven commands
- Dummy secrets for CI

**Usage** (in service repository):
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-ci-docker.yml@main
with:
  service_name: identity
  compose_files: 'docker-compose.yml docker-compose.ci.yml'
```

**What it does**:
- Runs `make lint`
- Runs `make type-check`
- Runs `make build`
- Runs `make test`

---

### **2. Release & ECR Push (reusable-release-ecr.yml)**

**Purpose**: Semantic versioning + Docker image deployment

**Features**:
- Conventional Commits analysis
- Automatic version bumping
- Git tag & GitHub release creation
- Docker build & push to ECR
- OIDC authentication (no access keys!)

**Usage** (in service repository):
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@main
with:
  service_name: identity
  ecr_repository: identity
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

**What it does**:
- Analyzes commits (conventional commits)
- Creates version (semantic versioning)
- Creates git tag & GitHub release
- Builds Docker image
- Pushes to ECR with version tag + latest

**Outputs**:
- `version` - New version tag (e.g., v1.2.3)
- `has_release` - Boolean if release was created

---

### **3. Terraform Deployment (reusable-terraform-deploy.yml)** 🚧

**Purpose**: Deploy static infrastructure

**Features**:
- Terraform in container
- S3 backend state management
- Plan & Apply
- Output extraction

**Usage** (in service repository):
```yaml
uses: ivodenwag/github-workflows/.github/workflows/reusable-terraform-deploy.yml@main
with:
  terraform_dir: terraform
  aws_region: eu-central-1
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

**What it does**:
- Runs `terraform init`
- Runs `terraform plan -out=tfplan` (saves plan to file)
- Uploads plan as GitHub artifact (audit trail)
- Runs `terraform apply tfplan` (executes saved plan)
- Exports terraform outputs

**Status**: In Development

---

### **4. ECS CodeDeploy (reusable-ecs-codedeploy.yml)** 🚧

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
