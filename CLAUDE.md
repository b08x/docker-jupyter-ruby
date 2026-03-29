# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo builds two OCI container images that provide JupyterLab with a Ruby (IRuby) kernel and NLP/LLM-focused gems:

- **`base`** — JupyterLab, Python data science stack, spaCy, Google Generative AI SDK (built on `jupyter/docker-stacks-foundation`)
- **`nlp`** — Ruby 3.3.8, IRuby kernel, 100+ gems for NLP, LLM integration, and vector search (built on `base`)

Runtime services via `compose.yaml`: `nlp-notebook`, `redis` (redis-stack), `pgvector` (PostgreSQL + pgvector extension).

## Build Commands

```bash
bundle install          # Install Rake and host-side Ruby deps first

rake build/base         # Pull base upstream image, then build notebook-base
rake build/nlp          # Pull base upstream image, then build notebook-nlp (also builds base first via dep chain)
rake build-all          # Build all images in sequence
rake tag/base           # Tag with git SHA (first 12 chars)
rake push/nlp           # Tag + push both :latest and :<sha> to DockerHub

# Direct container builds (skip Rakefile):
podman build --format docker -f base/Containerfile --rm -t b08x/notebook-base:latest .
podman build --format docker -f nlp/Containerfile  --rm -t b08x/notebook-nlp:latest  .
```

The Rakefile auto-detects the container engine by checking for a running `dockerd` process; falls back to `podman`. Override with `DOCKER_FLAGS` / `PODMAN_FLAGS` env vars. Image ownership comes from `DOCKER_USER` env var (falls back to `$USER`).

## Running

```bash
cp .env.example .env            # Set UID, GID, WORKSPACE, API keys
mkdir -p ./data                 # Or set WORKSPACE in .env

podman-compose up -d            # Start notebook + redis + pgvector
podman-compose logs nlp-notebook | grep token   # Get Jupyter token
podman-compose down
```

Access: Jupyter at `http://localhost:8888`, RedisInsight at `http://localhost:8003`, PostgreSQL at `localhost:5432` (db: `rubynlp`).

## Architecture

### Image Layering

```
jupyter/docker-stacks-foundation
    └── base/Containerfile  (notebook-base)
            └── nlp/Containerfile  (notebook-nlp)
                    ├── Stage 1: rubylang/ruby:3.3.8-jammy (builder)
                    │     Compiles all native gem extensions
                    └── Stage 2: FROM notebook-base
                          Copies compiled Ruby + gems from builder
```

The multi-stage build in `nlp/Containerfile` is critical: native gems (ffi, zmq, lapacke, etc.) are compiled against build-only system libraries in the builder stage, then only the resulting binaries are copied into the runtime image.

### Key Files

| File | Purpose |
|------|---------|
| `nlp/Gemfile` | All Ruby gems for the nlp image; edit here to add/update gems |
| `nlp/Gemfile.lock` | Locked dependency tree — must be committed with changes |
| `nlp/Containerfile` | Two-stage OCI build for the nlp image |
| `base/Containerfile` | Python/Jupyter base image |
| `base/jupyter_server_config.py` | Jupyter config; remote access enabled by default |
| `compose.yaml` | Service definitions; reads from `.env` |
| `Rakefile` | All build/tag/push tasks |
| `gems/` | Pre-built `.gem` files for gems not available on RubyGems (ferret, nmatrix, rbplotly) |
| `postgres/init.sql` | PostgreSQL initialization SQL |

### Gem Path Convention

Inside the nlp container, gems install to `~/.local/share/gem/ruby/<version>/`, symlinked as `~/.local/gem` (set as `GEM_HOME`). The builder stage uses `/root/.local/share/gem/` then chown-copies to the notebook user.

## CI

GitHub Actions (`.github/workflows/ci.yml`):
- Triggers on push to `main` and all PRs (ignores README/docs changes)
- Runs `rake build/base` then `rake build/nlp` sequentially (nlp job `needs: build_and_push_base`)
- Pushes to DockerHub **only on `refs/heads/main`** using `DOCKER_USERNAME`/`DOCKER_TOKEN` secrets and `DOCKER_USER` repo variable
- Tags use the first 12 chars of `GITHUB_SHA`

## Development Conventions

- **Container files are named `Containerfile`** (OCI standard), not `Dockerfile`. The `compose.yaml` `dockerfile:` key still points to `nlp/Containerfile`.
- **Ruby is pinned to 3.3.8** in `nlp/Containerfile` (`FROM rubylang/ruby:3.3.8-jammy`). Update both the Containerfile and `.ruby-version` together.
- **Always commit `Gemfile.lock`** after gem changes. The builder stage runs `bundle lock --add-platform x86_64-linux` to ensure cross-platform lock entries.
- **Patches** applied during the build (e.g., `respond_to_missing.patch` for ruby-spacy/pycall compatibility) live in `nlp/` and are applied in `nlp/Containerfile`.
- **No test harness is wired to CI.** `minitest` is in the Gemfile but tests aren't run in CI. Adding test infrastructure requires updating the CI workflow.
- **`POSTGRES_HOST_AUTH_METHOD`** is `scram-sha-256` in `compose.yaml` (was `trust` previously — do not revert).
- The `make/<image>` Rake tasks (vs `build/<image>`) pass `--build-arg REGISTRY=localhost` for local registry builds without pulling from DockerHub.
