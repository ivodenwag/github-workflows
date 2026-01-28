# Shared Configuration Files

This directory contains centralized configuration files used across all services.

## `.release-it.json`

Shared release-it configuration for semantic versioning and releases.

**Used by:** All services using `reusable-release-ecr.yml`

**Features:**
- ✅ Conventional Commits based versioning
- ✅ Automatic CHANGELOG.md generation
- ✅ GitHub Releases with auto-generated notes
- ✅ Git tags with semantic versioning

**How it works:**

The `reusable-release-ecr.yml` workflow automatically downloads this config:

```yaml
- name: 📥 Download shared release config
  run: |
    curl -sL -o .release-it.json \
      https://raw.githubusercontent.com/ivodenwag/github-workflows/main/shared/.release-it.json
```

**Modify once, applies everywhere!** ✨

## Adding New Shared Configs

To add new shared configurations:

1. Add file to this directory
2. Update reusable workflows to download it
3. Document usage here

## Testing Changes

Before committing changes to shared configs:

```bash
# Test in a service locally
cd ~/Projects/Services/identity
curl -sL https://raw.githubusercontent.com/ivodenwag/github-workflows/YOUR_BRANCH/shared/.release-it.json > .release-it.json
npx release-it@latest --dry-run
```
