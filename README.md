# Jupyter Notebook with Ruby Kernel & NLP Gems

[![CI](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml/badge.svg)](https://github.com/b08x/jupyter-ruby-docker/actions/workflows/ci.yml)

Based on RubyData's [docker-stacks](https://github.com/RubyData/docker-stacks) and the official [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks).

<div style="float: left; margin-right: 20px;">
  <img src="docs/assets/img/info_graphic.webp" alt="Process Diagram" width="500" height="500">
</div>
This project provides Docker images that bundle a Jupyter Notebook server preconfigured with a Ruby kernel (IRuby) and a comprehensive suite of RubyGems focused on Natural Language Processing (NLP) and language model utilities.

## Features

- **Ruby Kernel for Jupyter**: Use IRuby to run Ruby code interactively within Jupyter notebooks.
- **NLP Libraries**: Includes gems for text tokenization, sentiment analysis, entity extraction, and various language utilities.
- **Cross-Language Integration**: Leverage Python libraries (like spaCy, NLTK, Transformers) from Ruby via [pycall](https://github.com/mrkn/pycall.rb) for mixed-language workflows.
- **Data Analysis**: Utilize gems like [daru](https://github.com/SciRuby/daru) and [daru-view](https://github.com/SciRuby/daru-view) for data manipulation and visualization.
- **Vector Database Integration**: Includes `pgvector`, `chroma-db` and `redis` services via Docker Compose for vector similarity searches.
- **TTY Gem Suite**: Includes components from the [TTY-Toolkit](https://ttytoolkit.org/) to create great Terminal Apps.

## Prerequisites

- [Docker](https://www.docker.com/get-started) installed on your machine.
- [Docker Compose](https://docs.docker.com/compose/install/) (Recommended for full functionality).
- Basic familiarity with Jupyter Notebooks and Docker commands.

## Building the Images

The project contains two primary Docker images: `base` and `nlp`.

- **[base](./base/README.md)**: A foundational image built upon `jupyter/docker-stacks-foundation`, including JupyterLab, common Python data science libraries, and essential NLP tools like spaCy and Google Generative AI SDK.
- **[nlp](./nlp/README.md)**: Built upon the `base` image, this adds Ruby, IRuby kernel, and a wide range of Ruby NLP gems.

1. **Clone the Repository**

    ```bash
    git clone https://github.com/b08x/jupyter-ruby-docker.git
    cd jupyter-ruby-docker
    ```

2. **Build the Images**

    You can build the images individually using `docker build` or use the provided `Rakefile`:

    - **Using Docker Build (NLP image, depends on base):**

        ```bash
        # Build the base image first (if not pulled from a registry)
        docker build -t b08x/notebook-base:latest -f base/Dockerfile .

        # Build the NLP image
        docker build -t b08x/notebook-nlp:latest -f nlp/Dockerfile .
        ```

    - **Using Rake (Recommended):**

        ```bash
        # Install rake if not already installed
        bundle install
        
        # Build the nlp image (will build base automatically if needed)
        rake build/nlp

        # Or build all defined images
        rake build-all
        ```

## Running the Container

You can run the `nlp` image using either `docker run` or `docker-compose`. Using `docker-compose` is recommended as it also sets up the `redis` and `pgvector` services defined in `docker-compose.yml`.

### Using `docker run` (Notebook only)

This method starts only the Jupyter notebook container.

```bash
docker run --rm -p 8888:8888 \
  -v "${PWD}/work":/home/jovyan/work \
  --user $(id -u):$(id -g) \
  b08x/notebook-nlp:latest
```

**Explanation:**

- `--rm`: Automatically remove the container when it exits.
- `-p 8888:8888`: Map port 8888 on your host to port 8888 in the container.
- `-v "${PWD}/work":/home/jovyan/work`: Mount a local directory named `work` into the container's home directory. **Create this directory (`mkdir work`) in your project folder first.** This allows you to persist your notebooks and data.
- `--user $(id -u):$(id -g)`: Run the container using your current user and group ID to avoid permission issues with mounted volumes.
- `b08x/notebook-nlp:latest`: The image to run.

**Access Jupyter:** Open your browser and go to `http://localhost:8888` (or `http://<your-docker-ip>:8888`). The startup logs in your terminal will display an authentication token—use it to log in.

### Using `docker-compose` (Recommended: Notebook + Redis + Pgvector)

This method uses the `docker-compose.yml` file to start the `nlp-notebook`, `redis`, and `pgvector` services together.

1. **Environment Setup (`.env` file):**
    Copy the example environment file (`.env.example`) to `.env` and configure the variables:

    ```bash
    cp .env.example .env
    ```

    Edit the `.env` file to set the following:
    - **`UID` & `GID`**: (Optional) Set these to your user and group ID (`id -u` and `id -g`) if they are not 1000. This ensures correct file permissions for mounted volumes.
    - **`WORKSPACE`**: (Optional) Set this to the **absolute path** of the directory on your host machine where you want to store your notebooks and work files. If left empty or undefined, the `docker-compose.yml` will default to mounting a `./data` directory (relative to the project root) into the container at `/home/jovyan/work`.
    - **API Keys**: (Optional) Set `OPENAI_API_KEY`, `GOOGLE_PALM_API_KEY`, and `HUGGINGFACE_API_KEY` if you plan to use services requiring them within your notebooks.

2. **Create Necessary Directories:**
    Depending on your `.env` configuration and the `docker-compose.yml` settings, ensure the necessary host directories exist before starting the services.
    - **If `WORKSPACE` is set in `.env`:** Make sure the directory specified by `WORKSPACE` exists.
      ```bash
      # Example if WORKSPACE=/path/to/my/notebooks
      mkdir -p /path/to/my/notebooks
      ```
    - **If `WORKSPACE` is *not* set in `.env`:** Create the default `./data` directory.
      ```bash
      mkdir -p ./data
      ```
    - **For the read-only Library mount:** The default `docker-compose.yml` also mounts `$HOME/Documents` as read-only to `/home/jovyan/Library`. Ensure this directory exists if you intend to use it.
      ```bash
      mkdir -p ~/Documents
      ```

3. **Start Services:**

    ```bash
    docker-compose up -d
    ```

    - `-d`: Run containers in detached mode (in the background).

4. **Access Jupyter:**
    Open your browser to `http://localhost:8888`. Use the token from the logs (`docker-compose logs nlp-notebook`).

5. **Access RedisInsight:**
    Open your browser to `http://localhost:8001`.

6. **Stopping Services:**

    ```bash
    docker-compose down
    ```

## Overview of Included RubyGems

For a complete list, review the [nlp/README](./nlp/README.md).

## Customization & Development

You can customize the Docker images in several ways:

- **Ruby Gems:** To add or update Ruby gems, modify the `nlp/Gemfile`. Then, rebuild the `nlp` image using `rake build/nlp` or `docker build -f nlp/Dockerfile .`.
- **Python Packages:** To add or update Python packages, edit the `pip install` or `mamba install` commands within the `base/Dockerfile`. After making changes, you'll need to rebuild both the `base` and `nlp` images.
- **System Packages:** If you need additional system libraries, add the necessary `apt-get install` commands to the appropriate Dockerfile:
  - Use `base/Dockerfile` for dependencies needed by Python or the base environment.
  - Use `nlp/Dockerfile` for dependencies needed specifically for Ruby or at runtime.
    Remember to rebuild the relevant image(s) after adding packages.
- **Jupyter Configuration:** Modify Jupyter settings by editing `base/jupyter_server_config.py` or the startup scripts (like `base/start-notebook.py`). Rebuild the images after making configuration changes.
- **Kernel Patching:** The `nlp/Dockerfile` includes a step to apply `respond_to_missing.patch` to the `ruby-spacy` gem. You can customize this patch file if needed and rebuild the `nlp` image.

## Troubleshooting

- **Build Issues**: Verify build dependencies and Docker daemon resources. Check build logs for errors.
- **Kernel Registration**: If the Ruby kernel doesn't appear, ensure `iruby register --force` ran successfully during the `nlp` image build (`docker logs <container_id>` or check build output).
- **Port Conflicts**: If port 8888 (or others like 6379, 5432) is in use, modify the `docker run -p` mapping or the `ports` section in `docker-compose.yml`.
- **Volume Permissions**: If you encounter permission errors with mounted volumes, ensure the `--user $(id -u):$(id -g)` flag is used with `docker run` or that the `UID`/`GID` in your `.env` file matches your host user when using `docker-compose`.

## License

This project is licensed under the MIT License. See the LICENSE file (if available) or assume MIT if missing. Base images used may have their own licenses (e.g., BSD for Jupyter Docker Stacks).

## Contributing

Contributions are welcome! Please fork the repository and submit pull requests.
