# GitHub Workflows Repository

Centralized, reusable GitHub Actions workflows for all services.

## 🎯 Purpose

This repository contains reusable workflows that can be called from any service repository to standardize CI/CD processes across the organization.

## 📁 Structure

```
github-workflows/
├── .github/
│   └── workflows/
│       ├── reusable-ci.yml          # Reusable CI workflow
│       └── reusable-deploy-ecr.yml  # ECR deployment workflow
├── README.md
└── CHANGELOG.md
```

## 🚀 Usage

### In Your Service Repository

Create workflow files in `.github/workflows/`:

**Development CI (feature branches):**
```yaml
# .github/workflows/ci-development.yml
name: CI - Development

on:
  push:
    branches: ['feature/**', 'bugfix/**', 'hotfix/**']
  pull_request:
    branches: ['main']

jobs:
  ci:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci.yml@main
    with:
      environment: development
      run-tests: true
      coverage-threshold: 70
```

**Production CI & Deployment (main branch & tags):**
```yaml
# .github/workflows/ci-production.yml
name: CI/CD - Production

on:
  push:
    branches: ['main']
    tags: ['v*']

jobs:
  ci:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci.yml@main
    with:
      environment: production
      run-tests: true
      coverage-threshold: 80

  deploy:
    needs: ci
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    uses: ivodenwag/github-workflows/.github/workflows/reusable-deploy-ecr.yml@main
    with:
      environment: production
      version: ${{ github.ref_name }}
      ecr-repository: 'your-service-name'
      aws-region: 'eu-central-1'
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 🔧 Workflow Inputs

### `reusable-ci.yml`

| Input | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `environment` | ✅ | string | - | Environment: development, production |
| `run-tests` | ❌ | boolean | `true` | Whether to run tests |
| `coverage-threshold` | ❌ | number | `80` | Minimum test coverage percentage |

### `reusable-deploy-ecr.yml`

| Input | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
### Service Repository

Your service repository must have:

- **Docker Compose** setup with service named `skeleton-service`
- **Makefile** with these targets:
  - `make install` - Install dependencies
  - `make lint` - Lint code
  - `make type-check` - Type check
  - `make api-lint` - Validate OpenAPI spec (if applicable)
  - `make test-coverage` - Run tests with coverage
  - `make build` - Build application
- **Dockerfile** at `.docker/Dockerfile` with stages: `builder`, `runner`

### AWS Setup (for deployment)

- ECR Repository created
- AWS IAM User with permissions:
  - `ecr:GetAuthorizationToken`
  - `ecr:BatchCheckLayerAvailability`
  - `ecr:PutImage`
  - `ecr:InitiateLayerUpload`
  - `ecr:UploadLayerPart`
  - `ecr:CompleteLayerUpload`

Your service repository must have:

- `Makefile` with these targets:
  - `make install` - Install dependencies
  - `make lint` - Lint code
  - `make type-check` - Type check
  - `make api-lint` - Validate OpenAPI spec (if applicable)
  - `make test` - Run tests
  - `make test-coverage` - Run tests with coverage
  - `make build` - Build application

## 🔄 Versioning

Use 2.0.0` - Stable releases
- `v2` - Latest v2.x.x
- `main` - Latest development version

## 🎯 Workflow Features

### CI Workflow
- **Docker Compose based** - Runs in containers with DB/Redis support
- **Make-driven** - All commands via Makefile
- **Automatic version detection** - From Git tags or commit SHA
- **Coverage reports** - Stored as GitHub Artifacts
- **Multi-stage Docker builds** - Optimized caching

### ECR Deployment
- **AWS ECR push** - Automated image deployment
- **Smart tagging** - Version tags + latest for releases
- **Cache optimization** - Faster builds with layer caching
- `main` - Latest development version

## 📝 Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Test with a service repository
4. Create a pull request
5. Tag a new version after merge

## 📄 License

MIT
