# Copyright (c) Jupyter Development Team.
ARG BASE_IMAGE_TAG=latest
FROM quay.io/jupyter/base-notebook:$BASE_IMAGE_TAG
LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Install all OS dependencies for the Server that starts
# but lacks all features (e.g., download as all possible file formats)
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    # - Add necessary fonts for matplotlib/seaborn
    #   See https://github.com/jupyter/docker-stacks/pull/380 for details
    fonts-liberation \
    libyaml-dev \
    # - `pandoc` is used to convert notebooks to html files
    #   it's not present in the aarch64 Ubuntu image, so we install it here
    pandoc \
    # - `run-one` - a wrapper script that runs no more
    #   than one unique instance of some command with a unique set of arguments,
    #   we use `run-one-constantly` to support the `RESTARTABLE` option
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

USER ${NB_UID}

# Install JupyterHub, JupyterLab, NBClassic and Jupyter Notebook
# Generate a Jupyter Server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
WORKDIR /tmp
RUN mamba clean --all -f -y && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    pip install --no-cache-dir \
    'spacy' 'google-generativeai' 'google-adk' 'openai' 'chromadb' 'Markdown' 'pyyaml' \
    --extra-index-url https://download.pytorch.org/whl/cpu && \
    python3 -m spacy download en_core_web_lg --extra-index-url https://download.pytorch.org/whl/cpu && \
    python3 -m spacy download en_core_web_sm --extra-index-url https://download.pytorch.org/whl/cpu && \
    python -c "import sys, importlib.util as util; 1 if util.find_spec('nltk') else sys.exit(); import nltk; nltk.download('punkt')" && \
    fix-permissions "/home/${NB_USER}"

ENV JUPYTER_PORT=8888
EXPOSE $JUPYTER_PORT

# Configure container startup
CMD ["start-notebook.py"]

# Copy local files as late as possible to avoid cache busting
COPY base/start-notebook.py base/start-notebook.sh base/start-singleuser.py base/start-singleuser.sh /usr/local/bin/
COPY base/jupyter_server_config.py base/docker_healthcheck.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root
RUN fix-permissions /etc/jupyter/

# HEALTHCHECK documentation: https://docs.docker.com/engine/reference/builder/#healthcheck
# This healtcheck works well for `lab`, `notebook`, `nbclassic`, `server`, and `retro` jupyter commands
# https://github.com/jupyter/docker-stacks/issues/915#issuecomment-1068528799
HEALTHCHECK --interval=3s --timeout=1s --start-period=3s --retries=3 \
    CMD /etc/jupyter/docker_healthcheck.py || exit 1

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}

WORKDIR "${HOME}"