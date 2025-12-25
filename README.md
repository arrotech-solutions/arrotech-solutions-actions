# 🚀 Arrotech Solutions - Centralized GitHub Actions

[![Organization](https://img.shields.io/badge/org-arrotech--solutions-blue)](https://github.com/arrotech-solutions)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

This repository contains **centralized, reusable GitHub Actions workflows** for all repositories in the Arrotech Solutions organization. By using these shared workflows, we ensure consistency, security, and maintainability across all our projects.

## 📋 Table of Contents

- [Quick Start](#-quick-start)
- [Available Workflows](#-available-workflows)
- [Templates](#-templates)
- [Organization Secrets](#-organization-secrets)
- [Best Practices](#-best-practices)
- [Contributing](#-contributing)

---

## 🏃 Quick Start

### 1. Create a workflow file in your repository

Create `.github/workflows/ci.yml` in your repository:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Code Quality
  quality:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-code-quality.yml@main
    with:
      language: 'node'  # or: python, go, multi

  # Security Scanning
  security:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-security-scan.yml@main
    with:
      language: 'javascript'

  # Automated Tests
  test:
    needs: quality
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-test.yml@main
    with:
      language: 'node'
      coverage-threshold: 80
```

### 2. Configure organization secrets (one-time setup)

Go to **Organization Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|--------|-------------|
| `KUBE_CONFIG_STAGING` | Kubeconfig for staging cluster |
| `KUBE_CONFIG_PROD` | Kubeconfig for production cluster |
| `SNYK_TOKEN` | Snyk API token (optional) |
| `NPM_TOKEN` | npm registry token (optional) |
| `PYPI_TOKEN` | PyPI API token (optional) |

---

## 📦 Available Workflows

### CI Workflows (Continuous Integration)

#### `ci-code-quality.yml`
Runs linting, formatting, and type checking.

```yaml
jobs:
  lint:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-code-quality.yml@main
    with:
      language: 'node'          # node, python, go, or multi
      node-version: '20'        # Node.js version
      python-version: '3.11'    # Python version
      go-version: '1.21'        # Go version
      prettier-check: true      # Run Prettier (Node.js)
      fail-on-warnings: false   # Fail on lint warnings
```

**Supported languages:**
| Language | Tools Used |
|----------|------------|
| Node.js | ESLint, Prettier, TypeScript |
| Python | Ruff, Black, mypy |
| Go | golangci-lint |

---

#### `ci-security-scan.yml`
Comprehensive security scanning including SAST, dependency audit, and secret detection.

```yaml
jobs:
  security:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-security-scan.yml@main
    with:
      language: 'javascript'         # CodeQL language
      enable-codeql: true            # GitHub CodeQL scanning
      enable-dependency-scan: true   # Vulnerability scanning
      enable-secret-scan: true       # Detect exposed secrets
      enable-container-scan: false   # Scan Docker images
      fail-on-high-severity: true    # Fail on high/critical issues
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}  # Optional
```

**Security tools included:**
- 🔍 **CodeQL** - Static Application Security Testing (SAST)
- 📦 **npm audit / pip-audit / govulncheck** - Dependency vulnerability scanning
- 🔐 **Gitleaks & TruffleHog** - Secret detection
- 🐳 **Trivy** - Container image scanning

---

#### `ci-test.yml`
Runs automated tests with coverage reporting.

```yaml
jobs:
  test:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-test.yml@main
    with:
      language: 'node'              # node, python, go
      test-command: ''              # Custom test command (optional)
      coverage-threshold: 80        # Minimum coverage % (0 = disabled)
      upload-coverage: true         # Upload coverage artifact
      run-integration-tests: false  # Run integration tests
      parallel-tests: true          # Run tests in parallel
```

---

### CD Workflows (Continuous Deployment)

#### `cd-build-artifact.yml`
Builds and publishes artifacts (Docker images, npm packages, Python packages, Go binaries).

```yaml
jobs:
  build:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/cd-build-artifact.yml@main
    with:
      artifact-type: 'docker'       # docker, npm, python, go-binary
      registry: 'ghcr'              # ghcr, dockerhub
      push-image: true              # Push to registry
      docker-platforms: 'linux/amd64,linux/arm64'  # Multi-arch
    secrets:
      REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Artifact types:**

| Type | Output | Registry |
|------|--------|----------|
| `docker` | Container image | GHCR, Docker Hub |
| `npm` | npm package | npmjs.com |
| `python` | Wheel/sdist | PyPI |
| `go-binary` | Compiled binaries | GitHub Artifacts |

---

#### `cd-deploy.yml`
Deploys to various platforms and environments.

```yaml
jobs:
  deploy:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/cd-deploy.yml@main
    with:
      environment: 'staging'        # staging, production
      platform: 'kubernetes'        # kubernetes, ecs, flyio, vercel, netlify, ssh
      image: 'ghcr.io/org/app:v1.0' # Container image
      namespace: 'staging'          # K8s namespace
      wait-for-rollout: true        # Wait for deployment
      dry-run: false                # Test without deploying
    secrets:
      KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
```

**Supported platforms:**

| Platform | Use Case |
|----------|----------|
| `kubernetes` | K8s clusters (Helm or kubectl) |
| `ecs` | AWS Elastic Container Service |
| `flyio` | **Fly.io backend apps** |
| `vercel` | **Vercel frontend apps** |
| `netlify` | Netlify static sites |
| `ssh` | Direct SSH deployment |

---

#### `pr-checks.yml`
Validates pull requests for quality and compliance.

```yaml
jobs:
  pr-checks:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/pr-checks.yml@main
    with:
      require-conventional-commits: true  # Enforce commit format
      max-files-changed: 50              # Limit PR size
      block-wip: true                    # Block WIP PRs
      require-labels: true               # Require labels
```

---

#### `release.yml`
Automates releases with semantic versioning and changelog generation.

```yaml
jobs:
  release:
    uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/release.yml@main
    with:
      release-type: 'auto'          # auto, major, minor, patch
      generate-changelog: true      # Auto-generate changelog
      update-package-version: true  # Update package.json/pyproject.toml
      create-github-release: true   # Create GitHub release
    secrets:
      RELEASE_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 📄 Templates

Ready-to-use workflow templates for common project types:

| Template | Description | Copy Command |
|----------|-------------|--------------|
| [`node-full-pipeline.yml`](templates/node-full-pipeline.yml) | Complete Node.js CI/CD with Docker | `cp templates/node-full-pipeline.yml .github/workflows/` |
| [`python-full-pipeline.yml`](templates/python-full-pipeline.yml) | Complete Python CI/CD | `cp templates/python-full-pipeline.yml .github/workflows/` |
| [`minimal-ci.yml`](templates/minimal-ci.yml) | Lightweight CI for small projects | `cp templates/minimal-ci.yml .github/workflows/` |
| [`vercel-deploy.yml`](templates/vercel-deploy.yml) | **Frontend** deployment to Vercel | `cp templates/vercel-deploy.yml .github/workflows/` |
| [`flyio-deploy.yml`](templates/flyio-deploy.yml) | **Backend** deployment to Fly.io | `cp templates/flyio-deploy.yml .github/workflows/` |

---

## 🔐 Organization Secrets

Configure these secrets at the **organization level** for all repositories to inherit:

### Required for Deployments

| Secret | Description | Used By |
|--------|-------------|---------|
| `KUBE_CONFIG_STAGING` | Base64-encoded kubeconfig for staging | `cd-deploy.yml` |
| `KUBE_CONFIG_PROD` | Base64-encoded kubeconfig for production | `cd-deploy.yml` |
| `AWS_ACCESS_KEY_ID` | AWS credentials for ECS | `cd-deploy.yml` |
| `AWS_SECRET_ACCESS_KEY` | AWS credentials for ECS | `cd-deploy.yml` |

### Optional Enhancements

| Secret | Description | Used By |
|--------|-------------|---------|
| `SNYK_TOKEN` | Snyk API token | `ci-security-scan.yml` |
| `NPM_TOKEN` | npm registry token | `cd-build-artifact.yml` |
| `PYPI_TOKEN` | PyPI API token | `cd-build-artifact.yml` |
| `VERCEL_TOKEN` | Vercel deployment token | `cd-deploy.yml` |
| `FLY_API_TOKEN` | Fly.io API token | `cd-deploy.yml` |
| `NETLIFY_AUTH_TOKEN` | Netlify auth token | `cd-deploy.yml` |

### Setting Organization Secrets

1. Go to your GitHub organization
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New organization secret**
4. Configure repository access (all repos or selected repos)

---

## 🎯 Best Practices

### 1. Pin Workflow Versions

Use specific tags or commit SHAs for stability in production:

```yaml
# ✅ Good - pinned to a specific version
uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-test.yml@v1.0.0

# ✅ Good - pinned to a commit SHA
uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-test.yml@abc1234

# ⚠️ Use with caution - always gets latest
uses: arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-test.yml@main
```

### 2. Use Concurrency Control

Prevent duplicate runs and save resources:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 3. Set Up Environment Protection Rules

For production deployments, configure environment protection:

1. Go to **Repository Settings → Environments**
2. Create `production` environment
3. Add required reviewers
4. Set deployment branch rules

### 4. Leverage Caching

All workflows use appropriate caching (npm, pip, Go modules). Ensure your `package-lock.json`, `requirements.txt`, or `go.sum` files are committed.

### 5. Monitor Workflow Runs

Use the GitHub Actions dashboard at the organization level to monitor all workflow runs across repositories.

---

## 🔄 Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Pull Request / Push                          │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────┐
         │              CI Stage 1: Quality            │
         │  ┌─────────────┐    ┌──────────────────┐   │
         │  │ Code Quality│    │  Security Scan   │   │
         │  │  (Linting)  │    │ (CodeQL, Deps)   │   │
         │  └─────────────┘    └──────────────────┘   │
         └─────────────────────────────────────────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────┐
         │              CI Stage 2: Testing            │
         │  ┌─────────────┐    ┌──────────────────┐   │
         │  │ Unit Tests  │    │Integration Tests │   │
         │  │ + Coverage  │    │   (Optional)     │   │
         │  └─────────────┘    └──────────────────┘   │
         └─────────────────────────────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
              (on develop)                  (on main/release)
                    │                             │
                    ▼                             ▼
         ┌──────────────────┐          ┌──────────────────┐
         │   Build & Push   │          │   Build & Push   │
         │  (Docker/Package)│          │  (Docker/Package)│
         └──────────────────┘          └──────────────────┘
                    │                             │
                    ▼                             ▼
         ┌──────────────────┐          ┌──────────────────┐
         │ Deploy Staging   │          │Deploy Production │
         │ (Auto-deploy)    │          │(Manual approval) │
         └──────────────────┘          └──────────────────┘
```

---

## 🤝 Contributing

### Adding New Workflows

1. Create the workflow in `.github/workflows/`
2. Use `on: workflow_call:` trigger
3. Document all inputs, outputs, and secrets
4. Add a template in `templates/`
5. Update this README

### Testing Changes

1. Create a test repository
2. Reference your branch: `@feature-branch`
3. Test all input combinations
4. Verify outputs work correctly

### Versioning

We use semantic versioning:
- **Major**: Breaking changes to workflow inputs/outputs
- **Minor**: New features, new workflows
- **Patch**: Bug fixes, documentation updates

---

## 📚 Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Organization Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-an-organization)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

<p align="center">
  <strong>Built with ❤️ by the Arrotech Solutions team</strong>
</p>

