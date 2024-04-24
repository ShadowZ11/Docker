FROM node:20.10.0-alpine

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
    bash

RUN apk add --update --no-cache python3 && ln -sf python3 /usr/bin/python

# Installer kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Installer Helm
RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# Installer AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install

# Installer Terraform
RUN release=`curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest |  grep tag_name | cut -d: -f2 | tr -d \"\,\v | awk '{$1=$1};1'` && \
    curl "https://releases.hashicorp.com/terraform/${release}/terraform_${release}_linux_amd64.zip" -o "terraform_${release}_linux_amd64.zip" && \
    unzip terraform_${release}_linux_amd64.zip && \
    mv terraform /usr/bin/terraform && \
    rm terraform_${release}_linux_amd64.zip

WORKDIR /app
