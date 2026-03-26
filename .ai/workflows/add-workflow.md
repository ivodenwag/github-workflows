# Workflow: Add New Reusable Workflow — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Overview

How to create a new reusable GitHub Actions workflow in this repository.

---

## Architecture

```
.github/workflows/
    ↓
reusable-{scope}-{action}.yml    # New workflow
    ↓
shared/                           # Shared config (optional)
    ↓
examples/                         # Usage documentation
```

---

## Steps

### Step 1: Create Workflow File

**File:** `.github/workflows/reusable-{name}.yml`

```yaml
name: Descriptive Workflow Name

on:
  workflow_call:
    inputs:
      required_input:
        description: 'What this input does'
        required: true
        type: string
      optional_input:
        description: 'Optional parameter'
        required: false
        type: string
        default: 'default-value'
      boolean_input:
        description: 'Toggle feature'
        required: false
        type: boolean
        default: false
      number_input:
        description: 'Numeric value'
        required: false
        type: number
        default: 3000
    secrets:
      AWS_ROLE_ARN:
        description: 'IAM Role for OIDC authentication'
        required: true
      OPTIONAL_SECRET:
        description: 'Optional secret'
        required: false
    outputs:
      result:
        description: 'What this workflow outputs'
        value: ${{ jobs.main.outputs.result }}

permissions:
  id-token: write    # For OIDC (if using AWS)
  contents: read     # For checkout

jobs:
  main:
    name: Job Name
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.main-step.outputs.value }}
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
      
      - name: 🔐 Configure AWS credentials
        if: ${{ secrets.AWS_ROLE_ARN != '' }}
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: eu-central-1
          role-session-name: GitHubActions
      
      - name: 🚀 Main Step
        id: main-step
        run: |
          echo "Processing ${{ inputs.required_input }}..."
          RESULT="success"
          echo "value=$RESULT" >> $GITHUB_OUTPUT
      
      - name: 📊 Summary
        if: always()
        run: |
          echo "### Workflow Summary 📊" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Input:** ${{ inputs.required_input }}" >> $GITHUB_STEP_SUMMARY
          echo "**Status:** ${{ job.status }}" >> $GITHUB_STEP_SUMMARY
```

### Step 2: Define Input Types

**Available types:**
```yaml
inputs:
  string_input:
    type: string     # Text value
  boolean_input:
    type: boolean    # true/false
  number_input:
    type: number     # Numeric value
```

**Validation patterns:**
```yaml
# Required input
required_input:
  required: true

# Optional with default
optional_input:
  required: false
  default: 'default-value'
```

### Step 3: Define Outputs

```yaml
on:
  workflow_call:
    outputs:
      version:
        description: 'Released version'
        value: ${{ jobs.release.outputs.version }}
      success:
        description: 'Whether operation succeeded'
        value: ${{ jobs.main.outputs.success }}

jobs:
  release:
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - id: version
        run: echo "value=v1.2.3" >> $GITHUB_OUTPUT
```

### Step 4: Add Error Handling

```yaml
steps:
  - name: Validate inputs
    run: |
      if [ -z "${{ inputs.required_input }}" ]; then
        echo "❌ Error: required_input is empty"
        exit 1
      fi

  - name: Main operation
    id: operation
    continue-on-error: false
    run: |
      # Operation that might fail
      some_command || exit 1

  - name: Cleanup on failure
    if: failure()
    run: |
      echo "Cleaning up after failure..."
```

### Step 5: Create Usage Documentation

**File:** `examples/{workflow-name}.md`

```markdown
# Using {Workflow Name}

## Prerequisites
- List requirements

## Basic Usage
\`\`\`yaml
jobs:
  example:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-{name}.yml@v2.0.0
    with:
      required_input: value
    secrets:
      AWS_ROLE_ARN: \${{ secrets.AWS_ROLE_ARN }}
\`\`\`

## Inputs Reference
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `required_input` | Yes | — | Description |

## Outputs
| Output | Description |
|--------|-------------|
| `result` | What it returns |
```

### Step 6: Test Workflow

**In a test service repo:**
```yaml
# .github/workflows/test-new-workflow.yml
name: Test New Workflow

on:
  workflow_dispatch:

jobs:
  test:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-{name}.yml@feature/new-workflow
    with:
      required_input: 'test-value'
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

### Step 7: Create Release

```bash
git add .
git commit -m "feat: add new workflow for X"
git push origin feature/new-workflow

# After PR merge
git checkout main
git pull
git tag v2.1.0
git push origin v2.1.0
```

---

## ✅ Checklist

- [ ] Workflow file created with `workflow_call` trigger
- [ ] All inputs documented with descriptions
- [ ] Required secrets defined
- [ ] Outputs defined and documented
- [ ] Error handling implemented
- [ ] Permissions block added
- [ ] Summary step for GitHub Actions UI
- [ ] Usage documentation in `examples/`
- [ ] Tested with service repository
- [ ] Version tagged after merge

---

## Best Practices

### Naming
```
reusable-{scope}-{action}.yml
Examples:
  reusable-ci-docker.yml
  reusable-release-ecr.yml
  reusable-terraform-deploy.yml
```

### Emoji Prefixes for Steps
```
📥 Checkout
🔐 Authentication/Secrets
🐳 Docker operations
🏗️ Build operations
🧪 Test operations
🚀 Deploy operations
📊 Summary/Reporting
✅ Validation
❌ Error handling
```

### Input Validation
```yaml
- name: ✅ Validate inputs
  run: |
    [[ -n "${{ inputs.service_name }}" ]] || { echo "service_name required"; exit 1; }
```
