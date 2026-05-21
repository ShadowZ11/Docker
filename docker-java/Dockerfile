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
      libffi-dev \
      maven \
      openssh-client \
      openssl \
      python3 \
      python3-dev \
      py3-pip \
      py3-virtualenv \
      py3-setuptools \
      unzip \
      zip; \
    ln -sf python3 /usr/bin/python; \
    python3 -m venv /opt/venv; \
    . /opt/venv/bin/activate; \
    pip install --no-cache-dir --upgrade pip; \
    pip install --no-cache-dir awsebcli awscli azure-cli

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