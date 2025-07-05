# syntax=docker/dockerfile:1.4
# Use a specific, stable version for the base image if possible.
# If the base image is frequently updated, consider using a digest for pinning.
FROM khulnasoft/build-environment:0.6.18

# --- Dependency Versioning ---
# These versions are quite old as of July 2025.
# - Java 15: Released Sep 2020, EOL Mar 2021 (non-LTS). Current LTS is Java 17, next is Java 21.
# - Docker Compose 1.28.5: Released Feb 2021. Docker Compose V2 is standard now (plugin for Docker CLI).
# - Kind 0.9.0: Released Feb 2021. Current is around 0.23.0.
# - Kubectl 1.18.12: Released Oct 2020. Current is around 1.30.0.
# - Node 16.x: Released Apr 2021, EOL Sep 2023. Current LTS is Node 20.x, next is Node 22.x.
# - Yarn 1: Legacy Yarn Classic. Yarn Berry (v2+) is the modern version.

# --- General Best Practices ---
# 1. Consolidate RUN commands for related installations to minimize layers.
# 2. Always clean apt caches immediately after `apt-get install` in the same RUN command.
# 3. Use `\ ` for line continuations for readability.
# 4. Pin versions for reproducibility, but update to *current stable* versions.
# 5. Use `set -eux` in scripts for early exit on errors and debugging.

# --- Improved Dockerfile ---

# Install Java (updated to OpenJDK 17 LTS as of July 2025)
ENV JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
RUN set -eux; \
    add-apt-repository ppa:openjdk-r/ppa -y; \
    apt-get update; \
    apt-get install -y openjdk-17-jdk; \
    dpkg-query -l openjdk-17-jdk; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*;

# Install docker-compose (V2, which is now the standard)
# V2 is installed as a Docker CLI plugin, usually found at ~/.docker/cli-plugins/docker-compose
ENV DOCKER_COMPOSE_VERSION="v2.24.5" # Check latest stable from https://github.com/docker/compose/releases
RUN set -eux; \
    DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}; \
    mkdir -p $DOCKER_CONFIG/cli-plugins; \
    curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" \
        -o $DOCKER_CONFIG/cli-plugins/docker-compose; \
    chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose; \
    # Verify installation (optional, but good for CI)
    docker compose version; \
    # No apt list cleanup needed here as it's a curl download

# Install kind for Kubernetes testing (updated to a recent stable version)
ENV KIND_VERSION="v0.23.0" # Check latest stable from https://github.com/kubernetes-sigs/kind/releases
RUN set -eux; \
    curl -Lo /usr/local/bin/kind "https://kind.sigs.k8s.io/dl/${KIND_VERSION}/kind-linux-amd64"; \
    chmod +x /usr/local/bin/kind; \
    # Verify installation
    kind version;

COPY kind-config.yaml /kind-config.yaml

# Install kubectl (updated to a recent stable version, matching Kind if possible or a common version like 1.30.x)
# Always try to match kubectl version with the Kubernetes cluster version (kind) if possible.
# For simplicity, let's pick a recent stable client.
ENV KUBECTL_VERSION="v1.30.2" # Check latest stable from https://kubernetes.io/releases/
RUN set -eux; \
    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"; \
    chmod +x ./kubectl; \
    mv ./kubectl /usr/local/bin/kubectl; \
    # Verify installation
    kubectl version --client;

# Install libpq-dev required for psycopg2
# This seems to be a common requirement. Ensure it's not already in the base image.
RUN set -eux; \
    apt-get update; \
    apt-get install -y libpq-dev; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*;

# Update node and yarn version (updated to Node.js 20.x LTS)
# Note: npm install -g yarn@1 installs Yarn Classic. If you need Yarn Berry (v2+), the process is different.
# Assuming Yarn Classic (v1) is still desired given `yarn@1`.
ENV NODE_MAJOR_VERSION="20" # Node.js 20.x LTS as of July 2025
RUN set -eux; \
    apt-get update; \
    curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR_VERSION}.x | bash -; \
    apt-get install -y --no-install-recommends nodejs; \
    npm install -g yarn@1; \
    # Verify installations
    node -v; \
    npm -v; \
    yarn -v; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*;

COPY extended-entrypoint.sh /extended-entrypoint.sh

RUN chmod +x /extended-entrypoint.sh

# ENTRYPOINT: Ensure `/tini` is present in the base image or installed here.
# The `khulnasoft/build-environment` base image likely includes `tini`.
ENTRYPOINT ["/tini", "-g", "--", "/extended-entrypoint.sh"]
