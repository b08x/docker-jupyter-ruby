# PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-27
**Commit:** 7d0a347
**Branch:** release/1.0.0

## OVERVIEW
Docker/Podman images providing JupyterLab with Ruby kernel (IRuby) and NLP gems. Two-tier build: `base` (Python/AI) → `nlp` (Ruby + NLP gems).

## STRUCTURE
```
./
├── base/           # Base image: Jupyter + Python/AI stack
├── nlp/           # NLP image: Ruby 3.3.8 + 100+ gems
├── compose.yaml   # Services: notebook, redis, pgvector
├── Rakefile       # Build tasks (podman by default)
├── Gemfile        # Ruby build deps
└── .github/       # CI workflow + copilot-instructions.md
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Build base image | `base/Containerfile` | Python, Jupyter, spaCy |
| Build NLP image | `nlp/Containerfile` | Ruby 3.3.8, gems |
| Add Ruby gems | `nlp/Gemfile` | Then rebuild with `rake build/nlp` |
| Add Python deps | `base/Containerfile` | pip/mamba install |
| Run services | `compose.yaml` | notebook, redis, pgvector |
| CI pipeline | `.github/workflows/ci.yml` | Podman builds |
| Jupyter config | `base/jupyter_server_config.py` | Remote access enabled |

## CONVENTIONS
- **Runtime**: Podman (OCI `Containerfile`); Docker compatible
- **Build tool**: Rake (not Make) - `rake build/nlp`, `rake build/base`
- **Ruby**: Pinned to 3.3.8 with `Gemfile.lock` for reproducibility
- **Compose**: Uses `compose.yaml` (not docker-compose.yml)
- **Entry**: Python scripts in `base/` (`.sh` files deprecated with warnings)

## ANTI-PATTERNS
- Don't use `Dockerfile` - use `Containerfile`
- Don't use `docker-compose.yml` - use `compose.yaml`
- Don't use `docker` commands by default - use `podman`
- Don't update Ruby major version without updating `Gemfile.lock`
- Don't skip `Gemfile.lock` - it's required for reproducible builds

## COMMANDS
```bash
# Install deps
bundle install

# Build images
rake build/nlp      # Builds both base + nlp
rake build/base     # Base only
rake build-all

# Run dev stack
cp .env.example .env
mkdir -p ./data
podman-compose up -d

# Get Jupyter token
podman-compose logs nlp-notebook | grep token

# Rebuild after gem changes
rake build/nlp
```

## NOTES
- Jupyter remote access enabled by default (`base/jupyter_server_config.py`)
- `nlp/respond_to_missing.patch` fixes ruby-spacy/pycall compat
- CI uses Podman; `--format docker` flag for Docker compatibility
- Services: notebook (8888), redis (6379), pgvector (5432)
