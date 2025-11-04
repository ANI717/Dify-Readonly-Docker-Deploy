FROM golang:1.23-alpine AS builder

ARG VERSION=unknown

# copy project
COPY . /app

# set working directory
WORKDIR /app

# using goproxy if you have network issues
# ENV GOPROXY=https://goproxy.cn,direct

# build
RUN go build \
    -ldflags "\
    -X 'github.com/langgenius/dify-plugin-daemon/internal/manifest.VersionX=${VERSION}' \
    -X 'github.com/langgenius/dify-plugin-daemon/internal/manifest.BuildTimeX=$(date -u +%Y-%m-%dT%H:%M:%S%z)'" \
    -o /app/main cmd/server/main.go

FROM python:3.12-slim-bookworm

WORKDIR /app

# check build args
ARG PLATFORM=local

# Install additional packages needed for the application
RUN set -ex && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y curl \
        python3-dev python3-pip ffmpeg \
        build-essential git \
        cmake pkg-config \
        libcairo2-dev libjpeg-dev libgif-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# General environment variables (set BEFORE installing uv)
ENV HOME=/tmp
ENV TMPDIR=/tmp
ENV XDG_CACHE_HOME=/tmp/.cache

# UV (package installer) configuration (set BEFORE installing uv)
ENV UV_PATH=/usr/local/bin/uv
ENV UV_CACHE_DIR=/tmp/uv-cache
ENV UV_TOOL_DIR=/tmp/uv-tools
ENV UV_PYTHON_INSTALL_DIR=/tmp/uv-python
ENV UV_PROJECT_ENVIRONMENT=/tmp/python-packages

# Install dify_plugin to speedup the environment setup, test uv and preload tiktoken
RUN set -ex && \
    rm -f /usr/lib/python*/EXTERNALLY-MANAGED && \
    python3 -m pip install uv && \
    uv pip install --system dify_plugin && \
    python3 -c "from uv._find_uv import find_uv_bin;print(find_uv_bin());" && \
    python3 -c "import tiktoken; encodings = ['o200k_base', 'cl100k_base', 'p50k_base', 'r50k_base', 'p50k_edit', 'gpt2']; [tiktoken.get_encoding(encoding).special_tokens_set for encoding in encodings]"

# Pip configuration
ENV PIP_USER=1
ENV PIP_CACHE_DIR=/tmp/pip-cache
ENV PYTHONUSERBASE=/tmp/python-packages
# Include both possible site-packages locations in PYTHONPATH
ENV PYTHONPATH=/tmp/python-packages/lib/python3.12/site-packages:/tmp/python-packages/lib/python/site-packages
# Add user package binaries to PATH
ENV PATH=/tmp/python-packages/bin:$PATH

# Application configuration
ENV PLATFORM=$PLATFORM
ENV GIN_MODE=release
ENV PLUGIN_REMOTE_INSTALLING_ENABLED=true
ENV PLUGIN_REMOTE_INSTALLING_HOST=0.0.0.0
ENV PLUGIN_REMOTE_INSTALLING_PORT=5003
ENV PLUGIN_STORAGE_TYPE=local
ENV PLUGIN_STORAGE_LOCAL_ROOT=/tmp

# Plugin configuration for local platform (paths are relative to PLUGIN_STORAGE_LOCAL_ROOT for OSS)
ENV PLUGIN_WORKING_PATH=/tmp/plugins
ENV PLUGIN_INSTALLED_PATH=plugins/installed
ENV PLUGIN_PACKAGE_CACHE_PATH=plugins/packages
ENV PLUGIN_MEDIA_CACHE_PATH=media-cache

# Persistence storage configuration (relative to PLUGIN_STORAGE_LOCAL_ROOT)
ENV PERSISTENCE_STORAGE_PATH=persistence

# Create a non-root user
RUN groupadd -r dify && useradd -r -g dify -u 1001 dify

# Create necessary directories in tmp for write access
RUN mkdir -p /tmp/.tiktoken /tmp/storage /tmp/logs /tmp/python-packages \
    /tmp/.cache /tmp/uv-cache /tmp/uv-tools /tmp/uv-python /tmp/pip-cache \
    /tmp/plugins /tmp/plugins/installed /tmp/plugins/packages \
    /tmp/persistence /tmp/media-cache && \
    chmod -R 755 /tmp/.tiktoken /tmp/storage /tmp/logs /tmp/python-packages  \
    /tmp/.cache /tmp/uv-cache /tmp/uv-tools /tmp/uv-python /tmp/pip-cache \
    /tmp/plugins /tmp/plugins/installed /tmp/plugins/packages \
    /tmp/persistence /tmp/media-cache

COPY --from=builder /app/main /app/entrypoint.sh /app/

# Set permissions and ownership of the application files and directories to the dify user
RUN chmod +x /app/main /app/entrypoint.sh && \
    chown -R dify:dify /app /tmp/.tiktoken /tmp/storage /tmp/logs /tmp/python-packages \
    /tmp/.cache /tmp/uv-cache /tmp/uv-tools /tmp/uv-python /tmp/pip-cache \
    /tmp/plugins /tmp/plugins/installed /tmp/plugins/packages \
    /tmp/persistence /tmp/media-cache

# Change working directory to /tmp so the app creates storage there
WORKDIR /tmp

# Switch to non-root user
USER dify

# run the server, using sh as the entrypoint to avoid process being the root process
# and using bash to recycle resources
CMD ["/bin/bash", "-c", "/app/entrypoint.sh"]
