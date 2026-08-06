# Ruby Agentic CLI Dev Environment

[![CI](https://github.com/b08x/docker-jupyter-ruby/actions/workflows/ci.yml/badge.svg)](https://github.com/b08x/docker-jupyter-ruby/actions/workflows/ci.yml)

A containerized JupyterLab + Ruby (IRuby) environment for prototyping agentic command-line tools before they ever touch a terminal. Wire up an LLM loop, a tool-calling MCP server, or a TUI front-end in a notebook cell, iterate against real API responses, then lift the working code into a standalone Ruby CLI.

Two images stack on top of each other: `base` (JupyterLab + Python AI stack — spaCy, local inference, Google's agent SDKs) and `nlp` (Ruby 3.3.8 + 100+ gems for LLM orchestration, MCP, async I/O, and terminal UIs, built on `base`). Run the `nlp` image directly; `base` exists as a foundation and as a source of Python-side tool implementations your Ruby agents can call into via `pycall`.

## Why a notebook for CLI agents

Agent loops are mostly API calls and control flow — the two things a REPL is best at. Building the loop in a notebook first means you see the raw request/response for every tool call, can replay a single step without re-running the whole program, and keep provider credentials/rate limits contained to one session instead of restarting a process on every retry. Once the loop, prompts, and tool schema are stable, the same requires (`ruby_llm`, `ruby_llm-mcp`, TTY gems) run identically in a `ruby` script outside the notebook — nothing here is notebook-only magic.

## Features

- **Agent Runtime** — `ruby_llm` (provider-agnostic chat/completion client) + `ruby_llm-mcp` (Model Context Protocol client/server) + `ruby_llm-schema` (tool-call schema generation) for building request/response agent loops
- **DSPy & Structured Prompting** — Full `dspy` suite (OpenAI/Gemini adapters, `dspy-code_act` code-generating agent, `dspy-deep_research`/`dspy-deep_search`, `dspy-evals`, Langfuse observability) for programs that optimize their own prompts instead of hand-tuned strings
- **Multi-Agent Orchestration** — `claude_swarm` for coordinating multiple Claude agents; `async` + `falcon` + `jongleur`/`gush` for concurrent tool execution and workflow orchestration outside a single request/response cycle
- **CLI/TUI Front-Ends** — The Charm/Bubble suite (`bubbletea`, `lipgloss`, `gum`, `glamour`) and the full `tty-*` toolkit (`tty-prompt`, `tty-spinner`, `tty-table`, `tty-markdown`, ...) for the interactive shell an agent actually runs in
- **Tool Backends** — pgvector (PostgreSQL), Chroma DB, and Redis for RAG/memory tools; `kreuzberg`, `srt`/`webvtt`, `pragmatic_segmenter` for document-ingestion tools; `pycall` + `ruby-spacy` to call spaCy's NER/parsing pipeline as a tool from Ruby
- **CUDA/CPU Toggle** — The `base` image supports a `CUDA_SUPPORT` build arg for local model inference (`llama-cpp-python`, PyTorch-backed tools) without a GPU dependency at prototype time

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

Starts `nlp-notebook`, `redis` (redis-stack), and `pgvector` (PostgreSQL + pgvector) — the memory/RAG backends an agent's tools will reach for:

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

### A minimal agent loop with ruby_llm

```ruby
require 'ruby_llm'

RubyLLM.configure { |c| c.openai_api_key = ENV['OPENAI_API_KEY'] }

chat = RubyLLM.chat(model: 'gpt-4o-mini')
chat.with_tool(:search_docs) do |query:|
  # tool implementation — e.g. a pgvector nearest-neighbor lookup, see below
  "3 matching documents for '#{query}'"
end

response = chat.ask("Find docs about connection pooling, then summarize them.")
puts response.content
```

`ruby_llm` handles the tool-call/tool-result round trip; the block above is the entire tool implementation. This is the loop you'd extract verbatim into a CLI entrypoint.

### Exposing a tool over MCP with ruby_llm-mcp

```ruby
require 'ruby_llm/mcp'

server = RubyLLM::MCP::Server.new(name: 'notebook-tools') do |s|
  s.tool('lookup_entity') do |text:|
    require 'ruby-spacy'
    nlp = Spacy::Language.new('en_core_web_sm')
    nlp.read(text).ents.map { |e| { text: e.text, label: e.label } }
  end
end

server.start  # stdio transport — any MCP client (Claude Code, Claude Desktop, another agent) can now call `lookup_entity`
```

Prototype the tool body in a notebook cell against real input first; once it behaves, the `server.start` call is all that changes when it moves to a standalone script.

### An interactive TUI front-end with tty-prompt + gum

```ruby
require 'tty-prompt'
require 'tty-spinner'

prompt = TTY::Prompt.new
query = prompt.ask("Ask the agent:")

spinner = TTY::Spinner.new("[:spinner] thinking...", format: :dots)
spinner.auto_spin
response = chat.ask(query)   # `chat` from the ruby_llm example above
spinner.stop

puts response.content
```

This is the shell the agent actually lives in — build and test it in the notebook against a live `chat` session, then drop it unchanged into `bin/agent`.

### Semantic memory for an agent with pgvector

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

This is the `search_docs` tool body from the first example — pgvector as the agent's long-term memory. Add an HNSW index once the table has real volume:

```ruby
DB.add_index :documents, :embedding, type: 'hnsw', opclass: 'vector_l2_ops'
```

### Caching LLM calls with Redis

Agent loops re-ask the same tool/prompt combinations constantly during development — cache by prompt+model to stop burning API budget on repeats:

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

    result = RubyLLM.chat(model: model).ask(prompt)
    create(prompt: prompt, response: result.content, model: model).response
  end
end
```

## From notebook to CLI

Once a loop works in Jupyter:

1. Extract the tool blocks and agent setup into a plain `.rb` file — nothing in the examples above depends on the notebook kernel.
2. Wrap the entrypoint with a `tty-*`/`gum`-based prompt loop (or `sinatra`/`falcon` if it needs to serve HTTP instead of a terminal).
3. Add a `bin/` script with a shebang; the same `nlp` image gem set is what a production container for the CLI would need — see `nlp/README.md` for the full gem reference.

## Customization

- **Add or update gems** — Edit `nlp/Gemfile`, run `bundle update`, rebuild with `rake build/nlp`.
- **Change Ruby version** — Update `FROM rubylang/ruby:<version>-jammy` in `nlp/Containerfile` and `.ruby-version` together.
- **Python packages** — Edit `pip install` lines in `base/Containerfile`; rebuild both images after.
- **CUDA support** — Pass `--build-arg CUDA_SUPPORT=true` to `base/Containerfile` to switch all PyTorch installs to CUDA 12.1 wheels.
- **Jupyter config** — Edit `base/jupyter_server_config.py`. Remote access is disabled by default; set `c.ServerApp.allow_remote_access = True` to enable.

## Troubleshooting

**Build fails** — Ensure 4 GB+ available memory. Check `nlp/Gemfile` for conflicting gem constraints (`nlp/Gemfile.lock` is regenerated on every build, not committed).

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

Fork the repository and open a pull request. Include `rake build/nlp` output confirming a successful local build.

## License

MIT. Base images carry their own licenses; see [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks) for details.
