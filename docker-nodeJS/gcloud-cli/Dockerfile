FROM node:24-alpine

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=24

LABEL org.opencontainers.image.authors="ShadowZ11"
LABEL org.opencontainers.image.title="node-gcloud-cli"
LABEL org.opencontainers.image.description="Node.js image with Google Cloud CLI, kubectl, Helm and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/node-gcloud-cli"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

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
      python3 \
      tar \
      unzip \
      zip; \
    ln -sf python3 /usr/bin/python; \
    npm install -g pnpm vercel

ENV PATH="/opt/google-cloud-sdk/bin:$PATH"

RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) GOOGLE_ARCH=x86_64 ;; \
      aarch64|arm64) GOOGLE_ARCH=arm ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    curl -fsSLo google-cloud-cli.tar.gz "https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-${GOOGLE_ARCH}.tar.gz"; \
    tar -xzf google-cloud-cli.tar.gz -C /opt; \
    /opt/google-cloud-sdk/install.sh -q --command-completion true --path-update true --usage-reporting false; \
    rm google-cloud-cli.tar.gz

# kubectl, Helm and Terraform track upstream latest (see CLAUDE.md) but every
# download is checksum-verified against the publisher's own manifest.
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
    HELM_VERSION="$(curl -fsSL https://get.helm.sh/helm-latest-version)"; \
    HELM_TGZ="helm-${HELM_VERSION}-linux-${TOOL_ARCH}.tar.gz"; \
    curl -fsSLo "$HELM_TGZ" "https://get.helm.sh/${HELM_TGZ}"; \
    curl -fsSL "https://get.helm.sh/${HELM_TGZ}.sha256sum" | sha256sum -c -; \
    tar -xzf "$HELM_TGZ" --strip-components=1 -C /usr/local/bin "linux-${TOOL_ARCH}/helm"; \
    chmod 0755 /usr/local/bin/helm; \
    rm "$HELM_TGZ"; \
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
