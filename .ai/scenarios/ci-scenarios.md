# CI Testing Scenarios — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Overview

CI testing workflow scenarios for running tests in Docker Compose environment.

---

## Scenario 1: Run CI Tests

### Description

Execute lint, type-check, build, and test in Docker Compose environment.

### User Story

As a developer, I want to run CI tests on every push, so that I catch issues early.

### Workflow

`reusable-ci-docker.yml`

### Inputs

```yaml
compose_files: 'docker-compose.yml docker-compose.ci.yml'
service_name: 'identity'
```

### Usage Example

```yaml
# In service repo: .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci-docker.yml@v2.0.0
    with:
      service_name: identity
      compose_files: 'docker-compose.yml docker-compose.ci.yml'
```

### Flow

1. Checkout code
2. Create dummy secrets (`secrets/`)
3. Build Docker images
4. Run `make lint`
5. Run `make type-check`
6. Run `make build`
7. Run `make test`

### Test Scenarios

#### ✅ Happy Path

- All tests pass → Workflow succeeds with green status
- No secrets required → Uses dummy secrets

#### ❌ Error Cases

- **Lint fails**: Exit code 1 at lint step
- **Type check fails**: Exit code 1 at type-check step
- **Build fails**: Exit code 1 at build step
- **Test fails**: Exit code 1 at test step

### Key Files

- `.github/workflows/reusable-ci-docker.yml` — Workflow definition
- `docker-compose.yml` (service repo) — Development compose
- `docker-compose.ci.yml` (service repo) — CI compose override
- `Makefile` (service repo) — lint, type-check, build, test targets

### Service Requirements

Service must have Makefile with targets:
```makefile
lint:
	docker compose run --rm $(SERVICE) npm run lint

type-check:
	docker compose run --rm $(SERVICE) npm run type-check

build:
	docker compose run --rm $(SERVICE) npm run build

test:
	docker compose run --rm $(SERVICE) npm test
```

---

## Scenario 2: CI with Custom Compose Files

### Description

Run CI with non-standard Docker Compose configuration.

### User Story

As a developer with custom Docker setup, I want to specify which compose files to use, so that CI matches my environment.

### Inputs

```yaml
compose_files: 'docker-compose.yml docker-compose.test.yml docker-compose.ci.yml'
service_name: 'api'
```

### Usage Example

```yaml
jobs:
  test:
    uses: ivodenwag/github-workflows/.github/workflows/reusable-ci-docker.yml@v2.0.0
    with:
      service_name: api
      compose_files: 'docker-compose.yml docker-compose.test.yml docker-compose.ci.yml'
```

### Test Scenarios

#### ✅ Happy Path

- Multiple compose files merged correctly
- Environment variables from all files available

#### ❌ Error Cases

- **File not found**: `docker-compose.test.yml: No such file or directory`
- **Invalid YAML**: Parse error at compose file
