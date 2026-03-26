# Workflow: Add Composite Action — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Overview

How to create a reusable composite action for shared steps across workflows.

---

## When to Use

| Pattern | Use Case |
|---------|----------|
| **Reusable Workflow** | Complete job with multiple steps |
| **Composite Action** | Reusable step(s) within a job |

**Use Composite Actions for:**
- Common setup steps (AWS credentials, Docker login)
- Repeated patterns (build & push, notify)
- Step-level reuse across multiple workflows

---

## Architecture

```
.github/
├── workflows/
│   └── reusable-*.yml         # Full workflows
└── actions/
    └── {action-name}/
        └── action.yml         # Composite action
```

---

## Steps

### Step 1: Create Action Directory

```bash
mkdir -p .github/actions/{action-name}
```

### Step 2: Create action.yml

**File:** `.github/actions/{action-name}/action.yml`

```yaml
name: 'Action Name'
description: 'What this action does'
author: 'tec42'

inputs:
  required-input:
    description: 'What this input does'
    required: true
  optional-input:
    description: 'Optional parameter'
    required: false
    default: 'default-value'

outputs:
  result:
    description: 'What this action outputs'
    value: ${{ steps.main.outputs.result }}

runs:
  using: 'composite'
  steps:
    - name: Validate inputs
      shell: bash
      run: |
        if [ -z "${{ inputs.required-input }}" ]; then
          echo "::error::required-input is required"
          exit 1
        fi

    - name: Main operation
      id: main
      shell: bash
      run: |
        echo "Processing ${{ inputs.required-input }}..."
        echo "result=success" >> $GITHUB_OUTPUT
```

### Step 3: Use in Workflow

```yaml
# In a workflow file
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run custom action
        id: custom
        uses: ./.github/actions/{action-name}
        with:
          required-input: 'value'
      
      - name: Use output
        run: echo "Result: ${{ steps.custom.outputs.result }}"
```

---

## Example: AWS Setup Action

**File:** `.github/actions/aws-setup/action.yml`

```yaml
name: 'AWS Setup'
description: 'Configure AWS credentials and login to ECR'

inputs:
  role-arn:
    description: 'IAM Role ARN for OIDC'
    required: true
  aws-region:
    description: 'AWS Region'
    required: false
    default: 'eu-central-1'
  login-ecr:
    description: 'Login to ECR'
    required: false
    default: 'true'

outputs:
  ecr-registry:
    description: 'ECR Registry URL'
    value: ${{ steps.ecr-login.outputs.registry }}
  account-id:
    description: 'AWS Account ID'
    value: ${{ steps.get-account.outputs.account-id }}

runs:
  using: 'composite'
  steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ inputs.role-arn }}
        aws-region: ${{ inputs.aws-region }}
        role-session-name: GitHubActions

    - name: Get AWS Account ID
      id: get-account
      shell: bash
      run: |
        ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
        echo "account-id=$ACCOUNT_ID" >> $GITHUB_OUTPUT

    - name: Login to Amazon ECR
      id: ecr-login
      if: ${{ inputs.login-ecr == 'true' }}
      uses: aws-actions/amazon-ecr-login@v2
```

**Usage:**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup AWS
        id: aws
        uses: ./.github/actions/aws-setup
        with:
          role-arn: ${{ secrets.AWS_ROLE_ARN }}
          login-ecr: 'true'
      
      - name: Push to ECR
        run: |
          docker push ${{ steps.aws.outputs.ecr-registry }}/my-image:latest
```

---

## Example: Docker Build Action

**File:** `.github/actions/docker-build/action.yml`

```yaml
name: 'Docker Build'
description: 'Build Docker image with compose'

inputs:
  service-name:
    description: 'Docker compose service name'
    required: true
  compose-files:
    description: 'Docker compose files (space separated)'
    required: false
    default: 'docker-compose.yml'
  version:
    description: 'Image version tag'
    required: true

outputs:
  image-name:
    description: 'Built image name'
    value: ${{ steps.build.outputs.image }}

runs:
  using: 'composite'
  steps:
    - name: Build Docker image
      id: build
      shell: bash
      env:
        VERSION: ${{ inputs.version }}
      run: |
        COMPOSE_CMD="docker compose"
        for file in ${{ inputs.compose-files }}; do
          COMPOSE_CMD="$COMPOSE_CMD -f $file"
        done
        
        $COMPOSE_CMD build ${{ inputs.service-name }}
        echo "image=${{ inputs.service-name }}:$VERSION" >> $GITHUB_OUTPUT
```

---

## ✅ Checklist

- [ ] Action directory created: `.github/actions/{name}/`
- [ ] `action.yml` with `using: composite`
- [ ] All steps have `shell: bash` for run commands
- [ ] Inputs documented with descriptions
- [ ] Outputs defined and passed via `$GITHUB_OUTPUT`
- [ ] Error handling with `::error::` annotations
- [ ] Tested in workflow

---

## Composite vs Reusable Workflow

| Feature | Composite Action | Reusable Workflow |
|---------|------------------|-------------------|
| Scope | Steps within a job | Complete job(s) |
| Secrets | Passed explicitly | Can use `secrets: inherit` |
| Runners | Uses caller's runner | Own runner definition |
| Outputs | Step outputs | Job outputs |
| Location | `.github/actions/` | `.github/workflows/` |

**Rule of thumb:**
- **Composite**: Reuse within same job
- **Workflow**: Reuse complete job pipeline
