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

Outputs `digest` and `tags` so the optional `sign-container.yml` workflow can be chained.

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `registry` | `ghcr.io` | Container registry URL |
| `image-name` | repository name | Image name, e.g. `my-org/my-app` |
| `context` | `.` | Docker build context path |
| `dockerfile` | `Dockerfile` | Path to the Dockerfile |
| `default-branch` | `main` | Branch on which the `:latest` tag is pushed |
| `image-authors` | _(empty)_ | Value for the `org.opencontainers.image.authors` label |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `username` | Yes | Registry username |
| `password` | Yes | Registry password or token |

#### Outputs

| Output | Description |
|--------|-------------|
| `digest` | Image digest (`sha256:...`) |

---

### build-and-push-tagged-container.yml

Builds the image on every push. Only pushes to the registry when the commit carries a Git tag. On tagged commits the image receives both the tag name and `:latest`.

Outputs `digest`, `tags`, and `pushed` so the optional `sign-container.yml` workflow can be chained conditionally.

#### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `registry` | `ghcr.io` | Container registry URL |
| `image-name` | repository name | Image name, e.g. `my-org/my-app` |
| `context` | `.` | Docker build context path |
| `dockerfile` | `Dockerfile` | Path to the Dockerfile |
| `image-authors` | _(empty)_ | Value for the `org.opencontainers.image.authors` label |

#### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `username` | Yes | Registry username |
| `password` | Yes | Registry password or token |

#### Outputs

| Output | Description |
|--------|-------------|
| `digest` | Image digest (`sha256:...`), empty if no push occurred |
| `pushed` | `true` if the image was pushed (i.e. commit was tagged), `false` otherwise |

---

### sign-container.yml

Signs a previously pushed image using [cosign](https://github.com/sigstore/cosign) keyless signing via GitHub OIDC.

> **Requires OIDC:** works on personal repos and paid orgs. Free orgs do not support OIDC token issuance.  
> The caller must grant `id-token: write` and `packages: write`.

Intended to be used as a second job after `build-and-push-container.yml` or `build-and-push-tagged-container.yml`, consuming their outputs.

#### Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `image` | Yes | Full image reference without tag, e.g. `ghcr.io/my-org/my-app` |
| `digest` | Yes | Image digest (`sha256:...`) |

---

## Examples

| File | Demonstrates |
|------|-------------|
| `build-container-example.yml` | Build-only on every push/PR |
| `build-and-push-container-example.yml` | Always push, branch tag + `:latest` on main (optional signing) |
| `build-and-push-tagged-container-example.yml` | Push only on tagged commits (optional signing) |
