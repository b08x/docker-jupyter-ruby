# NLP Jupyter Image — Ruby 3.3.8

Ruby 3.3.8 with an IRuby kernel, Python bridge, and a broad collection of gems for NLP, LLM orchestration, DSPy, and terminal UI development.

Built on `notebook-base`; requires that image to exist locally or be pulled from the registry before building.

## Features

- **IRuby Kernel** — Run Ruby interactively in JupyterLab notebooks; the kernel registers automatically during build
- **Python Bridge** — `pycall` + `ruby-spacy` expose Python's spaCy NLP pipeline directly from Ruby, patched for newer pycall compatibility
- **DSPy Ecosystem** — Full `dspy` suite with provider extensions for OpenAI, Gemini, and Langfuse observability; structured LLM programming without prompt engineering by hand
- **Ruby LLM Stack** — `ruby_llm` with MCP and schema extensions; `langchainrb` for provider-agnostic LLM workflows; `rllama` for local GGUF inference
- **Charm/Bubble TUI Suite** — `bubbletea`, `bubbles`, `glamour`, `lipgloss`, `gum`, `harmonica`, `ntcharts` for building terminal UIs inside or alongside notebooks
- **Async Runtime** — `async`, `falcon`, `concurrent-ruby`, `circuit_breaker`, and `jongleur` for non-blocking I/O and workflow orchestration
- **Vector & Graph DBs** — `pgvector` (PostgreSQL), `chroma-db`, `redis`/`ohm` — all three persistence backends available without additional setup when running via Compose
- **Document Processing** — Subtitle/SRT parsing, WebVTT, PDF extraction via `kreuzberg`, Markdown via `kramdown`/`commonmarker`/`glamour`

## Installation

<details>
<summary>Via Rake (recommended — builds base first)</summary>

```bash
# From the repo root
bundle install
rake build/nlp
```

</details>

<details>
<summary>Podman (direct)</summary>

```bash
podman build \
  --format docker \
  -f nlp/Containerfile \
  --rm \
  -t b08x/notebook-nlp:latest \
  .
```

</details>

<details>
<summary>Docker</summary>

```bash
docker build \
  -f nlp/Containerfile \
  --rm \
  -t b08x/notebook-nlp:latest \
  .
```

</details>

> **Note:** The `nlp` image pulls `notebook-base` as its base. If building without network access or against a local registry, pass `--build-arg REGISTRY=localhost --build-arg OWNER=<user>`.

## Usage

Run standalone (notebook only):

```bash
podman run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user "$(id -u):$(id -g)" \
  b08x/notebook-nlp:latest
```

Run with the full service stack (notebook + Redis + pgvector):

```bash
cp .env.example .env   # set UID, GID, WORKSPACE, API keys
mkdir -p ./data
podman-compose up -d
```

Get the Jupyter token:

```bash
podman-compose logs nlp-notebook | grep token
```

### Build Arguments

- `--build-arg REGISTRY=<host>`: Registry prefix for the base image pull (default: `docker.io`)
- `--build-arg OWNER=<user>`: Image namespace for the base image (default: `b08x`)
- `--build-arg BASE_IMAGE_TAG=<tag>`: Tag of `notebook-base` to build from (default: `latest`)

## Gem Reference

### Kernel & Python Bridge

| Gem | Purpose |
|-----|---------|
| `iruby` | Jupyter kernel for Ruby |
| `pycall` | Call Python from Ruby; powers ruby-spacy |
| `ruby-spacy` | Ruby wrapper for spaCy NLP (patched — see below) |
| `numpy` | NumPy bindings via pycall |

### DSPy Ecosystem

| Gem | Purpose |
|-----|---------|
| `dspy` | Structured LLM programming framework |
| `dspy-openai` / `dspy-gemini` | Provider adapters |
| `dspy-deep_research` / `dspy-deep_search` | Agentic research modules |
| `dspy-code_act` | Code-generating agent |
| `dspy-evals` | Evaluation utilities |
| `dspy-schema` | Schema validation for DSPy pipelines |
| `dspy-o11y-langfuse` | Langfuse observability integration |
| `dspy-gepa` | GEPA optimization |
| `dspy-ruby_llm` | ruby_llm adapter |

### LLM Integration

| Gem | Purpose |
|-----|---------|
| `ruby_llm` | Provider-agnostic LLM client |
| `ruby_llm-mcp` | Model Context Protocol support |
| `ruby_llm-schema` | Schema generation for tool calls |
| `langchainrb` | LangChain framework for Ruby |
| `groq` | Groq API client |
| `ollama-ai` | Ollama local model client |
| `rllama` | Ruby bindings for llama.cpp (GGUF models) |
| `open_router` | OpenRouter API client |
| `hugging-face` | Hugging Face Hub client |
| `informers` | Transformer inference in Ruby |
| `claude_swarm` | Claude multi-agent orchestration |
| `deepsearch-rb` | Automated web exploration |
| `google-cloud-ai_platform-v1` | Google Cloud AI Platform client |
| `google_custom_search_api` | Google Custom Search |
| `google_search_results` | SerpApi results |

