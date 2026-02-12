# Fork Setup Guide

This document provides instructions for setting up the forked Fukuii repository with CI/CD and GitHub Actions.

## Overview

This repository has been forked from `chippr-robotics/fukuii` to `chris-mercer/fukuii`. The following changes have been made to ensure proper functionality:

### ✅ Completed Configuration Updates

1. **Workflow Files Updated**
   - All workflow files now use dynamic repository references (`${{ github.repository }}`, `${{ github.repository_owner }}`)
   - Docker registry configurations updated to use the forked repository
   - GitHub Pages URL updated dynamically

2. **README Updated**
   - CI/CD badges point to the forked repository
   - Docker Hub and GitHub Container Registry references updated
   - Documentation links updated to use the forked repository owner

3. **Dynamic References**
   - All hardcoded `chippr-robotics` references replaced with dynamic variables
   - All hardcoded `chipprbots` references replaced with dynamic variables

## Required GitHub Repository Settings

### 1. Enable GitHub Actions

1. Go to **Settings** → **Actions** → **General**
2. Under "Actions permissions", select:
   - ✅ "Allow all actions and reusable workflows"
3. Under "Workflow permissions", select:
   - ✅ "Read and write permissions"
   - ✅ "Allow GitHub Actions to create and approve pull requests"

### 2. Enable GitHub Pages

1. Go to **Settings** → **Pages**
2. Under "Source", select:
   - Source: **GitHub Actions**
3. Save the settings

The documentation will be available at: `https://chris-mercer.github.io/fukuii/`

### 3. Configure Branch Protection (Optional but Recommended)

See [BRANCH_PROTECTION.md](BRANCH_PROTECTION.md) for detailed instructions.

Quick setup for the `main` branch:

1. Go to **Settings** → **Branches** → **Add branch protection rule**
2. Branch name pattern: `main`
3. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
     - Status checks: `Test and Build (JDK 21, Scala 3.3.4)`
   - ✅ Require conversation resolution before merging

### 4. Repository Secrets (Optional)

Some workflows require secrets for full functionality. These are optional unless you plan to use specific features:

#### For Docker Hub Publishing (Optional)

If you want to publish Docker images to Docker Hub:

