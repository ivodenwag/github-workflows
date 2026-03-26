# Release & ECR Scenarios — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Overview

Release and ECR push workflow scenarios for semantic versioning and Docker image deployment.

---

## Scenario 1: Create Release and Push to ECR

### Description

Analyze commits, create semantic version release, build Docker image, and push to ECR.

### User Story

As a developer, I want automatic versioning on merge to main, so that I have consistent releases.

### Workflow

`reusable-release-ecr.yml`

### Inputs

```yaml
service_name: 'identity'
ecr_repository: 'identity'
aws_region: 'eu-central-1'
```

### Secrets

```yaml
AWS_ROLE_ARN: 'arn:aws:iam::123456789:role/GitHubActionsRole'
```

### Usage Example

```yaml
# In service repo: .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@v2.0.0
    with:
      service_name: identity
      ecr_repository: identity
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

### Flow

1. Run tests (lint, type-check, build, test)
2. Checkout with full history (`fetch-depth: 0`)
3. Download shared `.release-it.json`
4. Run release-it (analyze commits, bump version)
5. Create git tag and GitHub release
6. Build Docker image
7. Push to ECR with version tag

### Outputs

```yaml
version: 'v1.2.3'  # Released version tag
```

### Test Scenarios

#### ✅ Happy Path

- Commit with `feat:` prefix → Minor version bump (v1.1.0 → v1.2.0)
- Commit with `fix:` prefix → Patch version bump (v1.2.0 → v1.2.1)
- Image pushed to ECR with version tag

#### ❌ Error Cases

- **No conventional commits**: No release created, branch version used
- **OIDC fails**: `Not authorized to perform sts:AssumeRoleWithWebIdentity`
- **ECR push fails**: `denied: Your authorization token has expired`

### Key Files

- `.github/workflows/reusable-release-ecr.yml` — Workflow definition
- `shared/.release-it.json` — Release-it configuration
- `docker-compose.yml` (service repo) — Build configuration
- `.docker/Dockerfile` (service repo) — Docker build

### Business Rules

- Releases only on `main` branch
- Conventional Commits required for version bumps
- OIDC authentication (no static credentials)

---

## Scenario 2: Skip Tests on Release

### Description

Release without running tests (for deploy-only workflows where tests ran in PR).

### User Story

As a team with separate CI and CD workflows, I want to skip tests during release, so that deployment is faster.

### Inputs

```yaml
service_name: 'identity'
ecr_repository: 'identity'
skip_tests: true
```

### Usage Example

```yaml
jobs:
  release:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@v2.0.0
    with:
      service_name: identity
      ecr_repository: identity
      skip_tests: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

### Test Scenarios

#### ✅ Happy Path

- Tests skipped, release created
- Faster workflow execution

#### ❌ Error Cases

- Build fails (tests would have caught the issue)

---

## Scenario 3: Branch Build (No Release)

### Description

Build and push image from feature branch without creating a release.

### User Story

As a developer, I want to test my branch in staging, so that I can verify changes before merge.

### Trigger

Push to non-main branch with `workflow_dispatch`

### Flow

1. Run tests
2. Skip release-it (not on main)
3. Build Docker image
4. Push to ECR with branch-SHA tag (e.g., `feature-abc-a1b2c3d4`)

### Test Scenarios

#### ✅ Happy Path

- Image pushed with branch tag
- No git tag or GitHub release created

#### ❌ Error Cases

- Branch name contains invalid characters
