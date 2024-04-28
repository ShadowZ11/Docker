# [DEPRECATED] Java Amazon Corretto with Deployment Tools Docker Image

> **IMPORTANT:** This Docker image is deprecated. It's recommended to use the Maven image instead.  
> https://hub.docker.com/repository/docker/shad0wz7/maven-terraform/general  
> [Maven Image](../docker-maven)

This Docker image was previously built for a use case of providing an environment for deploying applications with various tools. The image was based on `amazoncorretto:11.0.21-alpine3.18`. However, this image is no longer maintained and updated, therefore it has been deprecated. Please switch to use the Maven Docker image which has similar functionalities and is actively maintained.

The Docker image used to be tagged with three separate tags corresponding to the version of Java it included: one for Java 8, one for Java 11, and one for Java 21. Please note that these tags would no longer receive updates since the image has been deprecated.

## Image Contents

- **Amazon Corretto (11.0.21-alpine3.18):** A no-cost, multi-platform, production-ready distribution of the Open Java Development Kit (OpenJDK).
- **Various dependencies:** CA certificates, curl for making HTTP requests, gnupg for cryptography, docker CLI for managing Docker containers, openssl for handling SSL/TLS certificates, Maven for building Java projects, unzip for decompressing files, git for version control, openssh-client for connecting to SSH servers, and bash for command interpretation. Additionally, Python3 is included in this Docker image.
- **kubectl:** A command-line tool for controlling Kubernetes clusters.
- **Helm:** A package manager for Kubernetes.
- **AWS CLI v2:** Amazon Web Services' Command Line Interface, version 2.
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool that allows defining and providing datacenter infrastructure.
- **Azure CLI:** A command-line tool for managing Azure resources.

## Usage

The Docker image was designed as a base for containers that build and deploy applications using several tools.

The working directory was set to `/app`. This means any command run within the container would be run within this directory.

## Please Note

Please switch to the Maven Docker image as this Docker image is deprecated and no longer maintained, and its versions for Java 8, 11 and 21 would no longer receive updates.