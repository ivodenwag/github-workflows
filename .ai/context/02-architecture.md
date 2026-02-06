# Architecture - GitHub Workflows

**Purpose**: Workflow design, orchestration & integration patterns  
**Audience**: Developers understanding the system

---

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    github-workflows Repository                   │
│                    (Central Workflow Library)                    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Atomic Workflows (Building Blocks)            │ │
│  │                                                            │ │
│  │  ┌──────────────────┐  ┌──────────────────────────┐     │ │
│  │  │ CI Testing       │  │ Terraform Deploy         │     │ │
│  │  │ - Lint           │  │ - Init                   │     │ │
│  │  │ - Type Check     │  │ - Plan                   │     │ │
│  │  │ - Build          │  │ - Apply                  │     │ │
│  │  │ - Test           │  │ - Output                 │     │ │
│  │  └──────────────────┘  └──────────────────────────┘     │ │
│  │                                                            │ │
│  │  ┌──────────────────┐  ┌──────────────────────────┐     │ │
│  │  │ Release & ECR    │  │ ECS CodeDeploy           │     │ │
│  │  │ - Release-it     │  │ - Task Definition        │     │ │
│  │  │ - Docker Build   │  │ - AppSpec                │     │ │
│  │  │ - ECR Push       │  │ - Deployment             │     │ │
│  │  │ - Tag Latest     │  │ - Health Check           │     │ │
│  │  └──────────────────┘  └──────────────────────────┘     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │           Orchestration Workflow (Master)                  │ │
│  │                                                            │ │
│  │          reusable-service-deployment.yml                   │ │
│  │          ┌───────────────────────────────┐                │ │
│  │          │ 1. Release & ECR              │                │ │
│  │          └──────────┬────────────────────┘                │ │
│  │                     ↓                                      │ │
│  │          ┌───────────────────────────────┐                │ │
│  │          │ 2. Terraform (optional)       │                │ │
│  │          └──────────┬────────────────────┘                │ │
│  │                     ↓                                      │ │
│  │          ┌───────────────────────────────┐                │ │
│  │          │ 3. ECS CodeDeploy             │                │ │
│  │          └───────────────────────────────┘                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ workflow_call
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     Service Repositories                         │
│                  (identity, render, payment)                     │
│                                                                  │
│  identity/.github/workflows/deploy.yml                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ jobs:                                                       │ │
│  │   deploy:                                                   │ │
│  │     uses: github-workflows/reusable-service-deployment.yml  │ │
│  │     with:                                                   │ │
│  │       service_name: identity                                │ │
│  │       cluster_name: tec42-cluster                           │ │
│  │       ecs_service_name: tec42-identity-service              │ │
│  │       codedeploy_application: tec42-identity-app            │ │
│  │       # ... (10 lines total)                                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Workflow Chaining with `needs`

### **Sequential Execution:**

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.release.outputs.version }}
    steps:
      - name: Create Release
        id: release
        run: echo "version=v1.2.3" >> $GITHUB_OUTPUT
  
  job2:
    needs: job1  # ⬅️ Waits for job1
    runs-on: ubuntu-latest
    steps:
      - name: Use Output
        run: echo "Version: ${{ needs.job1.outputs.version }}"
  
  job3:
    needs: [job1, job2]  # ⬅️ Waits for BOTH
    runs-on: ubuntu-latest
    steps:
      - name: Final Step
        run: echo "Both complete"
```

---

### **Workflow Chaining:**

```yaml
# Master Orchestration Workflow
jobs:
  release-ecr:
    uses: ./.github/workflows/reusable-release-ecr.yml
    # Outputs: version, has_release
  
  deploy-infra:
    needs: release-ecr  # ⬅️ Sequential
    uses: ./.github/workflows/reusable-terraform-deploy.yml
    # Waits for ECR push to complete
  
  deploy-ecs:
    needs: [release-ecr, deploy-infra]  # ⬅️ Waits for BOTH
    uses: ./.github/workflows/reusable-ecs-codedeploy.yml
    with:
      image_tag: ${{ needs.release-ecr.outputs.version }}
```

---

## 📊 Data Flow

### **Output Passing Between Workflows:**

```
┌─────────────────────────┐
│ reusable-release-ecr    │
│                         │
│ Outputs:                │
│ - version: "v1.2.3"     │
│ - has_release: true     │
└──────────┬──────────────┘
           │
           │ needs: release-ecr
           ↓
┌─────────────────────────┐
│ reusable-terraform      │
│                         │
│ Uses:                   │
│ - (no outputs needed)   │
│                         │
│ Outputs:                │
│ - terraform_outputs     │
└──────────┬──────────────┘
           │
           │ needs: [release-ecr, terraform]
           ↓
