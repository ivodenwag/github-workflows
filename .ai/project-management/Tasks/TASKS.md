# GitHub Workflows - ECS CodeDeploy Integration Tasks

**Date**: 6. Februar 2026  
**Goal**: Implement Blue/Green ECS Deployments with CodeDeploy  
**Status**: Planning Phase

---

## 🎯 Objective

Create reusable GitHub Actions workflows for:
1. ✅ Docker Image Build & ECR Push (existing)
2. 🆕 Terraform Infrastructure Deployment
3. 🆕 ECS Service Deployment via CodeDeploy Blue/Green
4. 🆕 Master Orchestration Workflow (coordinates all)

---

## 📋 Current State

### **Existing Workflows:**
- ✅ `reusable-ci-docker.yml` - CI Testing with Docker Compose
- ✅ `reusable-release-ecr.yml` - Release-it + ECR Push

### **Service Repositories:**
- ✅ `identity/.github/workflows/ci.yml` - Uses reusable-ci-docker.yml
- ✅ `identity/.github/workflows/deploy.yml` - Uses reusable-release-ecr.yml
- ✅ `render/.github/workflows/ci.yml` - Uses reusable-ci-docker.yml
- ✅ `render/.github/workflows/deploy.yml` - Uses reusable-release-ecr.yml

---

## 🏗️ Architecture Design

### **Problem:**
Current workflow only pushes Docker images to ECR, but doesn't deploy to ECS with Blue/Green strategy.

### **Solution:**
Create Master Orchestration Workflow that chains:
1. Release & ECR Push (existing)
2. Terraform Deploy (new)
3. CodeDeploy Deployment (new)

---

## 📁 Target Structure

```
github-workflows/
├── .github/workflows/
│   ├── reusable-ci-docker.yml              ✅ Existing
│   ├── reusable-release-ecr.yml            ✅ Existing
│   ├── reusable-terraform-deploy.yml       🆕 NEW
│   ├── reusable-ecs-codedeploy.yml         🆕 NEW
│   └── reusable-service-deployment.yml     🆕 NEW (Master Orchestration)
├── shared/
│   └── .release-it.json                    ✅ Existing
├── examples/
│   ├── deploy-ecr-with-release.md          ✅ Existing
│   └── deploy-ecs-with-codedeploy.md       🆕 NEW
├── README.md                                🔄 UPDATE
└── CHANGELOG.md                             🔄 UPDATE

Service Repositories (Service Ownership):
identity/
├── .github/workflows/
│   └── deploy.yml                          🔄 UPDATE (use master orchestration)
└── terraform/
    └── appspec.yaml                        🆕 NEW (service-owned)

render/
├── .github/workflows/
│   └── deploy.yml                          🔄 UPDATE (use master orchestration)
└── terraform/
    └── appspec.yaml                        🆕 NEW (service-owned)
```

---

## 🔨 Implementation Tasks

### **Phase 1: Terraform Deployment Workflow** ✅

**File:** `reusable-terraform-deploy.yml` ✅ **COMPLETED**

**Purpose:** Deploy static infrastructure (RDS, Redis, ECS Service Config, CodeDeploy Config)

**Inputs:**
```yaml
inputs:
  terraform_dir:
    description: 'Path to terraform directory'
    required: false
    type: string
    default: 'terraform'
  
  terraform_version:
    description: 'Terraform version'
    required: false
    type: string
    default: '1.7.0'
  
  aws_region:
    description: 'AWS region'
    required: false
    type: string
    default: 'eu-central-1'

secrets:
  AWS_ROLE_ARN:
    description: 'AWS IAM Role ARN for OIDC'
    required: true
```

**Steps:**
- [x] Checkout code ✅
- [x] Configure AWS credentials (OIDC) ✅
- [x] Setup Terraform ✅
- [x] Terraform Init ✅
- [x] Terraform Validate ✅
- [x] Terraform Plan (`-out=tfplan`) ✅
- [x] Upload Plan as Artifact (for audit trail) ✅
- [x] Terraform Apply (`tfplan` - uses saved plan) ✅
- [x] Handle errors (fail job on any step failure) ✅
- [x] Output summary ✅

**Error Handling:**
- Fail fast on validation/plan errors ✅
- Upload plan artifact even on failure (debugging) ✅
- Set job result appropriately for dependent jobs ✅

**Outputs:**
```yaml
outputs:
  terraform_outputs:
    description: 'JSON string of terraform outputs'
    value: ${{ jobs.terraform.outputs.tf_outputs }}
```

