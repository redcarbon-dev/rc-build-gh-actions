# RC Docker Build Action

> **Signing and SBOM generation were removed in v0.19** (OPE-1464). Sigstore was
> retired from the platform in OPE-1461, and a scan of all 86 repositories in the
> organisation found 49 workflows calling this action and **none** passing
> `enable-signing: true` — the path was dead code that still paid a cosign install
> on every build. Restore it from git history if it is ever wanted again.

Composite GitHub Action for building, signing, and publishing Docker images to Google Cloud Container Registry with comprehensive supply chain security features.

## Features

- **Docker Build & Push** - Build and optionally push Docker images to Google Cloud Container Registry
- **Layer Caching** - GitHub Actions cache integration for faster builds
- **Flexible Configuration** - Configurable network modes, build args, and signing options

## Prerequisites

- Google Cloud project with Container Registry enabled
- Service account with Container Registry permissions

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `docker-registry` | Yes | - | Container registry URL (e.g., `gcr.io`) |
| `docker-base-image-path` | Yes | - | Base path for images in registry |
| `docker-target` | Yes | - | Docker build target stage |
| `docker-context` | No | `.` | Build context path |
| `google-credentials` | Yes | - | GCP service account JSON credentials |
| `branch` | Yes | - | Git branch name (used in tagging) |
| `ts` | Yes | - | Build timestamp (used in tagging) |
| `sha` | Yes | - | Git commit SHA (must be 7-40 hex characters) |
| `push` | No | `false` | Whether to push image to registry |
| `build-args` | No | - | Docker build arguments (multiline) |
| `file` | No | - | Path to Dockerfile |
| `docker-network` | No | `host` | Docker build network mode (`host`, `bridge`, `none`) |
| `tag` | No | - | Optional additional tag for the image (e.g. `v1.0.1`) |


## Outputs

| Output | Description |
|--------|-------------|
| `digest` | Docker image digest (SHA256) |
| `image-tags` | Generated image tags (newline-separated) |

## Usage

### Basic Build (Local Only)

Build without pushing to registry:

```yaml
- uses: your-org/rc-build-gh-actions@v1
  with:
    docker-registry: gcr.io
    docker-base-image-path: my-project
    docker-target: production
    google-credentials: ${{ secrets.GCP_CREDENTIALS }}
    branch: ${{ github.ref_name }}
    ts: ${{ github.run_number }}
    sha: ${{ github.sha }}
    push: false
```

### Build and Push

Build and push to Google Container Registry:

```yaml
- uses: your-org/rc-build-gh-actions@v1
  with:
    docker-registry: gcr.io
    docker-base-image-path: my-project
    docker-target: production
    google-credentials: ${{ secrets.GCP_CREDENTIALS }}
    branch: ${{ github.ref_name }}
    ts: ${{ github.run_number }}
    sha: ${{ github.sha }}
    push: true
```

### Advanced Configuration

With custom build arguments, Dockerfile path, and network mode:

```yaml
- uses: your-org/rc-build-gh-actions@v1
  with:
    docker-registry: gcr.io
    docker-base-image-path: my-project
    docker-target: production
    docker-context: ./app
    file: ./app/Dockerfile.prod
    docker-network: bridge
    google-credentials: ${{ secrets.GCP_CREDENTIALS }}
    branch: ${{ github.ref_name }}
    ts: ${{ github.run_number }}
    sha: ${{ github.sha }}
    push: true
    build-args: |
      NODE_VERSION=20
      BUILD_ENV=production
```

### Release Tag

Add an extra release tag (e.g. from a git tag) to the image:

```yaml
- uses: your-org/rc-build-gh-actions@v1
  with:
    docker-registry: gcr.io
    docker-base-image-path: my-project
    docker-target: production
    google-credentials: ${{ secrets.GCP_CREDENTIALS }}
    branch: ${{ github.ref_name }}
    ts: ${{ github.run_number }}
    sha: ${{ github.sha }}
    push: true
    tag: ${{ github.ref_name }}  # e.g. v1.0.1
```

## Image Tagging Strategy

The action generates multiple tags for each image:

- `sha-<git-sha>` - Based on git commit SHA
- `<branch>-<sha>-<timestamp>` - Custom tag with branch, SHA, and timestamp
- `<version>` - Semantic version (if applicable)
- `<major>.<minor>` - Semantic version without patch (if applicable)
- `<tag>` - Custom release tag, if provided via the `tag` input (e.g. `v1.0.1`)

## Docker Layer Caching

The action uses a buildx **registry cache** hosted next to the images:

- **Cache Type**: `type=registry`, stored at `<docker-registry>/<docker-base-image-path>/cache/<repository>/<docker-target>`
- **Cache Mode**: `max` - Caches all intermediate layers
- **Runner-independent**: the cache lives in Artifact Registry, so ephemeral runners get warm builds too
- **Automatic**: No configuration needed, works out of the box

The cache ref is scoped per calling repository so that repos sharing a target
name (e.g. `production`) never overwrite each other's cache. Retention of the
`cache/` path is handled by the Artifact Registry cleanup policies defined in
`rc-infrastructure` (see ADR 0004 there).

Subsequent builds of the same image will reuse unchanged layers, significantly reducing build times.

## Complete Workflow Example

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - name: Build and Push
        id: docker
        uses: your-org/rc-build-gh-actions@v1
        with:
          docker-registry: gcr.io
          docker-base-image-path: my-project
          docker-target: production
          google-credentials: ${{ secrets.GCP_CREDENTIALS }}
          branch: ${{ github.ref_name }}
          ts: ${{ github.run_number }}
          sha: ${{ github.sha }}
          push: ${{ github.event_name != 'pull_request' }}

      - name: Output Image Details
        run: |
          echo "Image Digest: ${{ steps.docker.outputs.digest }}"
          echo "Image Tags: ${{ steps.docker.outputs.image-tags }}"
          echo "Full Image: ${{ steps.docker.outputs.image }}"
```

## Troubleshooting

### Invalid SHA Format Error

Ensure the `sha` input is a valid git commit SHA (7-40 hexadecimal characters):

```yaml
sha: ${{ github.sha }}  # Full SHA from GitHub context
```

### Cache Not Working

1. Ensure `docker/setup-buildx-action` is running (it should be automatic)
2. Check that the service account can push to the `cache/` path of the registry
   (same permission as the image push)
3. Cache retention is managed server-side by Artifact Registry cleanup policies
   - no manual cleanup needed

## License

[Your License Here]

## Contributing

[Your Contributing Guidelines Here]
