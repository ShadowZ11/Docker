FROM node:20-alpine

# Installer les dépendances nécessaires
RUN apk update && \
    apk add --no-cache --virtual .build-deps \
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
    zip \
    && ln -sf python3 /usr/bin/python

RUN npm install -g npm@latest && \
    npm install -g pnpm vercel

# Assurez-vous que les commandes utilisent l'environnement virtuel
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /installer

# Installer la cli google cloud
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
      GOOGLE_ARCH="x86_64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
      GOOGLE_ARCH="arm"; \
    else \
      echo "Architecture non supportée: $ARCH"; exit 1; \
    fi && \
    curl -OL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-${GOOGLE_ARCH}.tar.gz && \
    tar -xvzf google-cloud-cli-linux-${GOOGLE_ARCH}.tar.gz && \
    ./google-cloud-sdk/install.sh -q --command-completion true --path-update true && \
    rm google-cloud-cli-linux-${GOOGLE_ARCH}.tar.gz

# Ajouter manuellement Google Cloud SDK au PATH
ENV PATH=$PATH:installer/google-cloud-sdk/bin

# Nettoyer l'environnement de travail temporaire
WORKDIR /

# Installer kubectl
RUN ARCH=$(uname -m) && \
        if [ "$ARCH" = "x86_64" ]; then \
          KUBECTL_ARCH="amd64"; \
        elif [ "$ARCH" = "aarch64" ]; then \
          KUBECTL_ARCH="arm64"; \
        else \
          KUBECTL_ARCH="amd64"; \
        fi && \
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/${KUBECTL_ARCH}/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Installer Terraform
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
      TERRAFORM_ARCH="amd64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
      TERRAFORM_ARCH="arm64"; \
    else \
      echo "Architecture non supportée: $ARCH"; exit 1; \
    fi && \
    release=$(curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d: -f2 | tr -d \"\,\v | awk '{$1=$1};1') && \
    curl "https://releases.hashicorp.com/terraform/${release}/terraform_${release}_linux_${TERRAFORM_ARCH}.zip" -o "terraform_${release}_linux_${TERRAFORM_ARCH}.zip" && \
    unzip terraform_${release}_linux_${TERRAFORM_ARCH}.zip && \
    mv terraform /usr/bin/terraform && \
    rm terraform_${release}_linux_${TERRAFORM_ARCH}.zip \
    # Nettoyer l'environnement de travail temporaire
&& rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

WORKDIR /app