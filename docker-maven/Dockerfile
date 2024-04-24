FROM maven:3.9.6-eclipse-temurin-8
LABEL authors="Manub"

# Installer les dépendances nécessaires
RUN apt-get update && \
    apt-get install -y \
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
    && ln -sf python3 /usr/bin/python

RUN     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
        apt-get update && \
        apt-get install -y docker-ce docker-ce-cli containerd.io

# Installer terraform
RUN release=`curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest |  grep tag_name | cut -d: -f2 | tr -d \"\,\v | awk '{$1=$1};1'` && \
    curl "https://releases.hashicorp.com/terraform/${release}/terraform_${release}_linux_amd64.zip" -o "terraform_${release}_linux_amd64.zip" && \
    unzip terraform_${release}_linux_amd64.zip && \
    mv terraform /usr/bin/terraform && \
    rm terraform_${release}_linux_amd64.zip

# Installer AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install

CMD ["mvn", "--version"]