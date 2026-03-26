# Development — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Development Workflow

### 1. Create Feature Branch

```bash
git checkout -b feature/new-workflow
```

### 2. Develop Workflow

Create or modify workflow in `.github/workflows/`:

```yaml
# .github/workflows/reusable-new-workflow.yml
name: New Workflow

on:
  workflow_call:
    inputs:
      param1:
        description: 'Description'
        required: true
        type: string
    secrets:
      SECRET1:
        required: false

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello ${{ inputs.param1 }}"
```

### 3. Test in Service Repository

Reference your branch in a test service:

```yaml
# In service repo: .github/workflows/test.yml
jobs:
  test:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-new-workflow.yml@feature/new-workflow
    with:
      param1: "test-value"
```

### 4. Create Pull Request

```bash
git add .
git commit -m "feat: add new workflow for X"
git push origin feature/new-workflow
```

### 5. Merge & Tag

After merge to main:

```bash
git checkout main
git pull
git tag v2.1.0
git push origin v2.1.0
```

---

## Testing Workflows

### Local Validation

```bash
# Validate YAML syntax
yamllint .github/workflows/*.yml

# Check with actionlint (if installed)
actionlint
```

### Test in Fork

1. Fork this repository
2. Create workflow in fork
3. Test with `workflow_dispatch`
4. Verify all steps pass

### Test with Service

```yaml
# Temporarily use branch reference
uses: your-fork/github-workflows/.github/workflows/workflow.yml@your-branch
```

---

## Common Tasks

### Add New Workflow

1. Create file: `.github/workflows/reusable-{name}.yml`
2. Define `workflow_call` trigger
3. Add inputs, secrets, outputs
4. Implement jobs
5. Test with service
6. Document in `examples/`

### Update Existing Workflow

1. Create feature branch
2. Make changes
3. Test with service using branch reference
4. Create PR
5. After merge, tag new version

### Update Shared Config

```bash
# Edit shared/.release-it.json
# Workflows download this at runtime via curl
```

### Add Documentation

1. Create example in `examples/`
2. Update README.md
3. Update `.ai/` documentation

---

## Workflow Structure Template

```yaml
name: Descriptive Name

on:
  workflow_call:
    inputs:
      required_input:
        description: 'What this does'
        required: true
        type: string
      optional_input:
        description: 'Optional parameter'
        required: false
        type: string
        default: 'default-value'
    secrets:
      AWS_ROLE_ARN:
        description: 'IAM Role for OIDC'
        required: true
    outputs:
      output_name:
        description: 'What this returns'
        value: ${{ jobs.main.outputs.result }}

permissions:
  id-token: write    # For OIDC
  contents: read     # For checkout

jobs:
  main:
    name: Job Name
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.step-id.outputs.value }}
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
      
      - name: 🔐 Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: eu-central-1
      
      - name: 🚀 Main Step
        id: step-id
        run: |
          echo "value=result" >> $GITHUB_OUTPUT
      
      - name: 📊 Summary
        if: always()
        run: |
          echo "### Summary 📊" >> $GITHUB_STEP_SUMMARY
          echo "Status: ${{ job.status }}" >> $GITHUB_STEP_SUMMARY
```

---

## Version Tagging

### Semantic Versioning

```bash
# Patch (bug fixes)
git tag v2.0.1

# Minor (new features, backward compatible)
git tag v2.1.0

# Major (breaking changes)
git tag v3.0.0
```

### Tag Management

```bash
# List tags
git tag -l "v2.*"

# Delete tag (local)
git tag -d v2.0.0

# Delete tag (remote)
git push origin :refs/tags/v2.0.0

# Push all tags
git push origin --tags
```

---

## Debugging

### Enable Debug Logging

In GitHub Actions settings, add secret:
```
ACTIONS_STEP_DEBUG=true
```

### View Workflow Runs

```
https://github.com/ivodenwag/github-workflows/actions
```

### Check Calling Workflow

When testing from a service, check both:
1. Service workflow run
2. Called workflow (nested in service run)

---

## Further Documentation

[Quick Start](00-quick-start.md) | [Architecture](02-architecture.md) | [Troubleshooting](10-troubleshooting.md)