1. Create a Docker Hub account and repository: `https://hub.docker.com/r/chris-mercer/fukuii`
2. Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
3. Add:
   - `DOCKERHUB_USERNAME`: Your Docker Hub username
   - `DOCKERHUB_TOKEN`: Docker Hub access token (create at https://hub.docker.com/settings/security)

**Note:** If these secrets are not configured, Docker Hub publishing will be skipped, but images will still be published to GitHub Container Registry (ghcr.io).

#### For Codecov (Optional)

If you want code coverage reports:

1. Sign up at https://codecov.io with your GitHub account
2. Add the repository to Codecov
3. Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
4. Add:
   - `CODECOV_TOKEN`: Token from Codecov repository settings

#### For Launchpad PPA Publishing (Optional)

Only needed if you want to publish Ubuntu packages:

1. Create Launchpad account and PPA at https://launchpad.net/
2. Generate GPG key for signing (see [workflows/README.md](workflows/README.md#launchpad-ppa-setup))
3. Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
4. Add:
   - `LAUNCHPAD_GPG_PRIVATE_KEY`: GPG private key for signing
   - `LAUNCHPAD_GPG_PASSPHRASE`: GPG key passphrase

## Testing the CI/CD Setup

### 1. Test CI Workflow

The CI workflow runs automatically on every push and pull request. To test:

```bash
# Make a small change and push
git checkout -b test-ci
echo "# Test" >> README.md
git add README.md
git commit -m "test: CI workflow"
git push origin test-ci
```

Then create a pull request and verify:
- ✅ CI workflow runs successfully
- ✅ Tests pass
- ✅ Build artifacts are created

### 2. Test Docker Build

The Docker workflow runs on pushes to main branches. To test:

1. Merge the test PR to `main`
2. Check the **Actions** tab
3. Verify the "Docker Build" workflow completes successfully
4. Check https://github.com/chris-mercer/fukuii/pkgs/container/fukuii for published images

### 3. Test GitHub Pages

The documentation deployment happens automatically when docs are changed. To test:

1. Make a change to any file in the `docs/` directory
2. Push to `main` branch
3. Check the **Actions** tab for "Deploy Documentation to GitHub Pages" workflow
4. Visit https://chris-mercer.github.io/fukuii/ to see the deployed documentation

### 4. Test Release Process (When Ready)

When you're ready to create a release:

```bash
# Update version in version.sbt if needed
git tag -a v0.2.0 -m "Release version 0.2.0"
git push origin v0.2.0
```

This will trigger the release workflow which will:
- ✅ Build distribution packages
- ✅ Generate CHANGELOG
- ✅ Create GitHub release with artifacts
- ✅ Build and sign Docker images
- ✅ Close matching milestone (if exists)

## Workflow Overview

### Core Workflows (Always Active)

1. **CI Workflow** (`ci.yml`)
   - Runs on: Push to main/develop, Pull Requests
   - Purpose: Build, test, and validate code
   - Required: Yes - Should be configured as a required status check

2. **Docker Build** (`docker.yml`)
   - Runs on: Push to main/develop
   - Purpose: Build and publish Docker images
   - Publishes to: GitHub Container Registry (always), Docker Hub (if secrets configured)

3. **GitHub Pages** (`gh-pages.yml`)
   - Runs on: Push to main/develop (docs changes)
   - Purpose: Deploy documentation
   - Publishes to: https://chris-mercer.github.io/fukuii/

### Optional Workflows

4. **Release** (`release.yml`)
   - Runs on: Version tags (v*)
   - Purpose: Create releases with artifacts
   - Note: Only runs when you create a tag

5. **Nightly** (`nightly.yml`)
   - Runs on: Schedule (nightly)
   - Purpose: Create nightly development builds
   - Note: Disabled by default, can be enabled by uncommenting the schedule

6. **Fast Distro** (`fast-distro.yml`)
   - Runs on: Manual trigger, Schedule (optional)
   - Purpose: Quick builds without full test suite
   - Note: For development/testing only

7. **Launchpad PPA** (`launchpad-ppa.yml`)
   - Runs on: Release published
   - Purpose: Publish Ubuntu packages
   - Note: Requires GPG secrets to be configured

## Common Issues and Solutions

### Issue: CI Workflow Fails with "sbt: command not found"

**Solution:** The workflow automatically installs SBT. If this fails, it's usually a temporary issue with the Ubuntu package repository. Re-run the workflow.

### Issue: Docker Build Fails

**Possible causes:**
1. Dockerfile syntax errors (unlikely if forked without changes)
2. Base image build failed (check the logs for the base image job)
3. Insufficient permissions

**Solution:** Check the workflow logs in the Actions tab to identify the specific error.

### Issue: GitHub Pages Not Deploying

**Possible causes:**
1. GitHub Pages not enabled in repository settings
2. Workflow doesn't have write permissions

**Solution:**
1. Verify GitHub Pages is enabled (Settings → Pages → Source: GitHub Actions)
2. Verify workflow permissions (Settings → Actions → General → Workflow permissions: Read and write)

### Issue: Release Workflow Doesn't Trigger

**Possible cause:** Tag format is incorrect

**Solution:** Ensure tags start with 'v' (e.g., `v1.0.0`, not `1.0.0`)

### Issue: Docker Hub Publishing Skipped

**Expected behavior:** If `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets are not configured, Docker Hub publishing is automatically skipped. Images are still published to GitHub Container Registry.

**Solution:** This is not an error. If you want to publish to Docker Hub, configure the secrets as described above.

## Next Steps

1. ✅ Verify all workflows are running successfully
2. ✅ Set up branch protection rules (recommended)
3. ✅ Configure optional secrets if needed
4. ✅ Create milestones for tracking work
5. ✅ Review and update documentation as needed
6. ✅ Start development!

## Additional Resources

- [Workflows Documentation](workflows/README.md) - Detailed workflow documentation
- [Branch Protection](BRANCH_PROTECTION.md) - Branch protection setup guide
- [Quickstart Guide](QUICKSTART.md) - Development quickstart
- [Contributing Guide](../CONTRIBUTING.md) - Contribution guidelines

## Support

For issues with the fork setup:
1. Check the [Actions](https://github.com/chris-mercer/fukuii/actions) tab for workflow logs
2. Review this document for common issues
3. Check workflow documentation in [workflows/README.md](workflows/README.md)
4. Create an issue in the repository if you encounter problems

## Updating from Upstream

If you want to sync changes from the original repository:

```bash
# Add upstream remote (one time only)
git remote add upstream https://github.com/chippr-robotics/fukuii.git

# Fetch upstream changes
git fetch upstream

# Merge upstream changes into your main branch
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

**Note:** Be careful when merging upstream changes, as they may reintroduce hardcoded references that need to be updated.
