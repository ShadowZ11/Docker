# Utilise l'image Docker officielle DinD comme base
FROM docker:29-dind

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=29

LABEL org.opencontainers.image.title="docker-dind"
LABEL org.opencontainers.image.description="Docker-in-Docker image with Azure CLI, AWS CLI and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/docker-dind"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

# cargo/gcc/musl-dev/linux-headers only exist to compile the wheels the Azure CLI
# needs on musl; keeping them in the final image would cost several hundred MB.
RUN set -eux; \
    apk add --no-cache \
      bash \
      ca-certificates \
      curl \
      git \
      jq \
      nodejs \
      npm \
      py3-pip \
      python3 \
      tar \
      zip \
      unzip; \
    apk add --no-cache --virtual .build-deps \
      cargo \
      gcc \
      libffi-dev \
      linux-headers \
      musl-dev \
      openssl-dev \
      python3-dev; \
    npm i -g yarn; \
    python3 -m venv /opt/venv; \
    /opt/venv/bin/pip install --no-cache-dir --upgrade pip; \
    /opt/venv/bin/pip install --no-cache-dir azure-cli awsebcli awscli; \
    apk del .build-deps; \
    rm -rf /root/.cargo /root/.cache /root/.npm

ENV PATH="/opt/venv/bin:$PATH"

# Terraform tracks upstream latest (see CLAUDE.md) but the download is
# checksum-verified against HashiCorp's own SHA256SUMS manifest.
RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) TOOL_ARCH=amd64 ;; \
      aarch64|arm64) TOOL_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    TERRAFORM_VERSION="$(curl -fsSL https://checkpoint-api.hashicorp.com/v1/check/terraform | sed -n 's/.*"current_version":"\([^"]*\)".*/\1/p')"; \
    test -n "$TERRAFORM_VERSION"; \
    TERRAFORM_ZIP="terraform_${TERRAFORM_VERSION}_linux_${TOOL_ARCH}.zip"; \
    curl -fsSLo "$TERRAFORM_ZIP" "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/${TERRAFORM_ZIP}"; \
    curl -fsSL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS" \
      | grep " ${TERRAFORM_ZIP}$" | sha256sum -c -; \
    unzip -q "$TERRAFORM_ZIP" terraform -d /usr/local/bin; \
    chmod 0755 /usr/local/bin/terraform; \
    rm "$TERRAFORM_ZIP"

# Lance le daemon Docker à l'intérieur du conteneur
ENTRYPOINT ["dockerd-entrypoint.sh"]
