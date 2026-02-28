# Base Image Configuration

**Parent:** `./AGENTS.md`

## OVERVIEW
Jupyter base image with Python/AI stack. Builds on `quay.io/jupyter/base-notebook`.

## FILES
| File | Purpose |
|------|---------|
| `Containerfile` | Base image definition (86 lines) |
| `jupyter_server_config.py` | Jupyter config (remote access enabled) |
| `start-notebook.py` | Entry point script |
| `start-singleuser.py` | JupyterHub singleuser entry |
| `docker_healthcheck.py` | Health check |

## KEY CONFIG
- Base: `quay.io/jupyter/base-notebook`
- System deps: `build-essential`, `gcc`, `fonts-liberation`, `libyaml-dev`, `pandoc`
- Python deps: Added via pip in Containerfile (spaCy, google-generativeai, openai, chromadb, nltk, pytorch)
- Jupyter: Remote access enabled by default

## ADDING DEPENDENCIES
```bash
# Add Python packages - edit base/Containerfile
RUN pip install --no-cache-dir <package>

# Rebuild
rake build/base
```

## NOTES
- `.sh` entry scripts deprecated (emit warnings, delegate to `.py`)
- Health check at port 8888
- UID/GID configurable via build args
