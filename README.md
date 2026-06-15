# shared-workflows
Centralized place for reusable workflows.

## Available Workflows

### Go Docker CI (`.github/workflows/go-docker-ci.yml`)
A reusable workflow that runs Go tests, builds a Docker image using Buildx, and pushes it to Docker Hub. It employs a shift-left validation strategy where the Docker build is gated by the success of the Go tests.

#### Inputs
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `go-version` | string | No | `"1.22"` | The Go version to use for building and testing |
| `image-name` | string | **Yes** | N/A | The full Docker Hub image name (e.g., `username/app-name`) |
| `run-tests` | boolean | No | `true` | Whether to run `go test` before building |

#### Secrets
| Name | Required | Description |
|------|----------|-------------|
| `DOCKERHUB_USERNAME` | **Yes** | Docker Hub username |
| `DOCKERHUB_TOKEN` | **Yes** | Docker Hub access token |

#### Example Usage

See the full example in [`examples/go-ci.yml`](examples/go-ci.yml).

```yaml
name: Example Go Docker CI

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-push:
    # Replace 'owner/repo' with the repository where this shared workflow is hosted
    uses: owner/repo/.github/workflows/go-docker-ci.yml@main
    with:
      go-version: "1.22"
      image-name: "my-dockerhub-username/my-go-app"
      run-tests: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```
