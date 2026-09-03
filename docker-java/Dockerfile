# DEPRECATED - see README.md. Kept building only for backwards compatibility;
# the maintained replacement is docker-maven. Do not add functionality here.
FROM amazoncorretto:21-alpine3.19

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=21

LABEL org.opencontainers.image.title="java-aws-terraform"
LABEL org.opencontainers.image.description="Java tool image with Maven, AWS CLI, Azure CLI, kubectl, Helm and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/java-aws-terraform"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

RUN set -eux; \
    apk add --no-cache \
      bash \
      ca-certificates \
      curl \
      docker-cli \
      git \
      gnupg \
      maven \
      openssh-client \
      openssl \
      python3 \
      py3-pip \
      py3-virtualenv \
      py3-setuptools \
      tar \
      unzip \
      zip; \
    apk add --no-cache --virtual .build-deps \
      cargo \
      gcc \
      libffi-dev \
      linux-headers \
      musl-dev \
      openssl-dev \
      python3-dev; \
    ln -sf python3 /usr/bin/python; \
    python3 -m venv /opt/venv; \
    /opt/venv/bin/pip install --no-cache-dir --upgrade pip; \
    /opt/venv/bin/pip install --no-cache-dir awsebcli awscli azure-cli; \
    apk del .build-deps; \
    rm -rf /root/.cargo /root/.cache

ENV PATH="/opt/venv/bin:$PATH"

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
