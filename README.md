# Reusable Workflows

This repository contains reusable GitHub Actions workflows for building and publishing Docker container images.

## Available Workflows

### build-container.yml

Builds a Docker image but never pushes it to a registry. Use this to validate that the Dockerfile compiles correctly on every push or pull request.

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `registry` | `ghcr.io` | Container registry URL |
| `image-name` | repository name | Image name, e.g. `my-org/my-app` |
| `context` | `.` | Docker build context path |
| `dockerfile` | `Dockerfile` | Path to the Dockerfile |
| `image-authors` | _(empty)_ | Value for the `org.opencontainers.image.authors` label |

---

### build-and-push-container.yml

Builds and **always pushes** the image. Uses the branch name as the Docker tag. The `:latest` tag is only updated when pushing to the configured default branch.

The published image is always signed with [cosign](https://github.com/sigstore/cosign).

> **Note:** If enabling `sign: true`, the caller must also grant `id-token: write` permission and requires a GitHub plan that supports OIDC (not available on free orgs).

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `registry` | `ghcr.io` | Container registry URL |
| `image-name` | repository name | Image name, e.g. `my-org/my-app` |
| `context` | `.` | Docker build context path |
| `dockerfile` | `Dockerfile` | Path to the Dockerfile |
| `default-branch` | `main` | Branch on which the `:latest` tag is pushed |
| `image-authors` | _(empty)_ | Value for the `org.opencontainers.image.authors` label |
| `sign` | `false` | Sign the published image with cosign (requires OIDC — not available on free orgs) |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `username` | Yes | Registry username |
| `password` | Yes | Registry password or token |

---

### build-and-push-tagged-container.yml

Builds the image on every push. Only pushes to the registry when the commit carries a Git tag. On tagged commits the image receives both the tag name and `:latest`.

The published image is always signed with [cosign](https://github.com/sigstore/cosign).

> **Note:** If enabling `sign: true`, the caller must also grant `id-token: write` permission and requires a GitHub plan that supports OIDC (not available on free orgs).

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `registry` | `ghcr.io` | Container registry URL |
| `image-name` | repository name | Image name, e.g. `my-org/my-app` |
| `context` | `.` | Docker build context path |
| `dockerfile` | `Dockerfile` | Path to the Dockerfile |
| `image-authors` | _(empty)_ | Value for the `org.opencontainers.image.authors` label |
| `sign` | `false` | Sign the published image with cosign (requires OIDC — not available on free orgs) |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `username` | Yes | Registry username |
| `password` | Yes | Registry password or token |

---

## Examples

| File | Demonstrates |
|------|-------------|
| `build-container-example.yml` | Build-only on every push/PR |
| `build-and-push-container-example.yml` | Always push, branch tag + `:latest` on main |
| `build-and-push-tagged-container-example.yml` | Push only on tagged commits |
