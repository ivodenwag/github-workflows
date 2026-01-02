# GitHub Workflows Repository

Centralized, reusable GitHub Actions workflows for all services.

## 🎯 Purpose

This repository contains reusable workflows that can be called from any service repository to standardize CI/CD processes across the organization.

## 📁 Structure

```
github-workflows/
├── .github/
│   └── workflows/
│       └── reusable-ci.yml          # Reusable CI workflow
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
    branches: ['main', 'release/**', 'qa']

jobs:
  ci:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci.yml@main
    with:
      environment: development
      node-version: '20'
      working-directory: 'app'
```

**QA CI (release branches):**
```yaml
# .github/workflows/ci-qa.yml
name: CI - QA

on:
  push:
    branches: ['release/**', 'qa']

jobs:
  ci:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci.yml@main
    with:
      environment: qa
      node-version: '20'
      working-directory: 'app'
```

**Production CI (main branch):**
```yaml
# .github/workflows/ci-production.yml
name: CI - Production

on:
  push:
    branches: ['main']

jobs:
  ci:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci.yml@main
    with:
      environment: production
      node-version: '20'
      working-directory: 'app'
```

## 🔧 Workflow Inputs

### `reusable-ci.yml`

| Input | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `environment` | ✅ | string | - | Environment: development, qa, production |
| `node-version` | ❌ | string | `'20'` | Node.js version |
| `working-directory` | ❌ | string | `'app'` | Working directory for npm commands |
| `run-tests` | ❌ | boolean | `true` | Whether to run tests |
| `coverage-threshold` | ❌ | number | `80` | Minimum test coverage percentage |

## 📋 Requirements

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

Use semantic versioning with tags:
- `v1.0.0` - Stable releases
- `v1` - Latest v1.x.x
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
