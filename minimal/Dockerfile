# Stage 1: Builder
ARG BASE_IMAGE_TAG=latest
FROM rubylang/ruby:3.2.5-noble AS builder

ENV NB_USER=jovyan \
    NB_UID=1000 \
    NB_GID=100 \
    HOME=/home/jovyan

# Create jovyan user to match final image
RUN useradd -g ${NB_GID} -m ${NB_USER}

# Install build dependencies
RUN apt-get update && apt-get install -y -V --no-install-recommends \
    build-essential \
    cmake \
    gcc \
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
    && rm -rf /var/lib/apt/lists/*

RUN gem update --system

USER ${NB_USER}

# Set gem installation path for user
ENV GEM_HOME=/home/${NB_USER}/.local/share/gem/ruby/3.2.0 \
    PATH=/home/${NB_USER}/.local/share/gem/ruby/3.2.0/bin:$PATH

# Install gems as jovyan user
RUN gem install --no-document \
    irb \
    iruby \
    pycall \
    activesupport \
    awesome_print \
    bcrypt_pbkdf \
    bundler \
    childprocess \
    colorize \
    coltrane \
    debug \
    dry-initializer \
    dry-monads \
    ed25519 \
    eventmachine \
    ffi \
    fractional \
    geo_coord \
    hashie \
    highline \
    i18n \
    json \
    jsonl \
    kramdown \
    logging \
    minitest \
    mocha \
    multi_json \
    mysql2 \
    net-ssh \
    open3 \
    open4 \
    parallel \
    pastel \
    pg \
    pry \
    pry-doc \
    psych \
    rake \
    rdoc \
    redcarpet \
    rexml \
    rouge \
    rugged \
    ruport \
    sequel \
    sqlite3 \
    sync \
    sys-proctable \
    treetop \
    tty-box \
    tty-command \
    tty-config \
    tty-markdown \
    tty-prompt \
    tty-screen \
    yaml

# Stage 2: Final image
FROM quay.io/jupyter/scipy-notebook:${BASE_IMAGE_TAG}

USER root

ENV PATH=/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    HOME=/home/jovyan \
    CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=jovyan \
    NB_UID=1000 \
    NB_GID=100 \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    LANGUAGE=C.UTF-8

ARG GROUP=1000

# Install runtime dependencies only
RUN apt-get update && apt-get install -y -V --no-install-recommends \
    curl \
    exiftool \
    ghostscript \
    git \
    git-lfs \
    gnuplot \
    graphicsmagick \
    graphviz \
    libreoffice \
    libcairo2 \
    libczmq4 \
    libfftw3-bin \
    libgsl27 \
    libopenblas0 \
    libplot2c2 \
    libpoppler-glib8 \
    libsqlite3-0 \
    libzmq5 \
    libczmq-dev \
    liblink-grammar-dev \
    link-grammar \
    minisat \
    mysql-client \
    neovim \
    openjdk-17-jdk-headless \
    pandoc \
    pdftk \
    plotutils \
    poppler-utils \
    postgresql-client \
    python3-link-grammar \
    rsync \
    sqlite3 \
    tesseract-ocr \
    tidy \
    tzdata \
    zip \
    && rm -rf /var/lib/apt/lists/*

# Set up user and permissions
RUN groupadd -g $GROUP notebook && \
    usermod -a -G sudo $NB_USER && \
    usermod -a -G $GROUP $NB_USER && \
    echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/99-$NB_USER && \
    chmod 0440 /etc/sudoers.d/99-$NB_USER

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
  /usr/local/include/ruby-3.2.0/ \
  /usr/local/include/ruby-3.2.0/

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
  /home/${NB_USER}/.local/share/gem/ \
  /home/${NB_USER}/.local/share/gem/

WORKDIR /home/$NB_USER

# Configure gem environment
RUN rm -f /home/$NB_USER/.wget-hsts && \
    echo "gem: --user-install" >> /etc/gemrc && \
    echo "gem: --no-user-install --no-document" >> /root/.gemrc

USER $NB_UID

RUN mamba update --all --yes && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

RUN echo "[[ -f ~/.env ]] && source ~/.env" >> $HOME/.bashrc && \
    echo "gem: --user-install --no-document" >> $HOME/.gemrc

ENV PATH="/home/${NB_USER}/.local/share/gem/ruby/3.2.0/bin:$HOME/.local/bin:$PATH"
ENV GEM_HOME="/home/${NB_USER}/.local/share/gem/ruby/3.2.0"

RUN iruby register --force
