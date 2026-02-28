### ⚠️ Identified Issues, Risks & Suggested Improvements

| Item (Code/Design/Requirement) | Issue/Risk Type | Description & Suggested Improvement | Severity (1–5) |
| --- | --- | --- | --- |
| Backlog Missing | 🚧 Risk | Without tracked work items, previously identified architectural flaws (GPU mismatch, open ports, hardcoded DB auth) will persist in production. **Improvement:** Generate Epics/Stories. | 4 |
| Postgres `trust` Auth | 🛡️ Security Vulnerability | DB is open without a password. **Improvement:** Task: Update `compose.yaml` to require `POSTGRES_PASSWORD` ([compose.yaml](https://www.google.com/search?q=b08x/docker-jupyter-ruby/docker-jupyter-ruby-7870706ac453249b8b58289c8a84b44e494ddf58/compose.yaml%23L39)). | 5 |
| Hardcoded GEM_HOME | 🐛 Bug | Breaks upon Ruby point release. **Improvement:** Task: Use dynamic shell evaluation `ruby -e "puts RbConfig::CONFIG['ruby_version']"` ([nlp/Containerfile](https://www.google.com/search?q=b08x/docker-jupyter-ruby/docker-jupyter-ruby-7870706ac453249b8b58289c8a84b44e494ddf58/nlp/Containerfile%23L8)). | 4 |

### 📌 Issue & Improvement Summary

* **Security Fix** (Story: Secure Infrastructure Defaults): *As a developer, I need secure-by-default container networking and authentication so that my local ML datasets and caches are not exposed to my local network.*
* **Task 1:** Change `POSTGRES_HOST_AUTH_METHOD` to require a password injected from `.env`.
* **Task 2:** Update Redis port mapping in `compose.yaml` from `6379:6379` to `127.0.0.1:6379:6379`.
* **Task 3:** Disable Jupyter default remote access in `jupyter_server_config.py`.

* **Performance Improvement** (Story: Enable GPU Acceleration for ML Pipelines): *As a data scientist, I need my container to utilize the allocated NVIDIA GPU so that LLM inference and spaCy operations are performant.*
* **Task 1:** Parameterize the `pip install` commands in `base/Containerfile` using an `ARG CUDA_SUPPORT`.
* **Task 2:** Add `CMAKE_ARGS="-DGGML_CUDA=on"` for `llama-cpp-python` if `CUDA_SUPPORT` is true.

* **Refactoring Suggestion** (Story: Resilient Build Process): *As a maintainer, I need the container build script to dynamically resolve paths so that routine dependency updates do not break the build.*
* **Task 1:** Replace hardcoded `3.3.0` strings with dynamically evaluated Ruby ABI versions.
* **Task 2:** Refactor the `ruby-spacy` patch application to use `bundle info --path ruby-spacy` to locate the target directory.

### 💡 Potential Optimizations/Integrations

| Specification/Component | Status | Clarification & Details | Confidence (1–5) |
| --- | --- | --- | --- |
| Automated pgvector Init | Unconfirmed | **Task:** Add a `init.sql` script to `/docker-entrypoint-initdb.d/` in the Postgres container to automatically run `CREATE EXTENSION vector;`. | 4 |
| Healthcheck Dependencies | Unconfirmed | **Task:** Ensure `nlp-notebook` service uses `depends_on` with `condition: service_healthy` targeting the Postgres database to prevent premature app startup. | 5 |

### ⚙️ Revised System/Module Overview (Incorporating Feedback)

The revised target architecture focuses on a production-ready, secure, and performant containerized Ruby ML environment. By executing the generated backlog, the system will shift from an open, CPU-bound sandbox to an optimized, hardware-accelerated workstation. The database (`pgvector`) and caching layer (`redis`) will be securely locked down to local network bindings, preventing data leakage ([compose.yaml]).

Simultaneously, the build pipeline defined in the Containerfiles will be refactored to remove brittle, hardcoded version strings ([nlp/Containerfile]nlp/Containerfile)). Instead, it will dynamically query the Ruby environment for gem paths and use build arguments to toggle between CPU and CUDA-compiled Python wheels. This ensures that the NVIDIA GPU reservations requested in the orchestration layer are actually utilized by the underlying LLM processing engines.
