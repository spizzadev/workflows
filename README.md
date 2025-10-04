# Reusable Workflows

This repository contains reusable GitHub Actions workflows for common CI/CD tasks.

## Available Workflows

### build-and-deploy-container.yml

A flexible workflow for building and deploying Docker containers with the following features:

- **Builds container on every push** - Always builds the Docker image to catch build errors early
- **Automatic deployment on tagged commits** - Automatically deploys when a commit is tagged
- **Manual deployment option** - Allows manual deployment of untagged commits via the `force-deploy` input

#### Features

- **Smart tagging strategy:**
  - Tagged commits: Uses `latest` and the Git tag name
  - Forced deployments on untagged commits: Uses SHA-based tag
  - Build-only mode: No tags pushed to registry

- **Security:** Uses cosign to sign published Docker images
- **Efficiency:** Leverages GitHub Actions cache for faster builds

#### Usage

```yaml
name: Build and Deploy My App

on:
  push:
    branches:
      - main
  workflow_dispatch:
    inputs:
      force-deploy:
        description: 'Force deployment even on untagged commits'
        required: false
        type: boolean
        default: false

jobs:
  build-and-deploy:
    uses: Ruixey/workflows/.github/workflows/build-and-deploy-container.yml@main
    with:
      registry: ghcr.io
      image-name: my-org/my-app
      force-deploy: ${{ github.event.inputs.force-deploy || false }}
    secrets:
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
```

#### Inputs

| Input | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `registry` | Yes | string | - | Container registry URL (e.g., `ghcr.io`, `docker.io`) |
| `image-name` | Yes | string | - | Full image name including organization/user |
| `force-deploy` | No | boolean | `false` | Force deployment even on untagged commits |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `username` | Yes | Registry username |
| `password` | Yes | Registry password or token |

#### Behavior

1. **On every push (untagged commits):**
   - Builds the Docker image
   - Does NOT push to registry (unless `force-deploy: true`)
   - Validates that the build succeeds

2. **On tagged commits:**
   - Builds the Docker image
   - Pushes to registry with tags: `latest` and the Git tag name
   - Signs the image with cosign

3. **Manual deployment (workflow_dispatch with force-deploy: true):**
   - Builds the Docker image
   - Pushes to registry with SHA-based tag
   - Signs the image with cosign

### deploy-tagged-container.yml

The original workflow that only deploys on tagged commits. Builds containers on all commits but only pushes to the registry when a Git tag is present.

### deploy-container.yml

A simpler workflow that builds and deploys on every push (except pull requests).

## Examples

See the `build-and-deploy-example.yml` file for a complete example of using the `build-and-deploy-container.yml` workflow.
