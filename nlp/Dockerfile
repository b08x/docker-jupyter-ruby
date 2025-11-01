ARG REGISTRY=docker.io
ARG OWNER=b08x
ARG BASE_IMAGE=$REGISTRY/$OWNER/notebook-base
ARG BASE_IMAGE_TAG=latest

# Stage 1: Builder
FROM rubylang/ruby:3.3.8-jammy AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y -V --no-install-recommends \
  build-essential \
  cmake \
  gcc \
  git \
  git-lfs \
  link-grammar \
  liblink-grammar-dev \
  libcairo2-dev \
  libczmq-dev \
  libffi-dev \
  libfftw3-dev \
  libgdbm-dev \
  libgirepository1.0-dev \
  libglib2.0-dev \
  libgmp-dev \
  libgsl-dev \
  liblapacke-dev \
  libmagick++-dev \
  libmysqlclient-dev \
  libncurses5-dev \
  libopenblas-dev \
  libpoppler-glib-dev \
  libpq-dev \
  libreadline-dev \
  libsqlite3-dev \
  libssl-dev \
  libxml2-dev \
  libxslt1-dev \
  libyaml-dev \
  libzmq3-dev \
  zlib1g-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

RUN gem update --system

WORKDIR /root

COPY nlp/Gemfile .

RUN export GEM_HOME="${HOME}/.local/share/gem/ruby/3.3.0" && \
  export PATH="${HOME}/.local/share/gem/ruby/3.3.0/bin:${PATH}" && \
  bundle lock --add-platform x86_64-linux && \
  bundle config build.redic --with-cxx="clang++" --with-cflags="-std=c++0x" && \
  bundle install

COPY nlp/respond_to_missing.patch .

RUN cd ${HOME}/.local/share/gem/ruby/3.3.0/gems/ruby-spacy-0.2.3 && \
  patch -Np1 -i "${HOME}/respond_to_missing.patch"

# Stage 2: Final image
FROM ${BASE_IMAGE}:${BASE_IMAGE_TAG}

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# `nbconvert` dependencies
# https://nbconvert.readthedocs.io/en/latest/install.html#installing-tex

# Install runtime dependencies only
RUN apt-get update && apt-get install -y -V --no-install-recommends \
  curl \
  exiftool \
  fzf \
  ghostscript \
  git \
  git-lfs \
  gnuplot \
  graphicsmagick \
  graphviz \
  less \
  link-grammar \
  libzmq3-dev \
  minisat \
  mysql-client \
  neovim \
  openjdk-17-jdk-headless \
  openssh-client \
  pandoc \
  pdftk \
  plotutils \
  poppler-utils \
  postgresql-client \
  python3-link-grammar \
  ripgrep \
  rsync \
  sqlite3 \
  tesseract-ocr \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-plain-generic \
  tidy \
  tzdata \
  xclip \
  zip && \
  apt-get clean && rm -rf /var/lib/apt/lists/*

# Set up user and permissions
RUN groupadd -g 1000 notebook && \
  usermod -a -G sudo ${NB_USER} && \
  usermod -a -G notebook ${NB_USER} && \
  echo "${NB_USER} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/99-${NB_USER} && \
  chmod 0440 /etc/sudoers.d/99-${NB_USER}

# Copy Ruby from builder
COPY --from=builder \
  /usr/local/bin/bundle \
  /usr/local/bin/bundler \
  /usr/local/bin/erb \
  /usr/local/bin/gem \
  /usr/local/bin/irb \
  /usr/local/bin/racc \
  /usr/local/bin/rake \
  /usr/local/bin/rdoc \
  /usr/local/bin/ri \
  /usr/local/bin/ruby \
  /usr/local/bin/

COPY --from=builder \
  /usr/local/include/ruby-3.3.0/ \
  /usr/local/include/ruby-3.3.0/

COPY --from=builder \
  /usr/local/lib/libruby.so* \
  /usr/local/lib/

COPY --from=builder \
  /usr/local/lib/pkgconfig/ \
  /usr/local/lib/pkgconfig/

COPY --from=builder \
  /usr/local/lib/ruby/ \
  /usr/local/lib/ruby/

COPY --from=builder \
  /usr/local/share/man/man1/erb.1 \
  /usr/local/share/man/man1/irb.1 \
  /usr/local/share/man/man1/ri.1 \
  /usr/local/share/man/man1/ruby.1 \
  /usr/local/share/man/man1/

# Copy installed gems from builder
COPY --from=builder --chown=${NB_UID}:${NB_GID} \
  /root/.local/share/gem/ \
  /home/${NB_USER}/.local/share/gem/

COPY --from=builder --chown=${NB_UID}:${NB_GID} \
  /root/Gemfile \
  /root/Gemfile.lock \
  /home/${NB_USER}/

USER ${NB_UID}

WORKDIR "${HOME}"

ENV PATH="/home/${NB_USER}/.local/share/gem/ruby/3.3.0/bin:${HOME}/.local/bin:${PATH}"
ENV GEM_HOME="/home/${NB_USER}/.local/share/gem/ruby/3.3.0"

RUN echo "[[ -f ~/.env ]] && source ~/.env" >> ${HOME}/.bashrc && \
  echo "gem: --user-install --no-document" >> ${HOME}/.gemrc

# To fix: qt.qpa.xcb could not connect to display
ENV QT_QPA_PLATFORM=offscreen

RUN iruby register --force