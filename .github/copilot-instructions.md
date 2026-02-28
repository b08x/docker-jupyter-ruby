# Copilot / AI Agent Instructions for jupyter-ruby-docker ✅

Purpose: Give AI agents the minimal, actionable knowledge to be immediately productive in this repo. Keep edits safe, explain the "why", and prefer small, reversible changes.

## Quick project summary
- This repo builds Docker/Podman images that provide a JupyterLab server with the IRuby (Ruby) kernel and an NLP-focused gem collection (`nlp` image built on `base`).
- Primary container runtime: **Podman** (Docker is compatible; many commands can be run with `docker` instead).
- Key services (compose): **nlp-notebook**, **redis**, **pgvector** (see `compose.yaml`).
- CI: GitHub Actions runs `rake build/base` and `rake build/nlp` using Podman (see `.github/workflows/ci.yml`).

## When to involve a human / agent-organizer
- This repo includes a multi-agent orchestration guide in `CLAUDE.md`. Follow that for non-trivial tasks: invoke `agent-organizer` for feature work, refactors across files, or project-wide changes.
- Safe-to-automate tasks (AI agents can act directly): small docs fixes, tiny refactors limited to a single file, or example notebook updates that do not change build or deployment logic.

## Important files & places to look (short form)
- `README.md` — Usage, build & run workflows (good starting point)
- `Rakefile` — canonical build tasks: `rake build/nlp`, `rake build/base`, `rake build-all`
- `compose.yaml` — service names, ports, volumes, env vars
- `base/Containerfile`, `nlp/Containerfile` — where to add system & Python / Ruby build steps
- `nlp/Gemfile` and `Gemfile.lock` — add/update Ruby gems here; rebuild the `nlp` image after changes
- `nlp/respond_to_missing.patch` — example of patches applied during build (ruby-spacy/pycall compatibility)
- `base/jupyter_server_config.py` and scripts under `base/` — Jupyter configuration and entrypoints

## Common developer workflows (exact commands)
- Build NLP image (recommended):
  - Install dependencies: `bundle install`
  - Build: `rake build/nlp`
  - OR manually: `podman build -t b08x/notebook-nlp:latest -f nlp/Containerfile .`
- Build base image:
  - `rake build/base` or `podman build -t b08x/notebook-base:latest -f base/Containerfile .`
- Run stack for development (recommended, uses `podman-compose`):
  - `cp .env.example .env` and edit `UID`, `GID`, `WORKSPACE` if needed
  - Ensure workspace dir exists: `mkdir -p ${WORKSPACE:-./data}`
  - Start: `podman-compose up -d` (or `docker compose up -d`)
  - Logs & token: `podman-compose logs nlp-notebook` or `podman logs notebook-nlp` to get Jupyter token
  - Stop: `podman-compose down`
- Rebuild after changing gems or Containerfiles:
  - Edit `nlp/Gemfile` (or `base/Containerfile`), then `rake build/nlp`

## Patterns & conventions agents should follow
- Prefer small, incremental changes and one-file PRs where possible; large multi-component refactors should be planned via `agent-organizer` (see `CLAUDE.md`).
- **Immutable-build mentality**: Reproducible images rely on `Gemfile.lock` and pinned Ruby version (Ruby 3.3.8 in `nlp` image). If you update Ruby or major gems, update `Gemfile.lock` and include a CI-compatible change set.
- Containerfiles use the OCI name `Containerfile` (not `Dockerfile`) — use Podman by default and document any Docker-specific differences you make.
- Patches that modify gem behavior (e.g., `respond_to_missing.patch`) are applied in `nlp/Containerfile`; reference them when debugging compatibility issues with `pycall`/`ruby-spacy`.

## Debugging & tests
- There is no centralized test harness in the repo; the image includes `minitest` and dev utilities, but tests are not wired to CI by default (so check `Gemfile`/`nlp/` when adding test infra).
- Common debugging flows:
  - Inspect build logs: `podman build -f nlp/Containerfile .` or see GitHub Actions logs
  - Inspect running service logs: `podman logs notebook-nlp`, `podman logs redis`, `podman logs pgvector`
  - Use `podman-compose ps` and `podman-compose logs <service>` to inspect health

## Security & environment notes
- Jupyter is configured with remote access enabled by default (`base/jupyter_server_config.py`). Be careful exposing this publicly; use `.env` to set API keys and credentials, and never commit secrets.
- The compose stack uses `POSTGRES_HOST_AUTH_METHOD=trust` for local convenience. Do not use that in production.

## Example PR checklist (for agents to include in PR description)
- Build passes locally: `rake build/nlp` (or `rake build/base` + `rake build/nlp`)
- If changing dependency lists: `Gemfile.lock` updated and rationale in the PR
- If changing Containerfiles: brief note about the reason (security, size, compat)
- If adding a runtime service or port: update `compose.yaml` and `README.md` with instructions

---

If any of the above is unclear or you want more examples (e.g., a small real change worked end-to-end), tell me which area to expand and I will iterate. 👋