---

### **Phase 2: ECS CodeDeploy Workflow** ✅

**File:** `reusable-ecs-codedeploy.yml` ✅ **COMPLETED**

**Purpose:** Trigger Blue/Green deployment via CodeDeploy

**Inputs:**
```yaml
inputs:
  cluster_name:
    description: 'ECS cluster name'
    required: true
    type: string
  
  service_name:
    description: 'ECS service name'
    required: true
    type: string
  
  task_family:
    description: 'ECS task definition family'
    required: true
    type: string
  
  container_name:
    description: 'Container name in task definition'
    required: true
    type: string
  
  container_port:
    description: 'Container port'
    required: false
    type: number
    default: 3000
  
  ecr_repository:
    description: 'ECR repository name'
    required: true
    type: string
  
  image_tag:
    description: 'Docker image tag'
    required: true
    type: string
  
  codedeploy_application:
    description: 'CodeDeploy application name'
    required: true
    type: string
  
  codedeploy_deployment_group:
    description: 'CodeDeploy deployment group name'
    required: true
    type: string
  
  appspec_path:
    description: 'Path to appspec.yaml in service repository'
    required: false
    type: string
    default: 'terraform/appspec.yaml'
  
  aws_region:
    description: 'AWS region'
    required: false
    type: string
    default: 'eu-central-1'

secrets:
  AWS_ROLE_ARN:
    description: 'AWS IAM Role ARN for OIDC'
    required: true
```

**Steps:**
- [x] Checkout code ✅
- [x] Configure AWS credentials (OIDC) ✅
- [x] Check for active deployments (prevent duplicates) ✅
- [x] Download current Task Definition from ECS ✅
- [x] Create new Task Definition revision with new image tag ✅
- [x] Register new Task Definition with ECS ✅
- [x] Load appspec.yaml from service repository (default: `terraform/appspec.yaml`) ✅
- [x] Replace `<TASK_DEFINITION>` placeholder with new Task Definition ARN ✅
- [x] Create CodeDeploy Deployment ✅
- [x] Wait for initial health check (2-3 minutes, timeout on failure) ✅
- [x] Output deployment summary ✅

**Wait Strategy (Hybrid):**
- Wait 2-3 minutes for initial health check ✅
- Fast failure detection (bad container, port issues) ✅
- Full deployment continues in background ✅
- Use `timeout` command to prevent hanging ✅
- SNS notifications can be configured in CodeDeploy for final status ✅

**Active Deployment Check:** ✅
```bash
aws deploy list-deployments \
  --deployment-group-name $DG_NAME \
  --include-only-statuses InProgress
```
- Fail if deployment already running
- Prevent duplicate/concurrent deployments

**Outputs:**
```yaml
outputs:
  deployment_id:
    description: 'CodeDeploy deployment ID'
    value: ${{ jobs.deploy.outputs.deployment_id }}
  
  task_definition_arn:
    description: 'New task definition ARN'
    value: ${{ jobs.deploy.outputs.task_def_arn }}
```

---

### **Phase 3: Master Orchestration Workflow** ✅

**File:** `reusable-service-deployment.yml` ✅ **COMPLETED**

**Purpose:** Coordinate complete deployment pipeline (Release → Terraform → CodeDeploy)

**Inputs:**
```yaml
inputs:
  # Service Identity
  service_name:
    description: 'Service name (e.g., identity, render)'
    required: true
    type: string
  
  ecr_repository:
    description: 'ECR repository name'
    required: true
    type: string
  
  # ECS Configuration
  cluster_name:
    description: 'ECS cluster name'
    required: true
    type: string
  
  ecs_service_name:
    description: 'ECS service name'
    required: true
    type: string
  
  task_family:
    description: 'ECS task definition family'
    required: true
    type: string
  
  container_name:
    description: 'Container name in task definition'
    required: false
    type: string
    default: 'app'
  
  container_port:
    description: 'Container port'
    required: false
    type: number
    default: 3000
  
  # CodeDeploy Configuration
  codedeploy_application:
    description: 'CodeDeploy application name'
    required: true
    type: string
  
  codedeploy_deployment_group:
    description: 'CodeDeploy deployment group name'
    required: true
    type: string
  
  # Optional
  terraform_dir:
    description: 'Path to terraform directory'
    required: false
    type: string
    default: 'terraform'
  
  skip_terraform:
    description: 'Skip terraform deployment'
    required: false
    type: boolean
    default: false
  
  compose_files:
    description: 'Docker compose files (space separated)'
    required: false
    type: string
    default: 'docker-compose.yml docker-compose.ci.yml'
  
  aws_region:
    description: 'AWS region'
    required: false
    type: string
    default: 'eu-central-1'

secrets:
  AWS_ROLE_ARN:
    description: 'AWS IAM Role ARN for OIDC'
    required: true
  
  NPM_TOKEN:
    description: 'NPM token for release-it'
    required: false
```

