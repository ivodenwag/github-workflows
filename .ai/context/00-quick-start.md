# Quick Start - GitHub Workflows

**Goal**: Understand reusable workflows in 5 minutes  
**Audience**: Developers using these workflows  
**Scope**: github-workflows repository only

---

## 🎯 What This Repository Provides

This repository contains **reusable GitHub Actions workflows** that can be called from service repositories.

### **Available Workflows:**

1. **CI Testing** - `reusable-ci-docker.yml`
2. **Release & ECR** - `reusable-release-ecr.yml`
3. **Terraform Deploy** - `reusable-terraform-deploy.yml` (in development)
4. **ECS CodeDeploy** - `reusable-ecs-codedeploy.yml` (in development)
5. **Master Orchestration** - `reusable-service-deployment.yml` (in development)

---

## 📋 Workflow Inputs & Outputs

### **1. reusable-ci-docker.yml**

**Purpose**: Run tests in Docker Compose environment

**Inputs:**
```yaml
compose_files:     # Docker compose files (space separated)
  type: string
  default: 'docker-compose.yml docker-compose.ci.yml'

service_name:      # Docker compose service name
  type: string
  required: true
```

**Secrets:** None required

**Outputs:** None (pass/fail)

---

### **2. reusable-release-ecr.yml**

**Purpose**: Create semantic version release + push Docker image to ECR

**Inputs:**
```yaml
compose_files:     # Docker compose files
  type: string
  default: 'docker-compose.yml docker-compose.ci.yml'

service_name:      # Docker compose service name
  type: string
  required: true

ecr_repository:    # ECR repository name
  type: string
  required: true

aws_region:        # AWS region
  type: string
  default: 'eu-central-1'
```

**Secrets:**
```yaml
AWS_ROLE_ARN:      # IAM Role ARN for OIDC
  required: true

NPM_TOKEN:         # NPM token (optional)
  required: false
```

**Outputs:**
```yaml
version:           # Released version (e.g., v1.2.3)
has_release:       # Boolean if release was created
```

---

### **3. reusable-terraform-deploy.yml** (In Development)

**Purpose**: Deploy infrastructure via Terraform

**Inputs:**
```yaml
terraform_dir:     # Path to terraform directory
  type: string
  default: 'terraform'

terraform_version: # Terraform version
  type: string
  default: '1.7.0'

aws_region:        # AWS region
  type: string
  default: 'eu-central-1'
```

**Secrets:**
```yaml
AWS_ROLE_ARN:      # IAM Role ARN for OIDC
  required: true
```

**Outputs:**
```yaml
terraform_outputs: # JSON string of terraform outputs
```

---

### **4. reusable-ecs-codedeploy.yml** (In Development)

**Purpose**: Deploy ECS service via CodeDeploy Blue/Green

**Inputs:**
```yaml
cluster_name:              # ECS cluster name
  type: string
  required: true

service_name:              # ECS service name
  type: string
  required: true

task_family:               # Task definition family
  type: string
  required: true

container_name:            # Container name
  type: string
  required: true

container_port:            # Container port
  type: number
  default: 3000

ecr_repository:            # ECR repository
  type: string
  required: true

image_tag:                 # Docker image tag
  type: string
  required: true

codedeploy_application:    # CodeDeploy app name
  type: string
  required: true

codedeploy_deployment_group: # Deployment group
  type: string
  required: true
```

**Secrets:**
```yaml
AWS_ROLE_ARN:      # IAM Role ARN for OIDC
  required: true
```

**Outputs:**
```yaml
deployment_id:     # CodeDeploy deployment ID
task_definition_arn: # New task definition ARN
```

---

## 🔗 How to Use

These workflows are called from service repositories using `workflow_call`.

**Example:**
```yaml
jobs:
  deploy:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@main
    with:
      service_name: myservice
      ecr_repository: myservice
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

**Note**: Service-specific implementation examples are in `../../examples/`

---

## 📚 Learn More

- [Overview](01-overview.md) - Workflow purposes & features
- [Architecture](02-architecture.md) - Workflow design & chaining
- [Examples](../../examples/) - Service integration examples
