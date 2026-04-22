# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] - 2026-04-22

### Added
- **reusable-trigger-api-gateway-deploy.yml** — Triggers `workflow_dispatch` on `ivodenwag/tf-aws-api`
  - Input: `service_name` (e.g. `identity`, `cook`)
  - Requires `GH_PAT` secret with workflow write permissions on tf-aws-api
  - Fails with exit 1 if dispatch returns non-204 HTTP status

## [2.0.0] - 2026-02-06

### Added
- **reusable-service-deployment.yml** - Master orchestration workflow
- **reusable-terraform-deploy.yml** - Terraform infrastructure deployment
- **reusable-ecs-codedeploy.yml** - ECS Blue/Green deployment via CodeDeploy
- Complete ECS CodeDeploy integration with Blue/Green strategy
- Active deployment check (prevents concurrent deployments)
- Hybrid wait strategy (3 min initial health check + background)
- Service ownership principle (services provide appspec.yaml)
- Terraform plan artifact upload (30 days retention, audit trail)
- AWS OIDC authentication (no static credentials)
- Error handling with conditional job execution
- GitHub Step Summaries for all workflows
- AWS Console links in deployment summaries
- Comprehensive documentation and examples

### Changed
- Service workflows now use master orchestration workflow
- Simplified service repository workflow files (only configuration)
- Task Definition updates preserve secrets (downloaded + image update only)
- Workflow versioning best practice (@v2.0.0 instead of @main)

### Documentation
- Added examples/deploy-ecs-with-codedeploy.md (complete guide)
- Updated README.md with all workflow descriptions
- Added workflow versioning strategy
- Added security and monitoring sections
- Services must provide terraform/appspec.yaml (Service Ownership)

## [Unreleased]

### Changed
- Simplified to 2-stage approach: development and production (removed QA stage)
- Migrated from npm-based CI to Docker Compose based CI
- All commands now execute via Makefile for consistency
- Removed node-version and working-directory inputs (defined in Dockerfile)
- Version detection from Git tags for production releases
- Coverage reports stored as GitHub Artifacts instead of Codecov

### Added
- Docker Compose support for multi-container environments (DB, Redis, etc.)
- Automatic version tagging from Git tags (v1.0.0)
- Docker image labels with version and commit information
- ECR deployment workflow (reusable-deploy-ecr.yml)
- Production release summary in workflow output
- Coverage summary displayed in GitHub Actions UI

### Removed
- QA environment support
- Codecov integration (replaced with GitHub Artifacts)
- Direct npm command execution (now via Make)

## [1.0.0] - 2026-01-02

### Added
- Initial release of centralized GitHub Actions workflows
- Reusable CI workflow for Node.js projects
- Environment-specific configurations
- Documentation and usage examples
