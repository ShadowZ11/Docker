FROM node:24-alpine3.24 AS node

# Utilise l'image Docker officielle DinD comme base
FROM docker:29-dind

ARG BUILD_DATE
ARG VCS_REF
ARG VERSION=29

LABEL org.opencontainers.image.title="docker-dind"
LABEL org.opencontainers.image.description="Docker-in-Docker image with Node.js and Terraform"
LABEL org.opencontainers.image.source="https://github.com/ShadowZ11/Docker"
LABEL org.opencontainers.image.url="https://hub.docker.com/r/shad0wz7/docker-dind"
LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${VCS_REF}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

# Node + npm copiés depuis l'image officielle (au lieu du paquet apk nodejs, en retard)
COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules

# Kept free of cloud CLIs: when DinD runs as a CI service the job container calls
# them, not the daemon. Use the aws-cli / azure-cli variants if the job needs them.
RUN set -eux; \
    apk add --no-cache \
      bash \
      ca-certificates \
      curl \
      git \
      jq \
      libstdc++ \
      tar \
      zip \
      unzip; \
    ln -s ../lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm; \
    ln -s ../lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx; \
    node --version; \
    npm --version; \
    npm i -g yarn; \
    rm -rf /root/.npm

# Terraform tracks upstream latest (see CLAUDE.md) but the download is
# checksum-verified against HashiCorp's own SHA256SUMS manifest.
RUN set -eux; \
    ARCH="$(uname -m)"; \
    case "$ARCH" in \
      x86_64) TOOL_ARCH=amd64 ;; \
      aarch64|arm64) TOOL_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac; \
    TERRAFORM_VERSION="$(curl -fsSL https://checkpoint-api.hashicorp.com/v1/check/terraform | sed -n 's/.*"current_version":"\([^"]*\)".*/\1/p')"; \
    test -n "$TERRAFORM_VERSION"; \
    TERRAFORM_ZIP="terraform_${TERRAFORM_VERSION}_linux_${TOOL_ARCH}.zip"; \
    curl -fsSLo "$TERRAFORM_ZIP" "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/${TERRAFORM_ZIP}"; \
    curl -fsSL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS" \
      | grep " ${TERRAFORM_ZIP}$" | sha256sum -c -; \
    unzip -q "$TERRAFORM_ZIP" terraform -d /usr/local/bin; \
    chmod 0755 /usr/local/bin/terraform; \
    rm "$TERRAFORM_ZIP"

# Lance le daemon Docker à l'intérieur du conteneur
ENTRYPOINT ["dockerd-entrypoint.sh"]

# DEBUG UNIQUEMENT, ne pas déployer : remplace le démarrage de dockerd.
# Chemin complet obligatoire, sinon l'entrypoint le transforme en "docker node --version".
# CMD ["/usr/local/bin/node", "--version"]