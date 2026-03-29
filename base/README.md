# Base Jupyter Notebook Stack

JupyterLab foundation image with Python AI/NLP libraries, built on `quay.io/jupyter/base-notebook`.

This image sits at the bottom of the stack — the `nlp` image (Ruby + IRuby kernel) builds directly on top of it.

## Features

- **spaCy** — Production-grade NLP with `en_core_web_sm` and `en_core_web_lg` models pre-downloaded
- **Google AI Stack** — `google-genai` and `google-adk` for Gemini API access and agent development
- **Local LLM Support** — `llama-cpp-python` installed with CPU or CUDA wheels depending on build arg
- **LLM Integrations** — `openai`, `ollama`, `crewai` for multi-provider LLM workflows
- **Vector Search** — `chromadb` for in-process embedding storage and similarity queries
- **CUDA/CPU Toggle** — Single `CUDA_SUPPORT` build arg switches all PyTorch-backed installs between CPU and CUDA 12.1 wheels
- **Notebook Export** — `pandoc` and multiple inline figure formats (`png`, `svg`, `pdf`, `jpeg`) for full `nbconvert` support

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

This image is primarily consumed as a base for the `nlp` image. Running it directly gives a plain JupyterLab session without the Ruby kernel:

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

| Library | Purpose |
| --- | --- |
| `spacy` + models | NLP pipeline, NER, tokenization (`en_core_web_sm`, `en_core_web_lg`) |
| `google-genai` | Gemini API client |
| `google-adk` | Google Agent Development Kit |
| `openai` | OpenAI API client |
| `ollama` | Local Ollama model client |
| `crewai` | Multi-agent orchestration framework |
| `chromadb` | Embedded vector database |
| `llama-cpp-python` | Local GGUF model inference |
| `nltk` (punkt) | Tokenization utilities |
| `imagehash` | Perceptual image hashing |

## Upstream Reference

- [Jupyter Docker Stacks documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/)
- [quay.io/jupyter/base-notebook](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-base-notebook)
