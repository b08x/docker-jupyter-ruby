# Jupyter Notebook with Ruby Kernel & NLP Gems

[![CI](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml/badge.svg)](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml)

Based on RubyData's [docker-stacks](https://github.com/RubyData/docker-stacks) and official [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks).

<img src="docs/assets/img/info_graphic.webp" alt="Process Diagram" width="500">

Docker images that bundle Jupyter with a Ruby kernel (IRuby) and NLP-focused RubyGems.

What you get:

- **Ruby Kernel** — Run Ruby code in Jupyter notebooks via IRuby
- **NLP Libraries** — Text tokenization, sentiment analysis, entity extraction via ruby-spacy
- **Python Bridge** — Use Python libraries (spaCy, NLTK, Transformers) from Ruby through [pycall](https://github.com/mrkn/pycall.rb)
- **Data Tools** — [daru](https://github.com/SciRuby/daru) and [daru-view](https://github.com/SciRuby/daru-view) for data manipulation
- **Vector DBs** — pgvector, chroma-db, and redis via Docker Compose
- **TTY Toolkit** — Terminal apps via [tty-toolkit](https://ttytoolkit.org/)

## Prerequisites

- [Podman](https://podman.io/getting-started/installation) or [Docker](https://www.docker.com/get-started)
- [Podman Compose](https://github.com/containers/podman-compose) or [Docker Compose](https://docs.docker.com/compose/install/)

This project uses Podman by default. Replace `podman` with `docker` in commands if you prefer Docker.

## Building

Two images: `base` (Jupyter + Python/AI) and `nlp` (Ruby 3.3.8 + gems).

- **[base](./base/README.md)** — JupyterLab, Python data science libraries, spaCy, Google Generative AI SDK
- **[nlp](./nlp/README.md)** — Ruby 3.3.8, IRuby kernel, 100+ gems

Clone and build:

```bash
git clone https://github.com/b08x/jupyter-ruby-docker.git
cd jupyter-ruby-docker
bundle install
rake build/nlp      # Builds base + nlp (recommended)
```

Or build directly with Podman:

```bash
podman build -t b08x/notebook-base:latest -f base/Containerfile .
podman build -t b08x/notebook-nlp:latest -f nlp/Containerfile .
```

## Running

### With podman run

```bash
podman run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user $(id -u):$(id -g) \
  b08x/notebook-nlp:latest
```

Create the `work` directory first. Open `http://localhost:8888` and use the token from the container logs.

### With Compose (Recommended)

Starts notebook, redis, and pgvector services.

Setup:

```bash
cp .env.example .env
mkdir -p ./data
```

Edit `.env` to set `UID`, `GID`, and API keys (`OPENAI_API_KEY`, etc.).

Start:

```bash
podman-compose up -d
```

Access:
- Jupyter: `http://localhost:8888` — get token via `podman-compose logs nlp-notebook | grep token`
- RedisInsight: `http://localhost:8001`
- PostgreSQL + pgvector: `localhost:5432` (database: `rubynlp`)

Stop:

```bash
podman-compose down
```

## Examples

### NLP Text Processing

```ruby
require 'ruby-spacy'
require 'terminal-table'

nlp = Spacy::Language.new('en_core_web_sm')
text = "Apple Inc. is planning to open a new store in San Francisco."
doc = nlp.read(text)

rows = doc.ents.map { |ent| [ent.text, ent.label, ent.start_char, ent.end_char] }
table = Terminal::Table.new(headings: ['Entity', 'Type', 'Start', 'End'], rows: rows)
puts table
```

Ruby-spacy bridges Ruby and Python's spaCy through pycall.

### LLM Integration

```ruby
require 'langchain'

llm = Langchain::LLM::OpenAI.new(api_key: ENV['OPENAI_API_KEY'])
assistant = Langchain::Assistant.new(llm: llm, instructions: "You're a Ruby expert")
assistant.add_message_and_run!(content: "Explain procs vs lambdas")
puts assistant.messages.last.content
```

LangChain supports 10+ LLM providers: OpenAI, Groq, Ollama, Google, and others.

### Vector Search

```ruby
require 'sequel'
require 'pgvector'

DB = Sequel.connect('postgres://postgres@pgvector:5432/rubynlp')
DB.run('CREATE EXTENSION IF NOT EXISTS vector')

DB.run <<-SQL
  CREATE TABLE documents (id SERIAL, content TEXT, embedding vector(384))
SQL

# Insert and query similar documents
```

pgvector runs semantic search and RAG directly in PostgreSQL.

### Redis Caching

```ruby
require 'ohm'

class LLMResponse < Ohm::Model
  attribute :prompt
  attribute :response
  index :prompt

  def self.cached_or_fetch(prompt, model:)
    cached = find(prompt: prompt, model: model).first
    return cached.response if cached

    # Fetch from LLM, store in cache, return
  end
end
```

Caching cuts costs and latency for repeated LLM queries.

## Customization

- **Ruby Version** — Edit `nlp/Containerfile`, change `RUBY_VERSION`
- **Gems** — Edit `nlp/Gemfile`, run `bundle update`, rebuild with `rake build/nlp`
- **Python Packages** — Edit `pip install` commands in `base/Containerfile`
- **Jupyter Config** — Modify `base/jupyter_server_config.py` — remote access enabled by default

## Troubleshooting

- **Build fails** — Ensure 4GB+ memory; check `nlp/Gemfile.lock` for conflicts
- **Port binding** (rootless Podman) — Use ports ≥1024 or set `net.ipv4.ip_unprivileged_port_start=80`
- **Kernel missing** — Check `iruby register --force` in logs
- **Remote access** — Verify firewall allows port 8888
- **Token** — Get from logs: `podman-compose logs nlp-notebook | grep token`
- **Database issues** — Check service health: `podman-compose ps`

## Recent Changes

### Version 1.1.0 (December 2024)

- **Podman Migration** — Renamed `Dockerfile` → `Containerfile`, `docker-compose.yml` → `compose.yaml`
- **Ruby Pinning** — Locked Ruby to 3.3.8 with `Gemfile.lock`
- **Remote Access** — Enabled by default in Jupyter config
- **Local LLM** — Added `llama-cpp-python` support

## License

MIT. See LICENSE file.

## Contributing

Fork and submit pull requests.

---

Detailed documentation follows below.

---

Jupyter notebooks with a Ruby kernel (IRuby) and RubyGems focused on NLP and language model utilities.

## Features

- **Ruby Kernel for Jupyter** — Use IRuby to run Ruby code interactively within Jupyter notebooks.
- **NLP Libraries** — Gems for text tokenization, sentiment analysis, entity extraction, and language utilities.
- **Cross-Language Integration** — Use Python libraries (spaCy, NLTK, Transformers) from Ruby via [pycall](https://github.com/mrkn/pycall.rb).
- **Data Analysis** — [daru](https://github.com/SciRuby/daru) and [daru-view](https://github.com/SciRuby/daru-view) for data manipulation and visualization.
- **Vector Database Integration** — pgvector, chroma-db, and redis services via Docker Compose.
- **TTY Gem Suite** — Components from [TTY-Toolkit](https://ttytoolkit.org/) for terminal applications.

## Prerequisites

- [Podman](https://podman.io/getting-started/installation) installed (or [Docker](https://www.docker.com/get-started) if you prefer).
- [Podman Compose](https://github.com/containers/podman-compose) or [Docker Compose](https://docs.docker.com/compose/install/) (recommended).
- Basic familiarity with Jupyter Notebooks and containers.

This project uses Podman as the primary runtime (OCI-compliant Containerfiles). Replace `podman` with `docker` in any command if you prefer.

## Building the Images

Two primary images: `base` and `nlp`.

- **[base](./base/README.md)** — Built on `jupyter/docker-stacks-foundation`, includes JupyterLab, Python data science libraries, spaCy, Google Generative AI SDK.
- **[nlp](./nlp/README.md)** — Built on `base`, adds Ruby 3.3.8, IRuby kernel, and 100+ Ruby gems for NLP, LLM integration, and vector search.

1. Clone the repository:

    ```bash
    git clone https://github.com/b08x/jupyter-ruby-docker.git
    cd jupyter-ruby-docker
    ```

2. Build:

    Via Podman:

    ```bash
    # Build base first (if not pulling from registry)
    podman build -t b08x/notebook-base:latest -f base/Containerfile .

    # Build NLP image
    podman build -t b08x/notebook-nlp:latest -f nlp/Containerfile .
    ```

    Docker users: replace `podman` with `docker` above.

    Via Rake (recommended):

    ```bash
    bundle install
    rake build/nlp  # Uses Podman by default, builds base automatically
    rake build-all
    ```

    The Rakefile uses Podman by default. Change `CONTAINER_RUNTIME` to use Docker.

## Running the Container

Run `nlp` via `podman run` or `podman-compose`. Compose is recommended — it sets up redis and pgvector services too.

### Using podman run (notebook only)

Starts just the Jupyter notebook container.

```bash
podman run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user $(id -u):$(id -g) \
  b08x/notebook-nlp:latest
```

Docker users: replace `podman` with `docker`.

Flags explained:

- `--rm` — Remove container on exit
- `-p 8888:8888` — Map host port 8888 to container port 8888
- `-v "${PWD}/work":/home/jovyan/work` — Mount local `work` directory into container. Create it first with `mkdir work`.
- `--user $(id -u):$(id -g)` — Run as current user/group to avoid permission issues
- `b08x/notebook-nlp:latest` — Image to run

Access Jupyter at `http://localhost:8888`. The startup logs show the authentication token.

Note: Jupyter has remote access enabled by default. Use `http://<your-machine-ip>:8888` with the token to connect from another machine on your network.

### Using Compose (recommended: notebook + redis + pgvector)

Uses `compose.yaml` to start `nlp-notebook`, `redis`, and `pgvector` together.

1. Environment setup (`.env` file):

    Copy the example file:

    ```bash
    cp .env.example .env
    ```

    Edit `.env`:
    - **`UID` & `GID`** — (Optional) Set to your user/group ID (`id -u`, `id -g`) if not 1000. Ensures correct permissions for mounted volumes.
    - **`WORKSPACE`** — (Optional) Absolute path to directory for notebooks and work files. If empty, `compose.yaml` defaults to `./data` mounted at `/home/jovyan/work`.
    - **API Keys** — (Optional) Set `OPENAI_API_KEY`, `GOOGLE_PALM_API_KEY`, `HUGGINGFACE_API_KEY` if you plan to use those services.

2. Create necessary directories:

    If `WORKSPACE` is set in `.env`, ensure that directory exists:

    ```bash
    mkdir -p /path/to/my/notebooks
    ```

    If `WORKSPACE` is not set, create the default `./data`:

    ```bash
    mkdir -p ./data
    ```

    The default `compose.yaml` also mounts `$HOME/Documents` as read-only to `/home/jovyan/Library`. Ensure this exists if you plan to use it:

    ```bash
    mkdir -p ~/Documents
    ```

3. Start services:

    ```bash
    podman-compose up -d
    # or for Docker:
    docker compose up -d
    ```

    `-d` runs containers detached (background).

    Use `podman-compose` or `docker compose` (v2), not the older `docker-compose`.

4. Access services:
    - **Jupyter Lab**: `http://localhost:8888` — get token from `podman-compose logs nlp-notebook`
    - **RedisInsight**: `http://localhost:8001`
    - **PostgreSQL + pgvector**: `localhost:5432` (database: `rubynlp`, user: `postgres`)

5. Stopping:

    ```bash
    podman-compose down
    # or for Docker:
    docker compose down
    ```

## Included RubyGems

See [nlp/README](./nlp/README.md) for the complete list.

## Examples

Try these in a new Ruby notebook.

### Example 1: NLP text processing pipeline

Multilingual text analysis with ruby-spacy and named entity recognition:

```ruby
require 'ruby-spacy'
require 'terminal-table'

# Initialize spaCy with English model
nlp = Spacy::Language.new('en_core_web_sm')

# Process text and extract entities
text = "Apple Inc. is planning to open a new store in San Francisco."
doc = nlp.read(text)

# Display entities
rows = []
doc.ents.each do |ent|
  rows << [ent.text, ent.label, ent.start_char, ent.end_char]
end

table = Terminal::Table.new(
  headings: ['Entity', 'Type', 'Start', 'End'],
  rows: rows
)
puts table
# Output:
# +-----------+-------+-------+-----+
# | Entity    | Type  | Start | End |
# +-----------+-------+-------+-----+
# | Apple Inc | ORG   | 0     | 9   |
# | San Fran  | GPE   | 40    | 53  |
# +-----------+-------+-------+-----+
```

Ruby-spacy bridges Ruby and Python's spaCy via pycall, giving you production-grade NLP in Ruby.

### Example 2: LLM integration with LangChain

Build a conversational AI assistant:

```ruby
require 'langchain'

# Initialize LLM (supports OpenAI, Groq, Ollama, etc.)
llm = Langchain::LLM::OpenAI.new(
  api_key: ENV['OPENAI_API_KEY'],
  default_options: { temperature: 0.7, chat_model: 'gpt-4' }
)

# Create assistant
assistant = Langchain::Assistant.new(
  llm: llm,
  instructions: "You're a helpful Ruby programming expert"
)

# Conversation
assistant.add_message_and_run!(
  content: "Explain the difference between a proc and a lambda in Ruby"
)

puts assistant.messages.last.content
```

Structured output with JSON schema:

```ruby
require 'langchain'

json_schema = {
  type: "object",
  properties: {
    sentiment: { type: "string", enum: ["positive", "negative", "neutral"] },
    confidence: { type: "number", description: "Confidence score 0-1" },
    keywords: {
      type: "array",
      items: { type: "string" },
      description: "Key topics extracted"
    }
  },
  required: ["sentiment", "confidence", "keywords"]
}

parser = Langchain::OutputParsers::StructuredOutputParser.from_json_schema(json_schema)
prompt = Langchain::Prompt::PromptTemplate.new(
  template: "Analyze this text:\n{format_instructions}\n\nText: {text}",
  input_variables: ["text", "format_instructions"]
)

text = "I absolutely loved the new Ruby 3.3 features! Pattern matching is incredible."
prompt_text = prompt.format(
  text: text,
  format_instructions: parser.get_format_instructions
)

response = llm.chat(messages: [{ role: "user", content: prompt_text }])
analysis = parser.parse(response.chat_completion)
# => { "sentiment" => "positive", "confidence" => 0.95, "keywords" => ["Ruby 3.3", "pattern matching"] }
```

LangChain provides a unified API across 10+ LLM providers (OpenAI, Groq, Ollama, Google, etc.), making your code portable.

### Example 3: Vector database and semantic search

PostgreSQL with pgvector for semantic similarity:

```ruby
require 'sequel'
require 'pgvector'

# Connect to PostgreSQL
DB = Sequel.connect('postgres://postgres@pgvector:5432/rubynlp')

# Enable pgvector
DB.run('CREATE EXTENSION IF NOT EXISTS vector')

# Create table with vector column
DB.run <<-SQL
  CREATE TABLE IF NOT EXISTS documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(384)
  )
SQL

# Define model
class Document < Sequel::Model
  plugin :pgvector, :embedding
end

# Insert with embeddings
# (Use an embedding model like sentence-transformers in practice)
Document.create(
  content: "Ruby is great for web development",
  embedding: [0.1, 0.2, 0.3, ...] # 384-dimensional vector
)

Document.create(
  content: "Python is widely used for data science",
  embedding: [0.15, 0.18, 0.25, ...]
)

# Similarity search with Euclidean distance
query_embedding = [0.12, 0.19, 0.28, ...]
similar_docs = Document.nearest_neighbors(
  :embedding,
  query_embedding,
  distance: 'euclidean'
).limit(5)

similar_docs.each do |doc|
  puts "#{doc.content} (distance: #{doc.values[:distance]})"
end
```

Create an index for faster searches (after bulk inserts):

```ruby
# HNSW index for approximate nearest neighbor
DB.add_index :documents, :embedding, type: 'hnsw', opclass: 'vector_l2_ops'
```

Vector databases enable semantic search, RAG, and similarity-based recommendations. pgvector integrates into PostgreSQL, avoiding separate vector databases.

### Example 4: Caching LLM responses with Redis

Cache expensive LLM calls to avoid redundant API requests:

```ruby
require 'ohm'
require 'langchain'

# Ohm connects via OHM_URL env var
# (Set in compose.yaml: redis://redis:6379/1)

class LLMResponse < Ohm::Model
  attribute :prompt
  attribute :response
  attribute :model
  index :prompt

  def self.cached_or_fetch(prompt, model:)
    # Check cache first
    cached = find(prompt: prompt, model: model).first
    return cached.response if cached

    # Cache miss: fetch from LLM
    llm = Langchain::LLM::OpenAI.new(api_key: ENV['OPENAI_API_KEY'])
    result = llm.chat(
      messages: [{ role: "user", content: prompt }]
    )
    response_text = result.chat_completion

    # Store in cache
    create(
      prompt: prompt,
      response: response_text,
      model: model
    )

    response_text
  end
end

# Usage
response = LLMResponse.cached_or_fetch(
  "Explain closures in Ruby",
  model: "gpt-4"
)
puts response  # Subsequent calls return instantly from cache
```

LLM API calls are slow and expensive. Caching improves response times and reduces costs.

## Customization and Development

Customize the images:

- **Ruby Version** — Pinned to 3.3.8. Edit `nlp/Containerfile` and update `RUBY_VERSION` to change.
- **Ruby Gems** — Modify `nlp/Gemfile`, rebuild with `rake build/nlp` or `podman build -f nlp/Containerfile .`. Uses `Gemfile.lock` for reproducible builds. Run `bundle update` to update dependencies.
- **Python Packages** — Edit `pip install` or `mamba install` commands in `base/Containerfile`. Rebuild both images after.
- **System Packages** — Add libraries via `apt-get install`:
  - `base/Containerfile` for Python or base dependencies
  - `nlp/Containerfile` for Ruby-specific or runtime dependencies
- **Jupyter Configuration** — Modify `base/jupyter_server_config.py`. Remote access enabled by default (`c.ServerApp.allow_remote_access = True`). Disable if you only need local access.
- **Kernel Patching** — `nlp/Containerfile` applies `respond_to_missing.patch` to ruby-spacy for pycall compatibility. Customize and rebuild if needed.

## Troubleshooting

### Build issues

- **Verify resources** — Ensure 4GB+ memory for builds
- **Check build logs** — Podman: `podman build --no-cache -f nlp/Containerfile .` / Docker: `docker build --no-cache -f nlp/Containerfile .`
- **Dependency conflicts** — Check `nlp/Gemfile.lock` if gem installation fails

### Podman-specific issues

- **Port binding with rootless Podman** — Ports < 1024 need privileges. Use ports ≥1024 or configure sysctl:

  ```bash
  echo "net.ipv4.ip_unprivileged_port_start=80" | sudo tee /etc/sysctl.d/podman-ports.conf
  sudo sysctl --system
  ```

- **SELinux volume mount errors** — "Permission denied" on Fedora/RHEL:
  - The `:Z` flag in `compose.yaml` handles this automatically
  - For manual runs: `-v ./data:/home/jovyan/work:Z`

- **Compose command** — Use `podman-compose` or `docker compose` (v2), NOT `docker-compose`:

  ```bash
  # Correct
  podman-compose up -d
  docker compose up -d

  # Deprecated
  docker-compose up -d
  ```

### Jupyter and access issues

- **Kernel not appearing** — Ensure `iruby register --force` ran during build. Check logs:

  ```bash
  podman logs <container_id>  # or docker logs <container_id>
  ```

- **Remote access not working** —
  - Remote access enabled by default in `base/jupyter_server_config.py`
  - Check firewall: `sudo firewall-cmd --add-port=8888/tcp --permanent` (Fedora/RHEL)
  - Verify container listens on `0.0.0.0`: `podman port <container_name>`

- **Token issues** — Get token from logs:

  ```bash
  podman-compose logs nlp-notebook | grep token
  docker compose logs nlp-notebook | grep token
  ```

### Port conflicts

If ports 8888, 6379, or 5432 are in use:

- **Option 1** — Stop conflicting services
- **Option 2** — Change ports in `compose.yaml`:

  ```yaml
  ports:
    - "8889:8888"  # Map host port 8889 to container port 8888
  ```

### Volume permission errors

- **With podman run/docker run** — Use `--user $(id -u):$(id -g)`
- **With compose** — Set `UID` and `GID` in `.env` to match host user:

  ```bash
  echo "UID=$(id -u)" >> .env
  echo "GID=$(id -g)" >> .env
  ```

### Database connection issues

- **PostgreSQL not accessible** — Ensure pgvector service is healthy:

  ```bash
  podman-compose ps  # Check pgvector status
  ```

- **Redis connection refused** — Verify Redis on port 6379:

  ```bash
  podman exec -it redis redis-cli ping  # Should return "PONG"
  ```

## Recent Changes

### Version 1.1.0 (December 2024)

Infrastructure improvements:

- **Podman Migration** — Migrated from Docker to Podman
  - Renamed `Dockerfile` to `Containerfile` (OCI standard)
  - Updated `docker-compose.yml` to `compose.yaml`
  - Rakefile uses Podman by default (Docker still supported)

- **Ruby Version Pinning** — Locked to Ruby 3.3.8
  - Added `Gemfile.lock` for reproducible builds
  - Consistent behavior across environments

- **Jupyter Remote Access** — Enabled by default
  - Set `c.ServerApp.allow_remote_access = True` in `base/jupyter_server_config.py`
  - Access from any machine on your network (secure your deployment)

- **Local LLM Support** — Enhanced llama-cpp-python
  - Added `build-essential` and `gcc` to base image
  - Fixed PyPI index URL
  - Run local LLMs (Llama, Mistral, etc.) in notebooks

- **Documentation** —
  - Added illustrative examples (NLP pipelines, LLM integration, vector search)
  - Podman-first docs with Docker alternatives
  - Expanded troubleshooting with Podman-specific solutions

See [git commit history](https://github.com/b08x/jupyter-ruby-docker/commits/main) for full details.

## License

MIT License. See LICENSE file. Base images may have their own licenses (e.g., BSD for Jupyter Docker Stacks).

## Contributing

Fork the repository and submit pull requests.
