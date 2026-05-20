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
      apt-transport-https \
      bash \
      ca-certificates \
      curl \
      git \
      gnupg \
      gnupg-agent \
      jq \
      lsb-release \
      openssh-client \
      openssl \
      python3 \
      python3-dev \
      python3-pip \
      python3-pkg-resources \
      python3-setuptools \
      python3-venv \
      software-properties-common \
      unzip; \
    ln -sf python3 /usr/bin/python; \
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
      | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list; \
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
      | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg; \
    apt-get update; \
    apt-get install -y --no-install-recommends google-cloud-cli; \
    apt-get purge -y --auto-remove software-properties-common; \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /tmp/docker.gpg; \
    gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg /tmp/docker.gpg; \
    rm /tmp/docker.gpg; \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null; \
    apt-get update; \
    apt-get install -y --no-install-recommends docker-ce-cli; \
    python3 -m venv /opt/venv; \
    . /opt/venv/bin/activate; \
    pip install --no-cache-dir --upgrade pip setuptools wheel; \
    pip install --no-cache-dir "awscli==1.29.85"; \
    pip install --no-cache-dir "awsebcli==3.27.2"; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

ENV PATH="/opt/venv/bin:$PATH"

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
    TERRAFORM_VERSION="$(curl -fsSL https://releases.hashicorp.com/terraform/ | grep -oE 'terraform/[0-9]+\.[0-9]+\.[0-9]+/' | cut -d/ -f2 | sort -Vr | head -n1)"; \
    curl -fsSLo terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TERRAFORM_ARCH}.zip"; \
    unzip terraform.zip; \
    mv terraform /usr/local/bin/terraform; \
    rm terraform.zip

WORKDIR /app