┌─────────────────────────┐
│ reusable-ecs-codedeploy │
│                         │
│ Uses:                   │
│ - version: v1.2.3       │
│ - terraform outputs     │
│                         │
│ Creates:                │
│ - Task Definition :42   │
│ - CodeDeploy deployment │
└─────────────────────────┘
```

---

## 🎯 Design Patterns

### **Pattern 1: Atomic Workflows**

Each workflow does ONE thing well:

```yaml
# reusable-ci-docker.yml
# Purpose: ONLY run tests
# Input: service_name, compose_files
# Output: none (pass/fail)

# reusable-release-ecr.yml
# Purpose: ONLY release & push image
# Input: service_name, ecr_repository
# Output: version, has_release

# reusable-terraform-deploy.yml
# Purpose: ONLY apply terraform
# Input: terraform_dir
# Output: terraform_outputs

# reusable-ecs-codedeploy.yml
# Purpose: ONLY deploy to ECS
# Input: cluster_name, image_tag, etc.
# Output: deployment_id
```

**Benefits**:
- Easy to test
- Easy to debug
- Easy to replace
- Easy to understand

---

### **Pattern 2: Orchestration Workflow**

Master workflow coordinates atomic workflows:

```yaml
# reusable-service-deployment.yml
# Purpose: Orchestrate complete deployment
# Delegates to: release-ecr, terraform, ecs-codedeploy
# Handles: Sequential execution, output passing, error handling

jobs:
  release-ecr:
    uses: ./.github/workflows/reusable-release-ecr.yml
    
  deploy-infra:
    needs: release-ecr
    if: ${{ !inputs.skip_terraform }}  # Conditional
    uses: ./.github/workflows/reusable-terraform-deploy.yml
    
  deploy-ecs:
    needs: [release-ecr, deploy-infra]
    if: always() && needs.release-ecr.outputs.has_release == 'true'
    uses: ./.github/workflows/reusable-ecs-codedeploy.yml
```

**Benefits**:
- Single entry point for services
- Consistent orchestration logic
- Easy to add/remove steps

---

### **Pattern 3: Service Configuration**

Service repos provide business logic, not workflow logic:

```yaml
# identity/.github/workflows/deploy.yml
# Purpose: Configure identity-specific values
# Does NOT contain: workflow logic, steps, scripts

jobs:
  deploy:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-service-deployment.yml@main
    with:
      service_name: identity           # Business config
      cluster_name: tec42-cluster      # Business config
      ecs_service_name: tec42-identity # Business config
      # ... only configuration, no logic
```

**Benefits**:
- Service repos are trivial
- Changes to workflow don't affect services
- Easy to onboard new services

---

## 🔄 Deployment Lifecycle

### **Phase 1: Pre-Deployment**

```
Developer works on feature
        ↓
Create PR
        ↓
CI workflow runs (optional)
├─ Lint
├─ Type Check
├─ Build
└─ Test
        ↓
PR Review & Merge to main
```

---

### **Phase 2: Release & Build**

```
Push to main
        ↓
Master Workflow triggers
        ↓
Job 1: release-ecr
├─ Analyze commits (conventional commits)
├─ Determine version bump (major/minor/patch)
├─ Create git tag (v1.2.3)
├─ Create GitHub release
├─ Build Docker image
├─ Tag image: identity:v1.2.3
├─ Tag image: identity:latest
└─ Push to ECR
        ↓
Output: version=v1.2.3, has_release=true
```

---

### **Phase 3: Infrastructure**

```
Job 2: deploy-infra (needs: release-ecr)
├─ Checkout terraform/
├─ Terraform init (S3 backend)
├─ Terraform plan -out=tfplan
├─ Upload plan artifact (audit trail)
├─ Terraform apply tfplan (uses saved plan)
│  ├─ RDS (if changed)
│  ├─ Redis (if changed)
│  ├─ ECS Service (if changed)
│  ├─ CodeDeploy Config (if changed)
│  └─ Target Groups (if changed)
└─ Output terraform state
```

---

### **Phase 4: Application Deployment**

```
Job 3: deploy-ecs (needs: [release-ecr, deploy-infra])
├─ Download current Task Definition
├─ Create new Task Definition revision
│  └─ Update image: identity:v1.2.3
├─ Register Task Definition → :42 (new revision)
├─ Generate appspec.yaml
│  ├─ TaskDefinition: arn:...:42
│  ├─ ContainerName: identity
│  └─ ContainerPort: 3010
├─ Create CodeDeploy Deployment
│  ├─ Application: tec42-identity-app
│  ├─ DeploymentGroup: tec42-identity-dg
│  └─ AppSpec: appspec.yaml
└─ Monitor deployment
   ├─ Green tasks starting
   ├─ Health checks
   ├─ Traffic shift (10%/min)
   └─ Blue tasks terminating
```

---

## 🔒 Security Architecture

### **OIDC Authentication:**

```
┌─────────────────────┐
│ GitHub Actions      │
│ identity/deploy.yml │
└──────────┬──────────┘
           │
           │ 1. Request temporary credentials
           ↓
