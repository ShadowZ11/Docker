# Utilise l'image Docker officielle DinD comme base
FROM docker:29-dind

# Installe Azure CLI et autres outils nécessaires
RUN apk add --no-cache \
    cargo \
    curl \
    jq \
    linux-headers \
    musl-dev \
    bash \
    gcc \
    py3-pip \
    python3-dev \
    nodejs \
    npm \
    zip \
    git

RUN npm i -g yarn

RUN python3 -m venv /opt/venv && \
        . /opt/venv/bin/activate && \
        pip install --upgrade pip && \
        pip install azure-cli && \
        pip install awsebcli && pip install awscli

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

ENV PATH="/opt/venv/bin:$PATH"

# Vérifie que tout est installé
RUN docker --version && az --version && aws --version && node --version

# Lance le daemon Docker à l'intérieur du conteneur
ENTRYPOINT ["dockerd-entrypoint.sh"]
