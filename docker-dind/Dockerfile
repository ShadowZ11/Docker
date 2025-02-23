# Utilise l'image Docker officielle DinD comme base
FROM docker:28-dind

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
    python3-dev

RUN python3 -m venv /opt/venv && \
        . /opt/venv/bin/activate && \
        pip install --upgrade pip && \
        pip install azure-cli && \
        pip install awsebcli && pip install awscli

ENV PATH="/opt/venv/bin:$PATH"

# Vérifie que tout est installé
RUN docker --version && az --version && aws --version

# Lance le daemon Docker à l'intérieur du conteneur
ENTRYPOINT ["dockerd-entrypoint.sh"]
