FROM maven:3.9.6-eclipse-temurin-21
LABEL authors="Manub"

# Installer les dépendances nécessaires
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    apt-transport-https \
    software-properties-common \
    gnupg \
    gnupg-agent \
    openssl \
    unzip \
    git \
    openssh-client \
    bash \
    python3 \
    python3-pip \
    && ln -sf python3 /usr/bin/python \
    && apt remove -y software-properties-common \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Docker Installation
RUN curl -fsSL https://get.docker.com/ | bash

WORKDIR /tmp

# kubectl Installation
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# Helm Installation
RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# Terraform Installation
RUN release=`curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest |  grep tag_name | cut -d: -f2 | tr -d \"\,\v | awk '{$1=$1};1'` && \
    curl "https://releases.hashicorp.com/terraform/${release}/terraform_${release}_linux_amd64.zip" -o "terraform_${release}_linux_amd64.zip" && \
    unzip terraform_${release}_linux_amd64.zip && \
    mv terraform /usr/bin/terraform && \
    rm terraform_${release}_linux_amd64.zip

# Installation de AWS CLI v2
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir awsebcli awscli

# Installation de CLI Azure
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

CMD ["mvn", "--version"]