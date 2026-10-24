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
- **Terraform:** An Open Source Infrastructure as Code (IaC) tool. Terraform lets you define and
  provide datacenter infrastructure using a declarative configuration language.

## Variants

The base image no longer ships any cloud CLI, which keeps it small and fast to build. When the
job itself (not just the daemon) needs to talk to a cloud provider, use a variant:

| Folder | Docker Hub image | Adds |
| --- | --- | --- |
| [`docker-dind`](.) | `shad0wz7/docker-dind` | — (DinD + Node.js + Terraform only) |
| [`docker-dind/aws-cli`](aws-cli) | `shad0wz7/docker-dind-aws-cli` | AWS CLI + `awsebcli` (Elastic Beanstalk) |
| [`docker-dind/azure-cli`](azure-cli) | `shad0wz7/docker-dind-azure-cli` | Azure CLI |

The AWS and Azure CLIs are installed via pip into a venv, since Alpine/musl has no official
AWS CLI v2 or Azure CLI build.

> **Breaking change:** `shad0wz7/docker-dind` used to include both the AWS and Azure CLIs. If a
> pipeline runs `aws`, `eb` or `az` inside it, switch to the matching variant above, or pin the
> last `YYYYMMDD` tag published before the split to keep the old all-in-one image.

Unlike the [`docker-maven`](../docker-maven) and [`docker-nodeJS`](../docker-nodeJS) images, none
of these images include `kubectl`, Helm or the Google Cloud CLI. Add them yourself if your
pipeline needs them.

## Usage

This Docker image is designed to be run as a Docker-in-Docker service in a CI/CD pipeline (for
example as a GitLab CI `services:` entry, or a sidecar container), so that a job can build and
push images while also deploying with Terraform (or AWS / Azure with the variants). The entrypoint runs
`dockerd-entrypoint.sh`, which starts the Docker daemon.

Because this image runs a Docker daemon, it generally needs to be started in privileged mode
(`--privileged`) or with an equivalent DinD setup, exactly like the upstream `docker:29-dind`
image.

## Please Note

This image is intended for use by developers and DevOps engineers who are already familiar with
Docker-in-Docker and the included CLIs. If you're new to one or more of the included tools,
please refer to their respective official documentation.
