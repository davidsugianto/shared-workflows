# shared-workflows
Centralized place for reusable workflows.

## Available Workflows

### 1. Go Docker CI (`.github/workflows/go-docker-ci.yaml`)
A reusable workflow that runs Go tests, builds a Docker image using Buildx, and pushes it to Docker Hub. It employs a shift-left validation strategy where the Docker build is gated by the success of the Go tests.

#### Inputs
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `go-version` | string | No | `"1.22"` | The Go version to use for building and testing |
| `image-name` | string | **Yes** | N/A | The full Docker Hub image name (e.g., `username/app-name`) |
| `run-tests` | boolean | No | `true` | Whether to run `go test` before building |

#### Outputs
| Name | Description |
|------|-------------|
| `image-tag` | The tag generated and used for the pushed Docker image |

#### Secrets
| Name | Required | Description |
|------|----------|-------------|
| `DOCKERHUB_USERNAME` | **Yes** | Docker Hub username |
| `DOCKERHUB_TOKEN` | **Yes** | Docker Hub access token |

---

### 2. Kustomize Image Tag Update (`.github/workflows/kustomize-image-tag.yaml`)
A reusable workflow that checks out your code, uses Kustomize to update the image tag in a specific environment's `kustomization.yaml`, and commits/pushes the changes back to the repository.

#### Inputs
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `gitops-repo` | string | No | The target GitOps repository (e.g., `owner/repo`). If not provided, it runs against the current repository. |
| `image` | string | **Yes** | Image name to update in Kustomize |
| `tag` | string | **Yes** | New tag to use for the image (usually provided by the CI job) |
| `kustomize-path` | string | **Yes** | Path to the directory containing the `kustomization.yaml` file |
| `create-pr` | boolean | No | Whether to create a Pull Request instead of pushing directly to the target branch. |

#### Secrets
| Name | Required | Description |
|------|----------|-------------|
| `git-token` | **Yes** | GitHub token with write access to the repo. **Highly recommended to use a Personal Access Token (PAT)** rather than the default `GITHUB_TOKEN` so the commit can trigger downstream actions (like deployment workflows) and access external GitOps repositories. |

---

## Example Usage

See the full end-to-end example in [`examples/go-ci.yaml`](examples/go-ci.yaml).

```yaml
# Example CI pipeline in application repository
# .github/workflows/ci.yaml

name: CI/CD Pipeline

on:
  push:
    branches:
      - master
      - main
      - 'release/**'
    tags:
      - 'v*.*.*'
  pull_request:
    branches:
      - master
      - main

env:
  REGISTRY: docker.io
  IMAGE_NAME: ${{ github.repository }}
  GITOPS_REPO: davidsugianto/idp-gitops-manifests
  SERVICE_NAME: go-http-server

jobs:
  # Build and push image
  build-and-push:
    uses: davidsugianto/shared-workflows/.github/workflows/go-docker-ci.yaml@master
    with:
      go-version: "1.25"
      image-name: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
      run-tests: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # Deploy to DEV
  deploy-dev:
    needs: build-and-push
    if: github.ref == 'refs/heads/master' || github.ref == 'refs/heads/main'
    uses: davidsugianto/shared-workflows/.github/workflows/kustomize-image-tag.yaml@master
    with:
      gitops-repo: ${{ env.GITOPS_REPO }}
      image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
      tag: dev-latest
      kustomize-path: environments/dev/${{ env.SERVICE_NAME }}
    secrets:
      git-token: ${{ secrets.GITOPS_PAT }}

  # Deploy to STAGING
  deploy-staging:
    needs: build-and-push
    if: startsWith(github.ref, 'refs/heads/release/')
    uses: davidsugianto/shared-workflows/.github/workflows/kustomize-image-tag.yaml@master
    with:
      gitops-repo: ${{ env.GITOPS_REPO }}
      image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
      tag: staging-latest
      kustomize-path: environments/staging/${{ env.SERVICE_NAME }}
      create-pr: true  # Create PR for staging
    secrets:
      git-token: ${{ secrets.GITOPS_PAT }}

  # Deploy to PRODUCTION
  deploy-prod:
    needs: build-and-push
    if: startsWith(github.ref, 'refs/tags/v')
    uses: davidsugianto/shared-workflows/.github/workflows/kustomize-image-tag.yaml@master
    with:
      gitops-repo: ${{ env.GITOPS_REPO }}
      image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
      tag: ${{ github.ref_name }}
      kustomize-path: environments/prod/${{ env.SERVICE_NAME }}
      create-pr: true  # Create PR for production
    secrets:
      git-token: ${{ secrets.GITOPS_PAT }}