┌─────────────────────┐
│ GitHub OIDC         │
│ Provider            │
└──────────┬──────────┘
           │
           │ 2. Verify identity (JWT token)
           ↓
┌─────────────────────┐
│ AWS IAM             │
│ OIDC Provider       │
└──────────┬──────────┘
           │
           │ 3. Assume role (if trusted)
           ↓
┌─────────────────────┐
│ IAM Role            │
│ github-actions-ecr  │
│ Permissions:        │
│ - ECR Push          │
│ - ECS Update        │
│ - CodeDeploy Deploy │
└──────────┬──────────┘
           │
           │ 4. Temporary credentials (1 hour)
           ↓
     GitHub Actions
     (executes workflow)
```

**Benefits**:
- ✅ No long-lived credentials
- ✅ Automatic rotation
- ✅ Auditable (CloudTrail)
- ✅ Least privilege

---

### **Secret Management:**

```
Service Repository Secrets:
├─ AWS_ROLE_ARN (public, not sensitive)
└─ (NO AWS keys!)

GitHub Workflows Secrets:
├─ NPM_TOKEN (optional, for private packages)
└─ (NO AWS keys!)

AWS Secrets Manager:
├─ Database credentials
├─ API keys
├─ JWT secrets
└─ Referenced in Task Definition
```

---

## 📐 Module Architecture

### **Layered Design:**

```
Layer 1: Foundation (tf-aws-infra)
├─ VPC, Subnets, NAT Gateway
├─ ALB, CloudFront
├─ API Gateway (Private)
├─ ECS Cluster
├─ Route53 (Public + Private)
└─ CodeDeploy IAM Role

Layer 2: Service Infrastructure (identity/terraform)
├─ RDS PostgreSQL
├─ ElastiCache Redis
├─ ECS Service (CODE_DEPLOY mode)
├─ CodeDeploy Application
├─ CodeDeploy Deployment Group
├─ Target Groups (Blue + Green)
└─ IAM Roles (Task + Execution)

Layer 3: Application (identity/app)
├─ Docker Image
└─ Deployed via CodeDeploy
```

**Workflow Responsibility**:
- `terraform-deploy.yml` → Layer 2
- `ecs-codedeploy.yml` → Layer 3

---

## 🎛️ Configuration Strategy

### **Three Levels of Configuration:**

```
1. Workflow Defaults (in reusable workflow)
├─ aws_region: eu-central-1
├─ terraform_version: 1.7.0
└─ compose_files: docker-compose.yml docker-compose.ci.yml

2. Service Configuration (in service repo)
├─ service_name: identity
├─ cluster_name: tec42-cluster
└─ codedeploy_application: tec42-identity-app

3. Runtime Variables (from job outputs)
├─ image_tag: ${{ needs.release-ecr.outputs.version }}
└─ has_release: ${{ needs.release-ecr.outputs.has_release }}
```

---

## 🔍 Error Handling

### **Fail-Fast Strategy:**

```yaml
jobs:
  release-ecr:
    # If tests fail → STOP
    # If release fails → STOP
    # If build fails → STOP
  
  deploy-infra:
    needs: release-ecr
    # Only runs if release-ecr succeeds
    # If terraform fails → STOP
  
  deploy-ecs:
    needs: [release-ecr, deploy-infra]
    if: always() && needs.release-ecr.outputs.has_release == 'true'
    # Runs even if terraform fails (if needed)
    # CodeDeploy has built-in rollback
```

### **Rollback Mechanisms:**

1. **CodeDeploy Auto-Rollback**
   - Health check failure → rollback
   - CloudWatch alarm → rollback
   - Manual stop → rollback

2. **Terraform State**
   - `terraform plan` before apply
   - S3 state versioning
   - Manual rollback: `terraform apply` with old state

3. **Git Rollback**
   - Revert commit
   - Push to main
   - New deployment with old code

---

## 📊 Monitoring & Observability

### **GitHub Actions:**
- ✅ Workflow run history
- ✅ Job summaries
- ✅ Step-by-step logs
- ✅ Artifact uploads

### **AWS CloudWatch:**
- ✅ ECS task logs
- ✅ CodeDeploy deployment logs
- ✅ Application logs
- ✅ Metrics & alarms

### **AWS X-Ray:** (Future)
- ⏳ Request tracing
- ⏳ Performance analysis
- ⏳ Error tracking

---

## 📚 Related Documentation

- [Service Deployment Architecture](../../../../Management/Architecture/Service-Deployment.md)
- [AWS Architecture](../../../../Management/Architecture/AWS-Architecture-v2.0-Final.md)
- [IaC Architecture](../../../../Management/Architecture/IaC-Architecture.md)
- [TASKS.md](../../TASKS.md) - Implementation plan

---

## 🎯 Next Steps

1. Understand [Tech Stack](03-tech-stack.md)
2. Learn [Development](07-development.md) process
3. Review [Examples](../../examples/)
