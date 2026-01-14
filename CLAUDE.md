# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Composite GitHub Action for building, signing, and publishing Docker images to Google Cloud Container Registry with supply chain security features (SBOM generation and Cosign signing).

## Architecture

Single-file action (`action.yml`) that chains together:
1. Cosign installation for image signing
2. Docker metadata generation for tagging
3. Google Cloud authentication
4. Docker registry login
5. Docker build and push
6. SBOM generation (Anchore, SPDX format)
7. Image signing and attestation (Cosign with GCP KMS)

## Development

- No build process - edit `action.yml` directly
- No tests exist in this repository
- Image signing uses Google Cloud KMS key at `gcpkms://projects/redcarbon-bb8b2/locations/global/keyRings/sbom-slsa-keyring/cryptoKeys/sbom-slsa-key-feaf3d5`

## Action Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `docker-registry` | Yes | Container registry URL |
| `docker-base-image-path` | Yes | Base path for images |
| `docker-target` | Yes | Docker build target stage |
| `docker-context` | Yes (default: ".") | Build context path |
| `google-credentials` | Yes | GCP service account JSON |
| `branch` | Yes | Git branch name |
| `ts` | Yes | Build timestamp |
| `sha` | Yes | Git commit SHA |
| `push` | No (default: false) | Whether to push to registry |
| `build-args` | No | Docker build arguments |
| `file` | No | Path to Dockerfile |

## Action Output

- `digest`: Docker image digest from the build
