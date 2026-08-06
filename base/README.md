# Base Image — Python Tool & Inference Layer

JupyterLab foundation image with Python AI/NLP libraries, built on `quay.io/jupyter/base-notebook`.

This image sits at the bottom of the stack — the `nlp` image (Ruby + IRuby kernel, where the agent loop and CLI actually live) builds directly on top of it. Treat this layer as the Python-side tool implementations and local-inference backend a Ruby agent reaches into via `pycall`: spaCy for NLP tool bodies, `llama-cpp-python`/CUDA PyTorch for local model tools, `google-adk`/`google-genai` for Gemini-backed agents, `crewai` when a tool itself needs to be a small Python agent.

## Features

- **spaCy** — Production-grade NLP with `en_core_web_sm` and `en_core_web_lg` models pre-downloaded; the backend for an entity-extraction or dependency-parsing tool called from Ruby
- **Google AI Stack** — `google-genai` and `google-adk` for Gemini API access and agent development, usable directly or as a tool a Ruby agent shells/pycalls into
- **Local LLM Support** — `llama-cpp-python` installed with CPU or CUDA wheels depending on build arg, for tools that need offline/local inference instead of an API call
- **LLM Integrations** — `openai`, `ollama`, `crewai` for multi-provider LLM workflows and Python-side sub-agents
- **Vector Search** — `chromadb` for in-process embedding storage and similarity queries — a lighter-weight alternative to pgvector for a tool that doesn't need a separate service
- **CUDA/CPU Toggle** — Single `CUDA_SUPPORT` build arg switches all PyTorch-backed installs between CPU and CUDA 12.1 wheels
- **Notebook Export** — `pandoc` and multiple inline figure formats (`png`, `svg`, `pdf`, `jpeg`) for full `nbconvert` support, useful for exporting a working prototype notebook as documentation before extraction to a CLI

## Installation

### Podman (recommended)

```bash
podman build \
  --format docker \
  -f base/Containerfile \
  --rm \
  -t b08x/notebook-base:latest \
  .
```

### Docker

```bash
docker build \
  -f base/Containerfile \
  --rm \
  -t b08x/notebook-base:latest \
  .
```

### Via Rake (from repo root)

```bash
bundle install
rake build/base
```

### CUDA build

Pass `--build-arg CUDA_SUPPORT=true` to enable CUDA 12.1 wheels for PyTorch and `llama-cpp-python`:

```bash
podman build \
  --format docker \
  --build-arg CUDA_SUPPORT=true \
  -f base/Containerfile \
  --rm \
  -t b08x/notebook-base:cuda \
  .
```

## Usage

This image is primarily consumed as a base for the `nlp` image, where the actual agent/CLI code is written. Running it directly gives a plain JupyterLab session without the Ruby kernel — useful for prototyping a Python-only tool in isolation before wiring it up as something a Ruby agent calls:

```bash
podman run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user "$(id -u):$(id -g)" \
  b08x/notebook-base:latest
```

Access JupyterLab at `http://localhost:8888`. The token appears in the container logs.

### Build Arguments

- `--build-arg BASE_IMAGE_TAG=<tag>`: Upstream `quay.io/jupyter/base-notebook` tag to build from (default: `latest`)
- `--build-arg CUDA_SUPPORT=true|false`: Selects CUDA 12.1 vs CPU PyTorch index for all pip installs (default: `false`)

### Environment Variables

- `GEN_CERT`: Set to any value to generate a self-signed TLS certificate at startup; enables HTTPS for the Jupyter server
- `NB_UMASK`: Override the default umask for all server subprocesses (octal string, e.g., `022`)

## Configuration

Jupyter server config lives at `base/jupyter_server_config.py` and is copied to `/etc/jupyter/` during the build.

Notable defaults:

- Listens on all interfaces (`c.ServerApp.ip = ""`), bound to port 8888
- Browser auto-open disabled
- Remote access disabled by default (`c.ServerApp.allow_remote_access = False`) — enable explicitly if exposing beyond localhost
- Inline figure formats: `png`, `jpeg`, `svg`, `pdf`
- File deletion goes to filesystem directly, not trash (`delete_to_trash = False`)

## Included Python Libraries

| Library | Purpose | Agent-CLI role |
| --- | --- | --- |
| `spacy` + models | NLP pipeline, NER, tokenization (`en_core_web_sm`, `en_core_web_lg`) | Backend for entity/parsing tools called from Ruby via `pycall` |
| `google-genai` | Gemini API client | Provider backend for a Gemini-based agent loop |
| `google-adk` | Google Agent Development Kit | Reference/alternative agent framework, or a Python sub-agent tool |
| `openai` | OpenAI API client | Provider backend, or Python-side comparison against `ruby_llm` |
| `ollama` | Local Ollama model client | Local-inference tool backend |
| `crewai` | Multi-agent orchestration framework | Python-side multi-agent tool, comparable to `claude_swarm` on the Ruby side |
| `chromadb` | Embedded vector database | Lightweight in-process alternative to the `pgvector` service for agent memory |
| `llama-cpp-python` | Local GGUF model inference | Offline inference tool, no external API dependency |
| `nltk` (punkt) | Tokenization utilities | Text-preprocessing step for ingestion tools |
| `imagehash` | Perceptual image hashing | Dedup/identity tool for multimodal agent inputs |

## Upstream Reference

- [Jupyter Docker Stacks documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/)
- [quay.io/jupyter/base-notebook](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-base-notebook)
