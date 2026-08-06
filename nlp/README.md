# NLP Image — Ruby 3.3.8 Agentic CLI Toolkit

Ruby 3.3.8 with an IRuby kernel, Python bridge, and a broad collection of gems for building LLM tool-calling agents, MCP servers, and terminal-native CLI front-ends. This is the image where an agent loop gets written — prototyped in a notebook cell, then extracted verbatim into a `bin/` script.

Built on `notebook-base`; requires that image to exist locally or be pulled from the registry before building.

## Features

- **IRuby Kernel** — Run Ruby interactively in JupyterLab notebooks; the kernel registers automatically during build. Every gem below is available identically inside the kernel or a standalone `ruby script.rb` run
- **LLM Orchestration** — `ruby_llm` (provider-agnostic chat client), `ruby_llm-mcp` (MCP client/server — both directions), `ruby_llm-schema` (tool-call schema generation) for the request/response loop at an agent's core
- **DSPy Ecosystem** — Full `dspy` suite with provider extensions for OpenAI, Gemini, and Langfuse observability; structured LLM programming that optimizes prompts against examples instead of hand-tuning them
- **Multi-Agent & Async** — `claude_swarm` for coordinating multiple Claude agents; `async`, `falcon`, `concurrent-ruby`, `circuit_breaker`, `jongleur`, `gush` for concurrent tool calls and workflow orchestration outside a single blocking request
- **Python Bridge** — `pycall` + `ruby-spacy` call Python's spaCy NLP pipeline directly from Ruby, patched for newer pycall compatibility — a tool implementation that reaches into the `base` image's Python stack without leaving the Ruby process
- **CLI/TUI Suite (Charm/Bubble)** — `bubbletea`, `bubbles`, `glamour`, `lipgloss`, `gum`, `harmonica`, `ntcharts`, plus the full `tty-*` toolkit for building the interactive shell an agent runs inside
- **Agent Memory & Retrieval** — `pgvector` (PostgreSQL), `chroma-db`, `redis`/`ohm` for semantic search and response caching — all three available without extra setup when running via Compose
- **Tool/Ingestion Building Blocks** — Subtitle/SRT parsing, WebVTT, PDF extraction via `kreuzberg`, Markdown via `kramdown`/`commonmarker`/`glamour` for document-ingestion tools an agent can call

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

