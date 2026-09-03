FROM node:24-alpine

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=24

LABEL org.opencontainers.image.title="node-aws-cli"
LABEL org.opencontainers.image.description="Node.js image with AWS CLI, kubectl and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/node-aws-cli"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

# AWS CLI v2 has no official musl build, so the Alpine variants stay on the pip
# packages. Toolchain packages are grouped in a virtual dep so they can be dropped
# once the wheels that need compiling on arm64 are built.
RUN set -eux; \
    apk add --no-cache \
      bash \
      ca-certificates \
      curl \
      git \
      gnupg \
      openssh-client \
      openssl \
      py3-pip \
      py3-virtualenv \
      python3 \
      tar \
      unzip \
      zip; \
    apk add --no-cache --virtual .build-deps \
      gcc \
      libffi-dev \
      musl-dev \
      openssl-dev \
      python3-dev; \
    ln -sf python3 /usr/bin/python; \
    npm install -g pnpm vercel; \
    python3 -m venv /opt/venv; \
    /opt/venv/bin/pip install --no-cache-dir --upgrade pip; \
    /opt/venv/bin/pip install --no-cache-dir awsebcli awscli; \
    apk del .build-deps

ENV PATH="/opt/venv/bin:$PATH"

# kubectl and Terraform track upstream latest (see CLAUDE.md) but every download
# is checksum-verified against the publisher's own manifest.
RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) TOOL_ARCH=amd64 ;; \
      aarch64|arm64) TOOL_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    KUBECTL_VERSION="$(curl -fsSL https://dl.k8s.io/release/stable.txt)"; \
    curl -fsSLo kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TOOL_ARCH}/kubectl"; \
    echo "$(curl -fsSL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TOOL_ARCH}/kubectl.sha256")  kubectl" | sha256sum -c -; \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl; \
    rm kubectl; \
    TERRAFORM_VERSION="$(curl -fsSL https://checkpoint-api.hashicorp.com/v1/check/terraform | sed -n 's/.*"current_version":"\([^"]*\)".*/\1/p')"; \
    test -n "$TERRAFORM_VERSION"; \
    TERRAFORM_ZIP="terraform_${TERRAFORM_VERSION}_linux_${TOOL_ARCH}.zip"; \
    curl -fsSLo "$TERRAFORM_ZIP" "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/${TERRAFORM_ZIP}"; \
    curl -fsSL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS" \
      | grep " ${TERRAFORM_ZIP}$" | sha256sum -c -; \
    unzip -q "$TERRAFORM_ZIP" terraform -d /usr/local/bin; \
    chmod 0755 /usr/local/bin/terraform; \
    rm "$TERRAFORM_ZIP"

WORKDIR /app
