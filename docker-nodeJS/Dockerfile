FROM node:20-alpine3.20

# Installer les dépendances nécessaires
RUN apk update && \
    apk add --no-cache \
    ca-certificates \
    curl \
    gnupg \
    docker-cli \
    openssl \
    unzip \
    git \
    openssh-client \
    musl-dev \
    libffi-dev \
    gcc \
    python3-dev \
    linux-headers \
    cargo \
    bash

RUN apk add --no-cache zip

RUN apk add --update --no-cache python3 py3-pip py3-virtualenv && ln -sf python3 /usr/bin/python

RUN python3 -m venv /opt/venv

RUN npm install -g npm@latest && \
    npm install -g pnpm vercel firebase-tools

# Activer l'environnement virtuel et installer AWS CLI v2 et la cli azure
RUN . /opt/venv/bin/activate && \
    pip install --upgrade pip && \
    pip install awsebcli && pip install awscli && pip install azure-cli

# Assurez-vous que les commandes utilisent l'environnement virtuel
ENV PATH="/opt/venv/bin:$PATH"

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
ENV PATH=$PATH:/google-cloud-sdk/bin

# Installer kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Installer Helm
RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

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
    rm terraform_${release}_linux_${TERRAFORM_ARCH}.zip

WORKDIR /app