Run with the full service stack (notebook + Redis + pgvector — the memory/cache backends an agent's tools use):

```bash
cp .env.example .env   # set UID, GID, WORKSPACE, API keys
mkdir -p ./data
podman-compose up -d
```

Get the Jupyter token:

```bash
podman-compose logs nlp-notebook | grep token
```

### Running a standalone agent script

Once a loop is prototyped in the kernel, run the same code as a script against the running Compose stack (pgvector/Redis stay reachable at their service hostnames):

```bash
podman exec -it notebook-nlp ruby /home/jovyan/work/agent.rb
```

### Build Arguments

- `--build-arg REGISTRY=<host>`: Registry prefix for the base image pull (default: `docker.io`)
- `--build-arg OWNER=<user>`: Image namespace for the base image (default: `b08x`)
- `--build-arg BASE_IMAGE_TAG=<tag>`: Tag of `notebook-base` to build from (default: `latest`)

## Gem Reference

Grouped by role in an agent's architecture rather than alphabetically — find the gem for the layer you're building.

### Agent Loop & LLM Orchestration

| Gem | Purpose |
|-----|---------|
| `ruby_llm` | Provider-agnostic LLM client — the core chat/completion loop |
| `ruby_llm-mcp` | Model Context Protocol client and server — consume tools from other agents, or expose your own |
| `ruby_llm-schema` | Schema generation for tool calls |
| `langchainrb` | LangChain framework for Ruby — alternative orchestration layer |
| `groq` | Groq API client |
| `ollama-ai` | Ollama local model client |
| `rllama` | Ruby bindings for llama.cpp (GGUF models) — fully offline agent loop |
| `open_router` | OpenRouter API client |
| `claude_swarm` | Coordinate multiple Claude agents in one process |
| `deepsearch-rb` | Automated web exploration — a ready-made research tool |
| `hugging-face` | Hugging Face Hub client |
| `informers` | Transformer inference in Ruby, no Python bridge required |
| `google-cloud-ai_platform-v1` | Google Cloud AI Platform client |
| `google_custom_search_api` / `google_search_results` | Search-as-a-tool backends |

### DSPy — Structured/Self-Optimizing Prompting

| Gem | Purpose |
|-----|---------|
| `dspy` | Structured LLM programming framework |
| `dspy-openai` / `dspy-gemini` | Provider adapters |
| `dspy-deep_research` / `dspy-deep_search` | Agentic research modules |
| `dspy-code_act` | Code-generating agent |
| `dspy-evals` | Evaluation utilities — score an agent's outputs against a rubric |
| `dspy-schema` | Schema validation for DSPy pipelines |
| `dspy-o11y-langfuse` | Langfuse observability integration — trace an agent's calls |
| `dspy-gepa` | GEPA optimization |
| `dspy-ruby_llm` | `ruby_llm` adapter for DSPy |

### Kernel & Python Bridge (Tool Implementations)

| Gem | Purpose |
|-----|---------|
| `iruby` | Jupyter kernel for Ruby |
| `pycall` | Call Python from Ruby; powers ruby-spacy and any `base`-image Python tool |
| `ruby-spacy` | Ruby wrapper for spaCy NLP (patched — see below) — an NER/parsing tool |
| `numpy` | NumPy bindings via pycall |

### CLI/TUI Front-Ends (Charm/Bubble Suite)

| Gem | Purpose |
|-----|---------|
| `bubbletea` | Bubble Tea TUI framework — full-screen interactive agent shells |
| `bubbles` | Bubble Tea UI components |
| `glamour` | Markdown rendering in terminal — render an agent's Markdown responses |
| `lipgloss` | Terminal style definitions |
| `gum` | Charm Gum interactive prompts — quick input/selection UI without a full TUI |
| `harmonica` | Animation utilities |
| `ntcharts` | Terminal charts — visualize agent metrics/traces inline |
| `curses` | Curses TUI bindings |

All standard `tty-*` toolkit gems are also included (`tty-prompt`, `tty-table`, `tty-spinner`, `tty-box`, `tty-font`, `tty-command`, `tty-config`, `tty-editor`, `tty-file`, `tty-link`, `tty-logger`, `tty-markdown`, `tty-option`, `tty-tree`, `tty-screen`, `tty-exit`) — `tty-prompt` and `tty-spinner` are the two you'll reach for first for a request/response CLI.

### Async & Multi-Agent Concurrency

| Gem | Purpose |
|-----|---------|
| `async` | Async I/O framework — run tool calls concurrently instead of sequentially |
| `falcon` | Async HTTP server — serve an agent over HTTP instead of a terminal |
| `concurrent-ruby` | Thread-safe data structures |
| `circuit_breaker` | Circuit breaker pattern — fail fast on a flaky tool/provider |
| `jongleur` | Job orchestration |
| `gush` | Workflow engine — multi-step agent pipelines with dependencies |
| `parallel` | Parallel execution |

### Agent Memory & Retrieval (RAG backends)

| Gem | Purpose |
|-----|---------|
| `daru` / `daru-view` | DataFrame and interactive plotting — inspect retrieval results |
| `sequel` / `sequel_pg` | SQL toolkit + PostgreSQL adapter |
| `pgvector` | pgvector extension for Sequel — vector similarity search |
| `ohm` / `ohm-contrib` | Redis object-hash mapping — structured response/session caching |
| `redis` / `redic` | Redis clients |
| `chroma-db` | Chroma vector database client |
| `oj` / `yajl-ruby` | Fast JSON parsing — tool-call payloads |
| `jsonl` | JSON Lines support — logging agent traces |
| `psych` / `yaml` | YAML parsing — tool/prompt config |

### NLP & Text Processing (Tool Bodies)

| Gem | Purpose |
|-----|---------|
| `tokenizers` | Hugging Face tokenizers — count tokens before hitting a context limit |
| `pragmatic_segmenter` | Sentence boundary detection |
| `pragmatic_tokenizer` | Text tokenization |
| `lingua` | Language detection |
| `wordnet` / `wordnet-defaultdb` | WordNet lexical database |
| `tf-idf-similarity` | TF-IDF and cosine similarity — lightweight retrieval without embeddings |
| `bm25f` | BM25F ranking algorithm |
| `fuzzy_tools` | Fuzzy string matching |
| `linguistics` | Linguistic analysis framework |
| `linkparser` | Link-grammar parser |
| `kramdown` | Markdown parser |
| `commonmarker` | CommonMark parser |
| `loofah` | HTML sanitization — clean tool output before feeding it back to a prompt |
| `treetop` | PEG parser generator — hand-write a parser for structured tool output |

### Document Ingestion Tools

| Gem | Purpose |
|-----|---------|
| `kreuzberg` | Document text extraction |
| `front_matter_parser` | YAML/TOML front matter parsing |
| `srt` | SRT subtitle parsing |
| `webvtt` | WebVTT caption parsing |
| `mimemagic` | MIME type detection |
| `redcarpet` | Markdown to HTML rendering |

### Web & Utilities

| Gem | Purpose |
|-----|---------|
| `sinatra` | Lightweight HTTP server — expose an agent as an HTTP endpoint |
| `graphql` | GraphQL client/server |
| `ratelimit` | Rate limiting — respect provider rate limits |
| `daemons` | Process daemonization — run an agent as a background service |
| `terrapin` | External command execution — shell out to another CLI tool |
| `dotenv` | `.env` file loading — API keys outside source |
| `amazing_print` | Pretty printing — inspect tool-call payloads in the REPL |
| `debug_me` | Debug output utilities |
| `pastel` | Terminal color styling |

## Patches

**`respond_to_missing.patch`** — Applied during build to `ruby-spacy`. Fixes a `respond_to_missing?` incompatibility between ruby-spacy and newer pycall versions. Without it, certain spaCy method delegation calls raise `NoMethodError` at runtime.

If updating ruby-spacy or pycall major versions, verify the patch still applies cleanly:

```bash
patch --dry-run -p1 < nlp/respond_to_missing.patch
```

## Customization

- **Add gems** — Edit `nlp/Gemfile`, run `bundle update`, rebuild with `rake build/nlp`. `nlp/Gemfile.lock` is regenerated inside the builder stage on every build and is not committed.
- **Change Ruby version** — Update `FROM rubylang/ruby:<version>-jammy` in `nlp/Containerfile` and `.ruby-version` together. Retest native gem compilation.
- **Pre-built gems** — The `gems/` directory at the repo root contains `.gem` files for gems unavailable on RubyGems (`ferret`, `nmatrix`, `nmatrix-fftw`, `nmatrix-lapacke`, `rbplotly`). Add additional `.gem` files there for gems requiring offline installation.
