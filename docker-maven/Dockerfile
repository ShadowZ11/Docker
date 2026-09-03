FROM maven:3-eclipse-temurin-21

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=21

LABEL authors="Manub"
LABEL org.opencontainers.image.title="maven-terraform"
LABEL org.opencontainers.image.description="Maven tool image with AWS CLI, Azure CLI, Google Cloud CLI, kubectl, Helm and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/maven-terraform"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      bash \
      ca-certificates \
      curl \
      git \
      gnupg \
      jq \
      openssh-client \
      openssl \
      python3 \
      python3-venv \
      unzip; \
    ln -sf python3 /usr/bin/python; \
    DEB_ARCH="$(dpkg --print-architecture)"; \
    . /etc/os-release; \
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
      | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg; \
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
      > /etc/apt/sources.list.d/google-cloud-sdk.list; \
    curl -fsSL https://packages.microsoft.com/keys/microsoft.asc \
      | gpg --dearmor -o /usr/share/keyrings/microsoft.gpg; \
    echo "deb [arch=${DEB_ARCH} signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/azure-cli/ ${VERSION_CODENAME} main" \
      > /etc/apt/sources.list.d/azure-cli.list; \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
      | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg; \
    echo "deb [arch=${DEB_ARCH} signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu ${VERSION_CODENAME} stable" \
      > /etc/apt/sources.list.d/docker.list; \
    apt-get update; \
    apt-get install -y --no-install-recommends azure-cli docker-ce-cli google-cloud-cli; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# AWS CLI v2 ships as a self-contained bundle; v1 (pip) is in maintenance mode only.
# awsebcli has no v2 equivalent and stays in an isolated venv so its pinned
# botocore cannot conflict with the CLI.
RUN set -eux; \
    curl -fsSLo awscliv2.zip "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip"; \
    unzip -q awscliv2.zip; \
    ./aws/install; \
    rm -rf awscliv2.zip aws; \
    python3 -m venv /opt/awsebcli; \
    /opt/awsebcli/bin/pip install --no-cache-dir --upgrade pip setuptools wheel; \
    /opt/awsebcli/bin/pip install --no-cache-dir awsebcli; \
    ln -s /opt/awsebcli/bin/eb /usr/local/bin/eb

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
