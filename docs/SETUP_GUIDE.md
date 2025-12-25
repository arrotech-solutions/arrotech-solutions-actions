# 🔧 Setup Guide

This guide walks you through setting up centralized GitHub Actions for your organization.

## Prerequisites

1. You are an owner or admin of the `arrotech-solutions` organization
2. You have access to create/modify organization secrets
3. Your repositories have GitHub Actions enabled

---

## Step 1: Configure Organization Secrets

Navigate to your organization's settings and add the following secrets:

### Required Secrets

#### For Kubernetes Deployments

1. **Get your kubeconfig:**
   ```bash
   cat ~/.kube/config | base64 | tr -d '\n'
   ```

2. **Add as organization secret:**
   - Name: `KUBE_CONFIG_STAGING`
   - Value: Base64-encoded kubeconfig for staging
   - Access: Selected repositories (or all)

3. Repeat for production:
   - Name: `KUBE_CONFIG_PROD`

#### For AWS Deployments

1. Create an IAM user with appropriate ECS/ECR permissions
2. Add secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_REGION` (optional, can be set as variable)

#### For Vercel Deployments (Frontend Apps)

1. **Get your Vercel token:**
   - Go to [Vercel Account Settings → Tokens](https://vercel.com/account/tokens)
   - Create a new token with "Full Account" scope
   - Copy the token

2. **Add as organization secret:**
   - Name: `VERCEL_TOKEN`
   - Value: Your Vercel API token
   - Access: Selected repositories (frontend apps)

3. **In your frontend repository:**
   - Link your project to Vercel: `npx vercel link`
   - This creates a `.vercel` directory with project configuration

#### For Fly.io Deployments (Backend Apps)

1. **Install Fly.io CLI:**
   ```bash
   # macOS
   brew install flyctl

   # Linux
   curl -L https://fly.io/install.sh | sh

   # Windows
   powershell -Command "iwr https://fly.io/install.ps1 -useb | iex"
   ```

2. **Authenticate and get your API token:**
   ```bash
   # Login to Fly.io
   flyctl auth login

   # Create an API token
   flyctl tokens create deploy -x 999999h
   ```

3. **Add as organization secret:**
   - Name: `FLY_API_TOKEN`
   - Value: Your Fly.io API token
   - Access: Selected repositories (backend apps)

4. **In your backend repository, create `fly.toml`:**
   ```toml
   app = "your-app-name"
   primary_region = "lhr"  # London, change as needed

   [build]
     dockerfile = "Dockerfile"

   [http_service]
     internal_port = 3000  # Your app's port
     force_https = true
     auto_stop_machines = true
     auto_start_machines = true
     min_machines_running = 0

   [env]
     NODE_ENV = "production"
   ```

5. **Add environment-specific secrets:**
   - Create `STAGING_ENV_VARS` and `PRODUCTION_ENV_VARS` secrets
   - Format as JSON: `{"DATABASE_URL": "postgres://...", "REDIS_URL": "redis://..."}`

---

## Step 2: Set Up a Repository

### Option A: Frontend App (Vercel)

1. Copy the Vercel template:
   ```bash
   curl -o .github/workflows/deploy.yml \
     https://raw.githubusercontent.com/arrotech-solutions/arrotech-solutions-actions/main/templates/vercel-deploy.yml
   ```

2. Link your project to Vercel:
   ```bash
   npx vercel link
   ```

3. Push to your repository - deployments will run automatically!

### Option B: Backend App (Fly.io)

1. Copy the Fly.io template:
   ```bash
   curl -o .github/workflows/deploy.yml \
     https://raw.githubusercontent.com/arrotech-solutions/arrotech-solutions-actions/main/templates/flyio-deploy.yml
   ```

2. Create your `fly.toml` (see Fly.io setup section above)

3. Create a Dockerfile for your app

4. Push to your repository - deployments will run automatically!

### Option C: Full Pipeline (Custom deployment)

1. Copy the appropriate template:
   ```bash
   # For Node.js projects
   curl -o .github/workflows/pipeline.yml \
     https://raw.githubusercontent.com/arrotech-solutions/arrotech-solutions-actions/main/templates/node-full-pipeline.yml

   # For Python projects  
   curl -o .github/workflows/pipeline.yml \
     https://raw.githubusercontent.com/arrotech-solutions/arrotech-solutions-actions/main/templates/python-full-pipeline.yml
   ```

2. Customize the template:
   - Update `NODE_VERSION` or `PYTHON_VERSION`
   - Configure your deployment platform
   - Set coverage thresholds

3. Push to your repository

### Option D: Minimal CI (For smaller projects)

1. Copy the minimal template:
   ```bash
   curl -o .github/workflows/ci.yml \
     https://raw.githubusercontent.com/arrotech-solutions/arrotech-solutions-actions/main/templates/minimal-ci.yml
   ```

2. Update the `language` input to match your project

---

## Step 3: Configure Environments (for CD)

For deployment protection rules:

1. Go to **Repository Settings → Environments**
2. Create environments:
   - `staging` - Auto-deploy from `develop` branch
   - `production` - Require manual approval

3. Add environment-specific rules:
   - Required reviewers (for production)
   - Deployment branches
   - Wait timer (optional)

---

## Step 4: Branch Protection

Set up branch protection for `main`:

1. Go to **Repository Settings → Branches**
2. Add rule for `main`:
   - ✅ Require status checks to pass
   - ✅ Require branches to be up to date
   - Select required checks:
     - `Code Quality`
     - `Tests`
     - `Security Scan`
   - ✅ Require pull request reviews

---

## Step 5: Test Your Setup

1. Create a test PR:
   ```bash
   git checkout -b test/ci-setup
   echo "# Test" >> TEST.md
   git add TEST.md
   git commit -m "test: verify CI pipeline"
   git push origin test/ci-setup
   ```

2. Open a PR and verify:
   - Code quality checks run
   - Security scans complete
   - Tests execute

3. Merge to `develop` and verify:
   - Build artifacts are created
   - Staging deployment triggers (if configured)

---

## Troubleshooting

### Workflow not found

```
Error: .github#L1
  could not find workflow 'arrotech-solutions/arrotech-solutions-actions/.github/workflows/ci-code-quality.yml@main'
```

**Solution:** Ensure the `arrotech-solutions-actions` repository is public, or the calling repository has access to it.

### Permission denied for secrets

```
Error: Resource not accessible by integration
```

**Solution:** Check that organization secrets are configured for the repository and the workflow has correct permissions.

### Deployment fails with kubeconfig error

```
Error: Unable to connect to the server
```

**Solution:** 
1. Verify the kubeconfig is correctly base64-encoded
2. Ensure the cluster is accessible from GitHub's runners
3. Check that service account has correct RBAC permissions

---

## Next Steps

- [ ] Set up Slack notifications for workflow failures
- [ ] Configure branch protection rules
- [ ] Add repository-specific secrets as needed
- [ ] Set up deployment environments with approval rules

---

## Getting Help

- Review the [README](../README.md) for workflow documentation
- Check [GitHub Actions docs](https://docs.github.com/en/actions)
- Open an issue in this repository for bugs or feature requests