**Jobs:**
```yaml
jobs:
  # Job 1: Release & ECR Push ✅
  release-ecr:
    uses: ./.github/workflows/reusable-release-ecr.yml
    with:
      service_name: ${{ inputs.service_name }}
      ecr_repository: ${{ inputs.ecr_repository }}
      compose_files: ${{ inputs.compose_files }}
      aws_region: ${{ inputs.aws_region }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
  
  # Job 2: Terraform Infrastructure (optional) ✅
  deploy-infrastructure:
    needs: release-ecr
    if: ${{ !inputs.skip_terraform }}
    uses: ./.github/workflows/reusable-terraform-deploy.yml
    with:
      terraform_dir: ${{ inputs.terraform_dir }}
      aws_region: ${{ inputs.aws_region }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  
  # Job 3: ECS CodeDeploy ✅
  deploy-ecs:
    needs: [release-ecr, deploy-infrastructure]
    if: |
      always() && 
      needs.release-ecr.result == 'success' &&
      needs.release-ecr.outputs.has_release == 'true' &&
      (needs.deploy-infrastructure.result == 'success' || 
       needs.deploy-infrastructure.result == 'skipped')
    uses: ./.github/workflows/reusable-ecs-codedeploy.yml
    with:
      cluster_name: ${{ inputs.cluster_name }}
      service_name: ${{ inputs.ecs_service_name }}
      task_family: ${{ inputs.task_family }}
      container_name: ${{ inputs.container_name }}
      container_port: ${{ inputs.container_port }}
      ecr_repository: ${{ inputs.ecr_repository }}
      image_tag: ${{ needs.release-ecr.outputs.version }}
      codedeploy_application: ${{ inputs.codedeploy_application }}
      codedeploy_deployment_group: ${{ inputs.codedeploy_deployment_group }}
      aws_region: ${{ inputs.aws_region }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

---

### **Phase 4: Documentation & Examples**

---

### **Phase 4: Documentation & Examples**

**Update:** `README.md`

Add sections:
- Workflow: `reusable-service-deployment.yml`
- Workflow: `reusable-terraform-deploy.yml`
- Workflow: `reusable-ecs-codedeploy.yml`
- **Versioning Strategy**: Services should pin to stable versions
  - Development: `@main`
  - Production: `@v2.0.0` (semantic versioning)
  - Tag releases with git tags
- **Error Handling**: Conditional job execution, fail-fast strategy
- **Notifications**: GitHub Email enabled by default (Settings → Notifications → Actions)

**Update:** `CHANGELOG.md`

Add entry:
```markdown
## [2.0.0] - 2026-02-06

### Added
- `reusable-service-deployment.yml` - Master orchestration workflow
- `reusable-terraform-deploy.yml` - Terraform infrastructure deployment
- `reusable-ecs-codedeploy.yml` - ECS Blue/Green deployment via CodeDeploy
- Complete ECS CodeDeploy integration with Blue/Green strategy

### Changed
- Service workflows now use master orchestration workflow
- Simplified service repository workflow files

### Documentation
- Services must provide `terraform/appspec.yaml` (Service Ownership)
- See `examples/deploy-ecs-with-codedeploy.md` for integration guide
```

**New Example:** `examples/deploy-ecs-with-codedeploy.md`

Content should include:
- How to use the new workflows in service repositories
- Prerequisites (ECS Service with CODE_DEPLOY mode, CodeDeploy setup)
- Example workflow files (identity, render)
- Example appspec.yaml structure
- Troubleshooting guide

**Service Repository Requirements:**

Each service repository (identity, render) must provide:

```
service-repo/
├── .github/workflows/
│   └── deploy.yml              # Uses reusable-service-deployment.yml
├── terraform/
│   ├── appspec.yaml           # ← Service-owned deployment config
│   ├── main.tf
│   └── ...
└── docker-compose.yml
```

**Example appspec.yaml** (Service Ownership):

```yaml
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

**Note:** The workflow will read `terraform/appspec.yaml` from the service repository. Services have full control over their deployment configuration.

---

