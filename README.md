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
| `image` | string | **Yes** | Image name to update in Kustomize |
| `tag` | string | **Yes** | New tag to use for the image (usually provided by the CI job) |
| `kustomize-path` | string | **Yes** | Path to the directory containing the `kustomization.yaml` file |

#### Secrets
| Name | Required | Description |
|------|----------|-------------|
| `GIT_TOKEN` | **Yes** | GitHub token with write access to the repo. **Highly recommended to use a Personal Access Token (PAT)** rather than the default `GITHUB_TOKEN` so the commit can trigger downstream actions (like deployment workflows). |

---

## Example Usage

See the full end-to-end example in [`examples/go-ci.yaml`](examples/go-ci.yaml).

```yaml
name: Example Go Docker CI

on:
  push:
    branches: [ "master", "main" ]

  pull_request:
    branches: [ "master", "main" ]

jobs:
  build-and-push:
    # Replace 'owner/repo' with the repository where this shared workflow is hosted
    uses: owner/repo/.github/workflows/go-docker-ci.yaml@master
    with:
      go-version: "1.25"
      image-name: "docker.io/my-dockerhub-username/my-go-app"
      run-tests: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  update-image-tag:
    needs: build-and-push
    uses: owner/repo/.github/workflows/kustomize-image-tag.yaml@master
    with:
      image: "docker.io/my-dockerhub-username/my-go-app"
      tag: ${{ needs.build-and-push.outputs.image-tag }}
      # Point to the directory containing kustomization.yaml, not the file itself
      kustomize-path: "my-go-app/deployment/kubernetes/overlays/production"
    secrets:
      # Use a PAT to allow the resulting commit to trigger other workflows
      GIT_TOKEN: ${{ secrets.PAT_TOKEN }}
```
