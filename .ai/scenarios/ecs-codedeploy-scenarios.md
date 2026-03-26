# ECS CodeDeploy Scenarios — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Overview

ECS CodeDeploy Blue/Green deployment workflow scenarios.

---

## Scenario 1: Deploy to ECS via CodeDeploy

### Description

Update ECS Task Definition with new image and trigger CodeDeploy Blue/Green deployment.

### User Story

As a DevOps engineer, I want zero-downtime deployments, so that users are not affected by releases.

### Workflow

`reusable-ecs-codedeploy.yml`

### Inputs

```yaml
cluster_name: 'tec42-cluster'
service_name: 'tec42-identity-service'
task_family: 'tec42-identity-task'
container_name: 'identity'
container_port: 3010
ecr_repository: 'identity'
image_tag: 'v1.2.3'
codedeploy_application: 'tec42-identity-app'
codedeploy_deployment_group: 'tec42-identity-dg'
appspec_path: 'terraform/appspec.yaml'
```

### Secrets

```yaml
AWS_ROLE_ARN: 'arn:aws:iam::123456789:role/GitHubActionsRole'
```

### Usage Example

```yaml
jobs:
  deploy:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ecs-codedeploy.yml@v2.0.0
    with:
      cluster_name: tec42-cluster
      service_name: tec42-identity-service
      task_family: tec42-identity-task
      container_name: identity
      container_port: 3010
      ecr_repository: identity
      image_tag: v1.2.3
      codedeploy_application: tec42-identity-app
      codedeploy_deployment_group: tec42-identity-dg
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

### Flow

1. Checkout code (for appspec.yaml)
2. Configure AWS credentials (OIDC)
3. Check for active deployments (prevent duplicates)
4. Download current Task Definition
5. Build ECR image URI
6. Update Task Definition with new image
7. Register new Task Definition revision
8. Load AppSpec from service repo
9. Replace `<TASK_DEFINITION>` placeholder in AppSpec
10. Create CodeDeploy deployment
11. Wait for deployment (with timeout)

### Outputs

```yaml
deployment_id: 'd-ABC123XYZ'
task_definition_arn: 'arn:aws:ecs:eu-central-1:123456789:task-definition/tec42-identity-task:42'
```

### Test Scenarios

#### ✅ Happy Path

- No active deployment running
- Task Definition updated successfully
- CodeDeploy deployment starts
- Blue/Green switch completes

#### ❌ Error Cases

- **Active deployment**: `Deployment already in progress: d-ABC123XYZ`
- **Task Definition not found**: `task definition not found`
- **AppSpec missing**: `AppSpec not found at: terraform/appspec.yaml`
- **Health check fails**: Deployment rolls back automatically

### Key Files

- `.github/workflows/reusable-ecs-codedeploy.yml` — Workflow definition
- `terraform/appspec.yaml` (service repo) — Deployment spec

### AppSpec Template (Service Repo)

```yaml
# terraform/appspec.yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "identity"
          ContainerPort: 3010
        PlatformVersion: "LATEST"
```

### Business Rules

- Only one deployment at a time per service
- Automatic rollback on health check failure
- Service owns AppSpec (Service Ownership pattern)
- Wait timeout: 3 minutes

---

## Scenario 2: Deployment Blocked by Active Deployment

### Description

Attempt deployment while another deployment is in progress.

### User Story

As a DevOps engineer, I want deployments to fail fast if one is running, so that I don't cause conflicts.

### Flow

1. Check for active deployments
2. Active deployment found
3. Workflow fails with message

### Error Output

```
❌ Deployment already in progress: d-ABC123XYZ
Please wait for current deployment to complete or cancel it manually.
```

### Resolution

```bash
# Wait for deployment
aws deploy get-deployment --deployment-id d-ABC123XYZ

# Or cancel
aws deploy stop-deployment --deployment-id d-ABC123XYZ
```

---

## Scenario 3: Deployment with Health Check Timeout

### Description

Deployment starts but times out during health check phase.

### Trigger

New image fails to pass ALB health check.

### Flow

1. Deployment starts
2. New tasks launch (Blue)
3. Health check fails
4. CodeDeploy rolls back automatically
5. Workflow reports failure

### Recovery Steps

1. Check container logs: `aws logs get-log-events`
2. Check task status: `aws ecs describe-tasks`
3. Fix application issue
4. Re-trigger deployment
