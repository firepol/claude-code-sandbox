FROM ubuntu:24.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    openssh-client \
    python3 \
    python3-pip \
    build-essential \
    sudo \
    vim \
    nano \
    jq \
    ca-certificates \
    gnupg \
    iproute2 \
    net-tools \
    netcat-openbsd \
    socat \
    tmux \
    screen \
    pv \
    fd-find \
    ripgrep \
    shellcheck \
    sqlite3 \
    zstd \
    lz4 \
    xz-utils \
    brotli \
    zip \
    unzip \
    rsync \
    && rm -rf /var/lib/apt/lists/*

# Configure passwordless sudo for ubuntu user
RUN echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ubuntu && \
    chmod 440 /etc/sudoers.d/ubuntu

# Create workspace directory and set ownership
RUN mkdir -p /workspace && \
    chown -R ubuntu:ubuntu /workspace

# Install Node.js 22.x
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs

# Install GitHub CLI
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt-get update \
    && apt-get install -y gh \
    && rm -rf /var/lib/apt/lists/*

# Install Claude Code
RUN npm install -g @anthropic-ai/claude-code@latest

# Add /home/ubuntu/.local/bin to system-wide bash config for all interactive shells
# This is sourced by /etc/profile for bash shells, so no need to add to /etc/profile separately
RUN echo '' >> /etc/bash.bashrc && \
    echo '# Add user local bin to PATH' >> /etc/bash.bashrc && \
    echo 'export PATH="/home/ubuntu/.local/bin:$PATH"' >> /etc/bash.bashrc

# Set up SSH config for ubuntu user
RUN mkdir -p /home/ubuntu/.ssh && \
    echo "Host github.com\n\
    StrictHostKeyChecking no\n\
    UserKnownHostsFile /dev/null" > /home/ubuntu/.ssh/config && \
    chmod 600 /home/ubuntu/.ssh/config && \
    chown -R ubuntu:ubuntu /home/ubuntu/.ssh

# Set git to use credential helper
RUN git config --global credential.helper 'cache --timeout=3600'

# Create entrypoint script
RUN echo '#!/bin/bash\n\
set -e\n\
\n\
# Copy SSH keys if mounted\n\
if [ -d "/tmp/.ssh" ]; then\n\
    cp -r /tmp/.ssh/* /home/ubuntu/.ssh/ 2>/dev/null || true\n\
    chmod 600 /home/ubuntu/.ssh/* 2>/dev/null || true\n\
    chown -R ubuntu:ubuntu /home/ubuntu/.ssh\n\
fi\n\
\n\
# Copy git config if mounted\n\
if [ -f "/tmp/.gitconfig" ]; then\n\
    cp /tmp/.gitconfig /home/ubuntu/.gitconfig\n\
    chown ubuntu:ubuntu /home/ubuntu/.gitconfig\n\
fi\n\
\n\
# Execute the command\n\
exec "$@"' > /entrypoint.sh && \
    chmod +x /entrypoint.sh

# Switch to ubuntu user
USER ubuntu
WORKDIR /workspace

ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]