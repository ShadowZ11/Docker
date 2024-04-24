# Deployment Tools Docker Image NodeJS

This Docker image is built to be used as an environment for running deployment scripts and commands across multiple
platforms.
It is based on `node:20.10.0-alpine` and includes several useful tools for interacting with various services
and clusters.

## Image Contents

- **Node.js (20.10.0-alpine):** JavaScript runtime built on Chrome's V8 JavaScript engine.
- **Various dependencies:** Includes CA certificates, curl for making HTTP requests, gnupg for cryptography, docker-cli
  for managing Docker containers, openssl for handling SSL/TLS certificates, unzip for decompressing files, git for
  version control, openssh-client for connecting to SSH servers, and bash for command interpretation.
- **Python3:** A Python 3 interpreter.
- **kubectl:** A command line tool for controlling Kubernetes clusters.
- **Helm:** A package manager for Kubernetes.
- **AWS CLI v2:** Amazon Web Services' Command Line Interface, version 2.
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool that allows defining and providing datacenter
  infrastructure.

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