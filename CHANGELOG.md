## [1.2.0] - 2026-03-29

### 🐛 Bug Fixes

- Use DOCKER_USER environment variable for image ownership in CI and Rakefile.
- Use environment variables for DOCKER_USER in CI workflow steps.

### ⚙️ Miscellaneous Tasks

- Removed old backlog doc, updated changelog
- Update image tagging and pushing to use explicit docker.io registry. fixes: """Error: trying to reuse blob sha256:efafae78d70c98626c521c246827389128e7d7ea442db31bc433934647f0c791 at destination: pinging container registry localhost: Get "<https://localhost/v2/>": dial tcp [::1]:443: connect: connection refused"""

## [1.0.0] - 2026-02-28

### 🚀 Features

- Migrate build system from Docker to Podman, updating environment variables, file names, and build arguments.
- Enable Jupyter remote access, improve Docker Compose service configurations and permissions, and update Python dependencies.
- Migrate build system from Docker to Podman, updating environment variables, file names, and build arguments.
- Enable Jupyter remote access, improve Docker Compose service configurations and permissions, and update Python dependencies.

### 🚜 Refactor

- Use variable for Docker image tags in CI workflow

### 📚 Documentation

- Update README to promote Podman, enhance image descriptions, and added some Ruby NLP/LLM examples.
- Update README to promote Podman, enhance image descriptions, and added some Ruby NLP/LLM examples.
- Streamline READMEs and update NLP container configuration

### ⚙️ Miscellaneous Tasks

- Pin Ruby version to 3.3.8 and introduce locked dependencies for the main project and nlp sub-project.
- Add build-essential and gcc, and install llama-cpp-python from its dedicated PyPI index.
- Pin Ruby version to 3.3.8 and introduce locked dependencies for the main project and nlp sub-project.
- Add build-essential and gcc, and install llama-cpp-python from its dedicated PyPI index.
- Replace Docker commands with Podman in CI workflow
