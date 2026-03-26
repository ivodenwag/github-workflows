# Development Principles — github-workflows

**Version**: 1.0.0  
**Last Updated**: 26. März 2026

---

## Core Principles

### 1. Atomic Workflows

Each workflow does **ONE thing well**:

```
reusable-ci-docker.yml       → ONLY run tests
reusable-release-ecr.yml     → ONLY release & push image
reusable-terraform-deploy.yml → ONLY apply terraform
reusable-ecs-codedeploy.yml  → ONLY deploy to ECS
```

**Why**: Composability, testability, single responsibility.

---

### 2. Orchestration via Chaining

Combine atomic workflows for complex pipelines:

```yaml
jobs:
  release-ecr:
    uses: ./.github/workflows/reusable-release-ecr.yml
  
  deploy-infra:
    needs: release-ecr  # Sequential
    uses: ./.github/workflows/reusable-terraform-deploy.yml
  
  deploy-ecs:
    needs: [release-ecr, deploy-infra]  # Waits for both
    uses: ./.github/workflows/reusable-ecs-codedeploy.yml
    with:
      image_tag: ${{ needs.release-ecr.outputs.version }}
```

---

### 3. Service Ownership

Services own their configuration:

| File | Location | Owner |
|------|----------|-------|
| `appspec.yaml` | `{service}/terraform/appspec.yaml` | Service team |
| `docker-compose.yml` | `{service}/docker-compose.yml` | Service team |
| `Makefile` | `{service}/Makefile` | Service team |

**Workflows provide**:
- Standardized steps
- Input validation
- Error handling

**Services provide**:
- Configuration files
- Build commands
- Deployment specs

---

### 4. Fail Fast

Detect problems early:

```yaml
steps:
  - name: Lint       # First - fastest
  - name: Type Check # Second - fast
  - name: Build      # Third - medium
  - name: Test       # Last - slowest
```

---

### 5. Idempotency

Workflows can be re-run safely:

```yaml
- name: Check for active deployments
  run: |
    ACTIVE=$(aws deploy list-deployments --include-only-statuses InProgress)
    if [ "$ACTIVE" != "None" ]; then
      echo "Deployment already in progress"
      exit 1
    fi
```

---

### 6. Security First

- **OIDC Authentication**: No static AWS credentials
- **Least Privilege**: Minimal IAM permissions per workflow
- **No Secrets in Logs**: Use `::add-mask::` for sensitive values
- **Pinned Versions**: Use `@v4` not `@main` in production

---

## Design Patterns

### Pattern 1: Output Passing

```yaml
jobs:
  job1:
    outputs:
      version: ${{ steps.release.outputs.version }}
    steps:
      - id: release
        run: echo "version=v1.2.3" >> $GITHUB_OUTPUT

  job2:
    needs: job1
    steps:
      - run: echo "Using ${{ needs.job1.outputs.version }}"
```

### Pattern 2: Conditional Execution

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    # Only runs on main branch
```

### Pattern 3: Matrix Strategy

```yaml
jobs:
  test:
    strategy:
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

### Pattern 4: Artifact Passing

```yaml
jobs:
  build:
    steps:
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  deploy:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
```

---

## Versioning Strategy

### Semantic Versioning

```
v2.0.0  ← MAJOR: Breaking changes
v2.1.0  ← MINOR: New features (backward compatible)
v2.1.1  ← PATCH: Bug fixes
```

### Workflow References

| Environment | Reference | Example |
|-------------|-----------|---------|
| Production | Specific tag | `@v2.0.0` |
| Staging | Major version | `@v2` |
| Development | Branch | `@main` |

---

## Further Documentation

[Architecture](02-architecture.md) | [Development](07-development.md) | [Troubleshooting](10-troubleshooting.md)
