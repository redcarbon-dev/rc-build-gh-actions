# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Composite GitHub Action for building and publishing Docker images to Google Cloud Container Registry.

Signing and SBOM generation were removed in v0.19 (OPE-1464): Sigstore was retired from the platform, and no caller in the organisation had them enabled.

## Architecture

Single-file action (`action.yml`) that chains together:
2. Docker metadata generation for tagging
3. Google Cloud authentication
4. Docker registry login
5. Docker build and push

## Development

- No build process - edit `action.yml` directly
- No tests exist in this repository

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