## 🔄 Deployment Flow

### **Timeline:**

```
0:00  Push to main branch
      ↓
0:01  Job 1: release-ecr starts
      ├─ Run tests
      ├─ Create release (release-it)
      ├─ Build Docker image
      └─ Push to ECR
      
0:05  Job 2: deploy-infrastructure starts (needs: release-ecr)
      ├─ Terraform init
      ├─ Terraform plan
      └─ Terraform apply (RDS, Redis, ECS Service, CodeDeploy)
      
0:15  Job 3: deploy-ecs starts (needs: [release-ecr, deploy-infrastructure])
      ├─ Download current Task Definition
      ├─ Create new Task Definition revision with new image
      ├─ Generate appspec.yaml
      ├─ Create CodeDeploy deployment
      └─ Wait for Blue/Green deployment to complete
      
0:30  Deployment complete
      ├─ Traffic shifted to Green
      ├─ Blue tasks terminated
      └─ Success! ✅
```

---

## ✅ Success Criteria

- [ ] All 3 new workflows created and tested
- [ ] Identity service successfully deploys via CodeDeploy
- [ ] Render service successfully deploys via CodeDeploy
- [ ] Services provide their own `terraform/appspec.yaml` (Service Ownership)
- [ ] Zero downtime during deployments
- [ ] Automatic rollback on failure
- [ ] Documentation complete with service integration examples
- [ ] Examples working

---

## 🚧 Open Questions

1. **Terraform State Management:**
   - ✅ S3 backend with state file per service
   - ✅ DynamoDB Lock Table required (prevents concurrent modifications)
   - **Service Responsibility**: Each service configures backend in `terraform/backend.tf`
   - **Workflow Responsibility**: Workflows only execute terraform commands
   - S3 bucket + DynamoDB table managed by central infrastructure (tf-aws-infra)

2. **Secrets Management:**
   - How to pass database credentials to Task Definition?
   - Use AWS Secrets Manager references in Task Definition
   - Services define secrets in their own `terraform/` code

3. **AppSpec Ownership:**
   - ✅ Each service provides `terraform/appspec.yaml`
   - Workflow loads it and replaces `<TASK_DEFINITION>` placeholder
   - Services have full control over deployment configuration

3. **Health Checks:**
   - What endpoint should CodeDeploy use for health checks?
   - Recommendation: `/api/health` for all services

4. **Rollback Strategy:**
   - Automatic rollback on health check failure? YES
   - Manual rollback option? YES (via AWS CLI)

5. **Monitoring:**
   - CloudWatch alarms for auto-rollback?
   - Later phase after initial implementation

---

## 📝 Notes

### **Deployment Architecture:**
- **CodeDeploy requires:** ECS Service with `deployment_controller.type = CODE_DEPLOY`
- **Blue/Green requires:** 2 Target Groups (managed by Terraform)
- **AppSpec Ownership:** Each service provides `terraform/appspec.yaml` (Service Ownership principle)
- **Workflow responsibility:** Load appspec.yaml, replace `<TASK_DEFINITION>` placeholder, trigger deployment
- **Image tags:** Use version from release-it (semantic versioning)
- **Task Definition:** Always update with new image tag, ECS auto-increments revision

### **Best Practices (Implemented):**
- ✅ **Error Handling:** Conditional job execution with `if: always()` and result checks
- ✅ **Idempotency:** Active deployment check prevents concurrent deployments
- ✅ **Hybrid Wait:** 2-3 min initial health check, background completion
- ✅ **Workflow Versioning:** Services pin to @v2.0.0, not @main
- ✅ **Terraform Locking:** DynamoDB table (service responsibility)
- ✅ **Notifications:** GitHub Email (zero config, already enabled)
- ✅ **Fail Fast:** Terraform validate/plan errors stop pipeline immediately

---

## 🔗 Related Documentation

- [Service Deployment Architecture](../../../Management/Architecture/Service-Deployment.md)
- [AWS Architecture v2.0](../../../Management/Architecture/AWS-Architecture-v2.0-Final.md)
- [IaC Architecture](../../../Management/Architecture/IaC-Architecture.md)

---

**Next Steps:**
1. Implement Phase 1: Terraform workflow
2. Implement Phase 2: CodeDeploy workflow  
3. Implement Phase 3: Master orchestration workflow
4. Implement Phase 4: Documentation & examples
5. Service teams create `terraform/appspec.yaml` in their repos
6. Service teams update `.github/workflows/deploy.yml` to use master orchestration
