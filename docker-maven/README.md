# Maven with Deployment Tools Docker Image

This Docker image is built with the intention
of providing a usable environment for building applications with Maven
while also providing several tools for deployment.
The image is based on `maven:3-eclipse-temurin-x`.

The Docker image comes with four separate tags corresponding to the version of Java it includes:
Java 8, Java 11, Java 17, and Java 21.

## Image Contents

- **Maven (3-eclipse-temurin-x):** Maven is a build automation tool used primarily for Java projects.
- **Various dependencies:** Including CA certificates, curl for making HTTP requests, apt-transport-https and software-properties-common for package management, gnupg and gnupg-agent for cryptography, openssl for handling SSL/TLS certificates, unzip for decompressing files, git for version control, openssh-client for connecting to SSH servers, and bash for command interpretation. Additionally, Python3 and its package installer pip are included in this Docker image.
- **Docker CLI:** Command line interface for managing Docker containers.
- **kubectl:** A command line tool for controlling Kubernetes clusters.
- **Helm:** A package manager for Kubernetes.
- **AWS CLI v2:** Amazon Web Services' Command Line Interface, version 2 + awsebcli.
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool. Terraform lets you define and provide datacenter infrastructure using a declarative configuration language.
- **Azure CLI:** A command-line tool for managing Azure resources.
- \*_**New**_\* **Google Cloud CLI SDK:** A set of tools for managing Google Cloud resources. (only gcloud cli is installed, for other tools, please install them manually)

## Usage

This Docker image is designed as a base for containers that need to build applications with Maven and deploy them to various platforms such as AWS. The Docker entrypoint is set to run `mvn --version`, effectively verifying the Maven installation upon container startup.

## Please Note

This image is intended for use by developers who are already familiar with the included tools and the Maven build system. If you're new to one or more of the included tools or to Maven, please refer to their respective official documentation.