### NLP & Text Processing

| Gem | Purpose |
|-----|---------|
| `tokenizers` | Hugging Face tokenizers |
| `pragmatic_segmenter` | Sentence boundary detection |
| `pragmatic_tokenizer` | Text tokenization |
| `lingua` | Language detection |
| `wordnet` / `wordnet-defaultdb` | WordNet lexical database |
| `tf-idf-similarity` | TF-IDF and cosine similarity |
| `bm25f` | BM25F ranking algorithm |
| `fuzzy_tools` | Fuzzy string matching |
| `linguistics` | Linguistic analysis framework |
| `linkparser` | Link-grammar parser |
| `kramdown` | Markdown parser |
| `commonmarker` | CommonMark parser |
| `loofah` | HTML sanitization |
| `treetop` | PEG parser generator |

### Data & Persistence

| Gem | Purpose |
|-----|---------|
| `daru` / `daru-view` | DataFrame and interactive plotting |
| `sequel` / `sequel_pg` | SQL toolkit + PostgreSQL adapter |
| `pgvector` | pgvector extension for Sequel |
| `ohm` / `ohm-contrib` | Redis object-hash mapping |
| `redis` / `redic` | Redis clients |
| `chroma-db` | Chroma vector database client |
| `oj` / `yajl-ruby` | Fast JSON parsing |
| `jsonl` | JSON Lines support |
| `psych` / `yaml` | YAML parsing |

### Document Processing

| Gem | Purpose |
|-----|---------|
| `kreuzberg` | Document text extraction |
| `front_matter_parser` | YAML/TOML front matter parsing |
| `srt` | SRT subtitle parsing |
| `webvtt` | WebVTT caption parsing |
| `mimemagic` | MIME type detection |
| `redcarpet` | Markdown to HTML rendering |

### Async & Concurrency

| Gem | Purpose |
|-----|---------|
| `async` | Async I/O framework |
| `falcon` | Async HTTP server |
| `concurrent-ruby` | Thread-safe data structures |
| `circuit_breaker` | Circuit breaker pattern |
| `jongleur` | Job orchestration |
| `gush` | Workflow engine |
| `parallel` | Parallel execution |

### Terminal UI (Charm/Bubble Suite)

| Gem | Purpose |
|-----|---------|
| `bubbletea` | Bubble Tea TUI framework |
| `bubbles` | Bubble Tea UI components |
| `glamour` | Markdown rendering in terminal |
| `lipgloss` | Terminal style definitions |
| `gum` | Charm Gum interactive prompts |
| `harmonica` | Animation utilities |
| `ntcharts` | Terminal charts |
| `curses` | Curses TUI bindings |

All standard `tty-*` toolkit gems are also included (`tty-prompt`, `tty-table`, `tty-spinner`, `tty-box`, `tty-font`, `tty-command`, `tty-config`, `tty-editor`, `tty-file`, `tty-link`, `tty-logger`, `tty-markdown`, `tty-option`, `tty-tree`, `tty-screen`, `tty-exit`).

### Web & Utilities

| Gem | Purpose |
|-----|---------|
| `sinatra` | Lightweight HTTP server |
| `graphql` | GraphQL client/server |
| `ratelimit` | Rate limiting |
| `daemons` | Process daemonization |
| `terrapin` | External command execution |
| `dotenv` | `.env` file loading |
| `amazing_print` | Pretty printing |
| `debug_me` | Debug output utilities |
| `pastel` | Terminal color styling |

## Patches

**`respond_to_missing.patch`** — Applied during build to `ruby-spacy`. Fixes a `respond_to_missing?` incompatibility between ruby-spacy and newer pycall versions. Without it, certain spaCy method delegation calls raise `NoMethodError` at runtime.

If updating ruby-spacy or pycall major versions, verify the patch still applies cleanly:

```bash
patch --dry-run -p1 < nlp/respond_to_missing.patch
```

## Customization

- **Add gems** — Edit `nlp/Gemfile`, run `bundle update`, rebuild with `rake build/nlp`. Commit the updated `Gemfile.lock`.
- **Change Ruby version** — Update `FROM rubylang/ruby:<version>-jammy` in `nlp/Containerfile` and `.ruby-version` together. Retest native gem compilation.
- **Pre-built gems** — The `gems/` directory at the repo root contains `.gem` files for gems unavailable on RubyGems (`ferret`, `nmatrix`, `nmatrix-fftw`, `nmatrix-lapacke`, `rbplotly`). Add additional `.gem` files there for gems requiring offline installation.
