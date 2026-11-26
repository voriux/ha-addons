ARG BUILD_FROM=ghcr.io/home-assistant/base-debian:trixie
FROM ${BUILD_FROM}

# Pinned: a Claude Code update ships as a new app version, never as a surprise on a running house.
ARG CLAUDE_VERSION=2.1.311
ARG TTYD_VERSION=1.7.7

ENV LANG=C.UTF-8 \
    DISABLE_AUTOUPDATER=1

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates curl git gh jq less nginx openssh-client procps python3 ripgrep \
    && rm -rf /var/lib/apt/lists/* \
    && useradd --uid 1000 --home-dir /data --no-create-home --shell /bin/bash claude

# ttyd is not packaged in trixie; static release binary, checked against the release checksums.
RUN arch="$(uname -m)" \
    && cd /tmp \
    && curl -fsSLO "https://github.com/tsl0922/ttyd/releases/download/${TTYD_VERSION}/ttyd.${arch}" \
    && curl -fsSL "https://github.com/tsl0922/ttyd/releases/download/${TTYD_VERSION}/SHA256SUMS" | grep " ttyd.${arch}$" | sha256sum -c - \
    && install -m 755 "ttyd.${arch}" /usr/local/bin/ttyd \
    && rm "ttyd.${arch}" \
    && ttyd --version

RUN curl -fsSL https://claude.ai/install.sh -o /tmp/install.sh \
    && HOME=/opt/claude bash /tmp/install.sh "${CLAUDE_VERSION}" \
    && chmod -R a+rX /opt/claude \
    && ln -s "$(readlink -f /opt/claude/.local/bin/claude)" /usr/local/bin/claude \
    && rm /tmp/install.sh \
    && claude --version

COPY rootfs /
