# Docker-in-Docker with Deployment Tools Docker Image

This Docker image is built to provide a Docker-in-Docker (DinD) environment for CI/CD pipelines
that need to build and push container images while also having common cloud deployment tools
available in the same job. It is based on `docker:29-dind`.

## Image Contents

- **Docker-in-Docker (docker:29-dind):** Runs a Docker daemon inside the container, so the
  container itself can build and run other containers.
- **Various dependencies:** CA certificates, curl for making HTTP requests, git for version
  control, jq for JSON processing, unzip/zip for decompressing files, and bash for command
  interpretation.
- **Node.js and npm:** Included for scripts that need a JavaScript runtime, plus Yarn installed
  globally.
- **Python3:** A Python 3 interpreter, used to install the Python-based CLIs below.
- **AWS CLI:** Amazon Web Services' Command Line Interface, plus `awsebcli` for Elastic
  Beanstalk. Installed via pip since Alpine/musl has no official AWS CLI v2 build.
- **Azure CLI:** A command-line tool for managing Azure resources. Installed via pip for the
  same musl reason as above.
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool. Terraform lets you define and
  provide datacenter infrastructure using a declarative configuration language.

Unlike the [`docker-maven`](../docker-maven) and [`docker-nodeJS`](../docker-nodeJS) images, this
image does **not** include `kubectl`, Helm or the Google Cloud CLI. It is intentionally scoped to
DinD + AWS/Azure/Terraform; add the missing tools yourself if your pipeline needs them.

## Usage

This Docker image is designed to be run as a Docker-in-Docker service in a CI/CD pipeline (for
example as a GitLab CI `services:` entry, or a sidecar container), so that a job can build and
push images while also deploying with AWS, Azure or Terraform. The entrypoint runs
`dockerd-entrypoint.sh`, which starts the Docker daemon.

Because this image runs a Docker daemon, it generally needs to be started in privileged mode
(`--privileged`) or with an equivalent DinD setup, exactly like the upstream `docker:29-dind`
image.

## Please Note

This image is intended for use by developers and DevOps engineers who are already familiar with
Docker-in-Docker and the included CLIs. If you're new to one or more of the included tools,
please refer to their respective official documentation.
