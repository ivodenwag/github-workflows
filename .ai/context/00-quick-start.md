# Quick Start — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Prerequisites

- GitHub repository with Actions enabled
- AWS account with OIDC provider configured
- IAM Role with trust policy for GitHub Actions

---

## Quick Start (Full Deployment)

Add to your service repository:

```yaml
# .github/workflows/deploy.yml
name: Deploy Service

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-service-deployment.yml@v2.0.0
    with:
      service_name: your-service
      ecr_repository: your-service
      cluster_name: tec42-cluster
      ecs_service_name: tec42-your-service
      task_family: tec42-your-task
      container_name: your-container
      codedeploy_application: tec42-your-app
      codedeploy_deployment_group: tec42-your-dg
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

---

## Available Workflows

| Workflow | Purpose | Key Inputs |
|----------|---------|------------|
| `reusable-ci-docker.yml` | Run tests | `service_name` |
| `reusable-release-ecr.yml` | Release + ECR push | `service_name`, `ecr_repository` |
| `reusable-terraform-deploy.yml` | Terraform apply | `terraform_dir` |
| `reusable-ecs-codedeploy.yml` | ECS Blue/Green | `cluster_name`, `image_tag` |
| `reusable-service-deployment.yml` | Full pipeline | 13 inputs |

---

## CI Only (Testing)

```yaml
jobs:
  test:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci-docker.yml@v2.0.0
    with:
      service_name: your-service
```

---

## Release + ECR Only

```yaml
jobs:
  release:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@v2.0.0
    with:
      service_name: your-service
      ecr_repository: your-service
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

---

## Versioning

| Environment | Reference | Example |
|-------------|-----------|---------|
| Production | Specific version | `@v2.0.0` |
| Staging | Major version | `@v2` |
| Development | Branch | `@main` |

---

## Further Documentation

[Overview](01-overview.md) | [Architecture](02-architecture.md) | [Tech Stack](03-tech-stack.md)
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
