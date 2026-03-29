# Jupyter Notebook with Ruby Kernel & NLP Gems

[![CI](https://github.com/b08x/docker-jupyter-ruby/actions/workflows/ci.yml/badge.svg)](https://github.com/b08x/docker-jupyter-ruby/actions/workflows/ci.yml)

Docker images providing JupyterLab with a Ruby (IRuby) kernel, a Python NLP bridge, and a broad gem collection for LLM integration, DSPy, vector search, and terminal UI development.

Two images stack on top of each other: `base` (JupyterLab + Python AI stack) and `nlp` (Ruby 3.3.8 + 100+ gems, built on `base`). Run the `nlp` image directly; `base` exists as a foundation.

## Features

- **IRuby Kernel** — Ruby runs natively inside JupyterLab notebooks; no subprocess wrappers, no language switching
- **Python NLP Bridge** — `pycall` + `ruby-spacy` expose spaCy's full pipeline (NER, tokenization, dependency parsing) from Ruby code
- **DSPy & LLM Stack** — Full `dspy` suite with provider adapters, plus `ruby_llm`, `langchainrb`, `groq`, `ollama-ai`, and `rllama` for local GGUF inference
- **Vector & Semantic Search** — pgvector (PostgreSQL), Chroma DB, and Redis available without extra setup when running via Compose
- **Async Runtime** — `async`, `falcon`, `circuit_breaker`, and `jongleur` for non-blocking workflows alongside synchronous notebook code
- **Charm/Bubble TUI** — `bubbletea`, `glamour`, `lipgloss`, `gum`, and the full `tty-*` toolkit for terminal UIs built and tested in notebooks
- **CUDA/CPU Toggle** — The `base` image supports a `CUDA_SUPPORT` build arg that switches all PyTorch-backed installs between CPU and CUDA 12.1 wheels

## Prerequisites

- [Podman](https://podman.io/getting-started/installation) or [Docker](https://www.docker.com/get-started)
- [Podman Compose](https://github.com/containers/podman-compose) or Docker Compose v2

Podman is the default runtime. Replace `podman` / `podman-compose` with `docker` / `docker compose` throughout if preferred.

## Building

```bash
git clone https://github.com/b08x/docker-jupyter-ruby.git
cd docker-jupyter-ruby
bundle install

rake build/nlp      # Builds base then nlp (recommended)
rake build/base     # Base image only
rake build-all      # Both images in sequence
```

The Rakefile detects the container engine automatically (checks for a running `dockerd`, falls back to `podman`). Image ownership is read from `DOCKER_USER` env var, defaulting to `$USER`.

Direct build (skips Rake):

```bash
podman build --format docker -f base/Containerfile --rm -t b08x/notebook-base:latest .
podman build --format docker -f nlp/Containerfile  --rm -t b08x/notebook-nlp:latest  .
```

## Running

### Notebook only

```bash
mkdir -p ./work
podman run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user "$(id -u):$(id -g)" \
  b08x/notebook-nlp:latest
```

Open `http://localhost:8888`. The authentication token appears in container stdout.

### Full stack via Compose (recommended)

Starts `nlp-notebook`, `redis` (redis-stack), and `pgvector` (PostgreSQL + pgvector):

```bash
cp compose.yaml.example compose.yaml   # Customize volumes, ports, or GPU settings
cp .env.example .env                   # Set UID, GID, WORKSPACE, and API keys
mkdir -p ./data

podman-compose up -d
podman-compose logs nlp-notebook | grep token   # Get Jupyter token
podman-compose down
```

`compose.yaml.example` is the baseline — it omits personal host directory mounts present in the default `compose.yaml`. Edit the copy to add any additional volume bindings before starting.

Service endpoints:

- Jupyter: `http://localhost:8888`
- RedisInsight: `http://localhost:8003`
- PostgreSQL/pgvector: `localhost:5432` (database: `rubynlp`, user: `postgres`)

## Examples

### NLP with ruby-spacy

```ruby
require 'ruby-spacy'
require 'terminal-table'

nlp = Spacy::Language.new('en_core_web_sm')
doc = nlp.read("Apple Inc. is planning to open a new store in San Francisco.")

rows = doc.ents.map { |ent| [ent.text, ent.label, ent.start_char, ent.end_char] }
puts Terminal::Table.new(headings: ['Entity', 'Type', 'Start', 'End'], rows: rows)
```

`ruby-spacy` delegates to the Python spaCy process via `pycall`. The `respond_to_missing.patch` in `nlp/` keeps the delegation working across pycall versions.

### LLM integration with LangChain

```ruby
require 'langchain'

llm = Langchain::LLM::OpenAI.new(api_key: ENV['OPENAI_API_KEY'])
assistant = Langchain::Assistant.new(llm: llm, instructions: "You're a Ruby expert")
assistant.add_message_and_run!(content: "Explain procs vs lambdas")
puts assistant.messages.last.content
```

`langchainrb` supports OpenAI, Groq, Ollama, Google, and other providers behind a common interface.

### Semantic search with pgvector

```ruby
require 'sequel'
require 'pgvector'

DB = Sequel.connect('postgres://postgres@pgvector:5432/rubynlp')
DB.run('CREATE EXTENSION IF NOT EXISTS vector')
DB.run('CREATE TABLE IF NOT EXISTS documents (id SERIAL PRIMARY KEY, content TEXT, embedding vector(384))')

class Document < Sequel::Model
  plugin :pgvector, :embedding
end

similar = Document.nearest_neighbors(:embedding, query_vector, distance: 'euclidean').limit(5)
```

After bulk inserts, add an HNSW index for approximate nearest-neighbor queries:

```ruby
DB.add_index :documents, :embedding, type: 'hnsw', opclass: 'vector_l2_ops'
```

### LLM response caching with Redis

```ruby
require 'ohm'

class LLMResponse < Ohm::Model
  attribute :prompt
  attribute :response
  attribute :model
  index :prompt

  def self.cached_or_fetch(prompt, model:)
    cached = find(prompt: prompt, model: model).first
    return cached.response if cached

    result = Langchain::LLM::OpenAI.new(api_key: ENV['OPENAI_API_KEY'])
                                   .chat(messages: [{ role: 'user', content: prompt }])
    create(prompt: prompt, response: result.chat_completion, model: model).response
  end
end
```

## Customization

- **Add or update gems** — Edit `nlp/Gemfile`, run `bundle update`, rebuild with `rake build/nlp`. Commit the updated `Gemfile.lock`.
- **Change Ruby version** — Update `FROM rubylang/ruby:<version>-jammy` in `nlp/Containerfile` and `.ruby-version` together.
- **Python packages** — Edit `pip install` lines in `base/Containerfile`; rebuild both images after.
- **CUDA support** — Pass `--build-arg CUDA_SUPPORT=true` to `base/Containerfile` to switch all PyTorch installs to CUDA 12.1 wheels.
- **Jupyter config** — Edit `base/jupyter_server_config.py`. Remote access is disabled by default; set `c.ServerApp.allow_remote_access = True` to enable.

## Troubleshooting

**Build fails** — Ensure 4 GB+ available memory. Check `nlp/Gemfile.lock` for conflicting constraints.

**IRuby kernel missing** — Verify `iruby register --force` ran during build:

```bash
podman logs notebook-nlp | grep iruby
```

**Port conflicts** — Change the host-side port in `compose.yaml` (e.g., `"8889:8888"`) if 8888 is occupied.

**SELinux volume errors** (Fedora/RHEL) — The `:Z` flag in `compose.yaml` handles relabeling automatically. For manual `podman run`, append `:Z` to volume mounts.

**Database not ready** — pgvector has a healthcheck; `nlp-notebook` won't start until it passes. Check with:

```bash
podman-compose ps
podman exec -it redis redis-cli ping   # should return PONG
```

## Contributing

Fork the repository and open a pull request. Include `rake build/nlp` output confirming a successful local build. Update `Gemfile.lock` if changing gem dependencies.

## License

MIT. Base images carry their own licenses; see [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks) for details.
