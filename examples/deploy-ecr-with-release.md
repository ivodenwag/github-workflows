# Example: Deploy to ECR with Release

This example shows how to use the reusable workflow for releasing and deploying to AWS ECR.

## Prerequisites

### 1. Add GitHub Secrets

In your repository settings → Secrets and variables → Actions:

- `AWS_ROLE_ARN`: `arn:aws:iam::616707572453:role/github-actions-ecr-oidc`
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions

**Note:** No need to install release-it or copy config files! The reusable workflow uses a shared `.release-it.json` from the github-workflows repository.

### 2. Create workflow file

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to ECR

on:
  push:
    branches:
      - master

jobs:
  deploy:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-release-ecr.yml@main
    with:
      service_name: identity-service
      ecr_repository: identity
      aws_region: eu-central-1
      compose_files: 'docker-compose.yml docker-compose.ci.yml'
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## How it works

1. **Push to master** → Workflow triggers
2. **CI Tests** → Lint, type-check, build, test
3. **Release** → release-it analyzes commits and creates version
4. **Docker Build** → Builds image with new version tag
5. **ECR Push** → Pushes to AWS ECR with version + latest tags

## Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# New feature → minor version bump (1.0.0 → 1.1.0)
git commit -m "feat: add two-factor authentication"

# Bug fix → patch version bump (1.0.0 → 1.0.1)
git commit -m "fix: validate email format correctly"

# Breaking change → major version bump (1.0.0 → 2.0.0)
git commit -m "feat!: redesign authentication API"

# With body and footer
git commit -m "feat: add user roles

Adds admin, editor, and viewer roles with permissions.

BREAKING CHANGE: removes superuser flag"
```

## Version Bump Rules

| Commit Type | Example | Version Bump |
|-------------|---------|--------------|
| `feat:` | New feature | Minor (1.0.0 → 1.1.0) |
| `fix:` | Bug fix | Patch (1.0.0 → 1.0.1) |
| `feat!:` or `BREAKING CHANGE:` | Breaking change | Major (1.0.0 → 2.0.0) |
| `docs:` | Documentation | No release |
| `chore:` | Maintenance | No release |
| `refactor:` | Code refactor | Patch (configurable) |

## What gets created

- ✅ Git tag (e.g., `v1.2.3`)
- ✅ GitHub Release with changelog
- ✅ Updated CHANGELOG.md
- ✅ Docker images in ECR:
  - `identity:v1.2.3`
  - `identity:latest`

## Troubleshooting

### "Nothing to release"

No commits with `feat:` or `fix:` since last release. Add a commit or use:

```bash
npx release-it patch --ci  # Force patch bump
```

### "Permission denied" during release

Ensure `contents: write` permission in workflow file (already included in reusable workflow).

### Docker tag mismatch

The workflow checks out the exact git tag after release to ensure the Docker image matches the version.
