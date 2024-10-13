FROM maven:3-eclipse-temurin-21
LABEL authors="Manub"

# Installer les dépendances nécessaires
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    bash \
    ca-certificates \
    curl \
    git \
    gnupg \
    gnupg-agent \
    jq \
    openssh-client \
    openssl \
    python3 \
    python3-pip \
    python3-venv \
    software-properties-common \
    unzip \
    && ln -sf python3 /usr/bin/python \
    && apt remove -y software-properties-common

#Installer la CLI Google Cloud
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
    | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
    && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
    | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg  \
    && apt-get update -y  \
    && apt-get install google-cloud-cli -y \
    && apt-get clean

ENV PATH="/opt/venv/bin:$PATH"

RUN ARCH=$(uname -m) && \
    # Détecter l'architecture pour kubectl
    if [ "$ARCH" = "x86_64" ]; then \
      KUBECTL_ARCH="amd64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
      KUBECTL_ARCH="arm64"; \
    else \
      KUBECTL_ARCH="amd64"; \
    fi && \
    # Télécharger et installer kubectl
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/${KUBECTL_ARCH}/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# Installer Helm
RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# Détecter l'architecture pour Terraform
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
      TERRAFORM_ARCH="amd64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
      TERRAFORM_ARCH="arm64"; \
    else \
      echo "Architecture non supportée: $ARCH"; exit 1; \
    fi && \
    # Télécharger et installer Terraform
    release=$(curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d: -f2 | tr -d \"\,\v | awk '{$1=$1};1') && \
    curl "https://releases.hashicorp.com/terraform/${release}/terraform_${release}_linux_${TERRAFORM_ARCH}.zip" -o "terraform_${release}_linux_${TERRAFORM_ARCH}.zip" && \
    unzip terraform_${release}_linux_${TERRAFORM_ARCH}.zip && \
    mv terraform /usr/bin/terraform && \
    rm terraform_${release}_linux_${TERRAFORM_ARCH}.zip

# Nettoyage des paquets inutiles et des fichiers temporaires
RUN apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app