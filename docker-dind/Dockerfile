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

RUN set -eux; \
    apk add --no-cache \
      bash \
      cargo \
      ca-certificates \
      curl \
      gcc \
      git \
      jq \
      linux-headers \
      musl-dev \
      nodejs \
      npm \
      py3-pip \
      python3-dev \
      zip; \
    npm i -g yarn; \
    python3 -m venv /opt/venv; \
    . /opt/venv/bin/activate; \
    pip install --no-cache-dir --upgrade pip; \
    pip install --no-cache-dir azure-cli awsebcli awscli

ENV PATH="/opt/venv/bin:$PATH"

RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) TERRAFORM_ARCH=amd64 ;; \
      aarch64|arm64) TERRAFORM_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    TERRAFORM_VERSION="$(curl -fsSL https://api.github.com/repos/hashicorp/terraform/releases/latest | grep '"tag_name":' | head -n1 | cut -d '"' -f4)"; \
    curl -fsSLo terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip"; \
    unzip terraform.zip; \
    mv terraform /usr/local/bin/terraform; \
    rm terraform.zip && true

# Vérifie que tout est installé
RUN docker --version && az --version && aws --version && node --version

# Lance le daemon Docker à l'intérieur du conteneur
ENTRYPOINT ["dockerd-entrypoint.sh"]
