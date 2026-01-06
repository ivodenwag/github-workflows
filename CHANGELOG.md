# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
