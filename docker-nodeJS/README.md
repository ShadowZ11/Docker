# Deployment Tools Docker Image NodeJS

This Docker image is built to be used as an environment for running deployment scripts and commands across multiple
platforms.
It is based on `node:24-alpine` and includes several useful tools for interacting with various services
and clusters.

Every image published from `main` is a multi-architecture manifest covering `amd64`
and `arm64`, so `latest` works as-is on ARM hosts; no separate architecture-specific
tag is needed.

> ### Breaking change: Helm 4
>
> These images now install the latest **Helm 4.x**. They previously shipped the
> latest Helm 3.x, so upgrading to a newer image tag crosses a major Helm version
> with breaking changes.
>
> If your pipeline is not ready to migrate, pin `shad0wz7/<image>:<commit-sha>`
> using a commit from before this change instead of tracking `latest`. Builds from
> now on also publish a dated `YYYYMMDD` tag, which is the easiest thing to pin
> going forward. See the [Helm documentation](https://helm.sh/docs/) for migration
> guidance.

## Image Contents

- **Node.js (24-alpine):** JavaScript runtime built on Chrome's V8 JavaScript engine. Upgraded from Node 20,
  which reached end-of-life on 2026-04-30; Node 24 is the current Active LTS and is supported until 2028-04-30.
- **Various dependencies:** Includes CA certificates, curl for making HTTP requests, gnupg for cryptography, docker-cli
  for managing Docker containers, openssl for handling SSL/TLS certificates, unzip for decompressing files, git for
  version control, openssh-client for connecting to SSH servers, and bash for command interpretation.
- **Python3:** A Python 3 interpreter.
- **kubectl:** A command line tool for controlling Kubernetes clusters.
- **Helm 4:** A package manager for Kubernetes (see the Helm 4 notice above). Present in this image and in the
  `gcloud-cli` variant; the `aws-cli` and `azure-cli` variants do not ship Helm.
- **AWS CLI v2:** Amazon Web Services' Command Line Interface, version 2.
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool that allows defining and providing datacenter
  infrastructure.
- **Azure CLI:** A command-line tool for managing Azure resources.
- \*_**New**_\* **Google Cloud CLI SDK:** A set of tools for managing Google Cloud resources.

## Usage

This Docker image serves as a base
for creating containers intended for deployment and management of resources in various platforms,
including Kubernetes, AWS, and others.

The working directory is set to `/app`.
This means any command run within the container
(unless a different path is specified in the command itself) will be run within this directory.
You can place your deployment scripts and other required files here.

The primary use case for this image is to run deployment scripts and commands inside your pipeline.
You can run the container interactively to execute commands manually,
or you can run it non-interactively to execute a script or command.

## Please Note

This image is intended for use by developers and devops who are already familiar with the included tools.
If you're new to one or more of the included tools, please refer to their respective official documentation.