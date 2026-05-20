FROM node:20-alpine

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=20

LABEL org.opencontainers.image.title="node-gcloud-cli"
LABEL org.opencontainers.image.description="Node.js image with Google Cloud CLI, kubectl and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/node-gcloud-cli"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

RUN set -eux; \
    apk add --no-cache \
      bash \
      ca-certificates \
      cargo \
      curl \
      gcc \
      git \
      gnupg \
      libffi-dev \
      linux-headers \
      musl-dev \
      openssh-client \
      openssl \
      py3-pip \
      py3-virtualenv \
      python3 \
      python3-dev \
      unzip \
      zip; \
    ln -sf python3 /usr/bin/python; \
    npm install -g npm@latest pnpm vercel; \
    python3 -m venv /opt/venv; \
    . /opt/venv/bin/activate; \
    pip install --no-cache-dir --upgrade pip

ENV PATH="/opt/venv/bin:/opt/google-cloud-sdk/bin:$PATH"

WORKDIR /opt

RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) GOOGLE_ARCH=x86_64 ;; \
      aarch64|arm64) GOOGLE_ARCH=arm ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    curl -fsSLo google-cloud-cli.tar.gz "https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-${GOOGLE_ARCH}.tar.gz"; \
    tar -xzf google-cloud-cli.tar.gz; \
    mv google-cloud-sdk /opt/google-cloud-sdk; \
    /opt/google-cloud-sdk/install.sh -q --command-completion true --path-update true --usage-reporting false; \
    rm google-cloud-cli.tar.gz

RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) KUBECTL_ARCH=amd64; TERRAFORM_ARCH=amd64 ;; \
      aarch64|arm64) KUBECTL_ARCH=arm64; TERRAFORM_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    KUBECTL_VERSION="$(curl -fsSL https://dl.k8s.io/release/stable.txt)"; \
    curl -fsSLo kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${KUBECTL_ARCH}/kubectl"; \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl; \
    rm kubectl; \
    curl -fsSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash; \
    TERRAFORM_VERSION="$(curl -fsSL https://api.github.com/repos/hashicorp/terraform/releases/latest | grep '"tag_name":' | head -n1 | cut -d '"' -f4)"; \
    curl -fsSLo terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip"; \
    unzip terraform.zip; \
    mv terraform /usr/local/bin/terraform; \
    rm terraform.zip; \
    rm -rf /var/cache/apk/* /tmp/* /var/tmp/* && true

WORKDIR /app