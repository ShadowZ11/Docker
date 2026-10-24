# Docker tool images

This repository contains ready-to-use Docker images for CI/CD pipelines that need common build and deployment tools.
It is now structured to be **public-ready**:

- automatic Docker Hub publishing only when a given image folder changes;
- public-friendly image tags (`latest` + commit SHA);
- clearer image metadata and build instructions;
- a repository license so the project can be shared publicly with confidence.

## Published images

| Folder | Docker Hub image | Purpose |
| --- | --- | --- |
| [`docker-dind`](docker-dind) | `shad0wz7/docker-dind` | Docker-in-Docker + Node.js + Terraform |
| [`docker-dind/aws-cli`](docker-dind/aws-cli) | `shad0wz7/docker-dind-aws-cli` | Docker-in-Docker + AWS deployment tooling |
| [`docker-dind/azure-cli`](docker-dind/azure-cli) | `shad0wz7/docker-dind-azure-cli` | Docker-in-Docker + Azure deployment tooling |
| [`docker-java`](docker-java) | `shad0wz7/java-aws-terraform` | Java runtime + deployment tooling |
| [`docker-maven`](docker-maven) | `shad0wz7/maven-terraform` | Maven build image + deployment tooling |
| [`docker-maven/aws-cli`](docker-maven/aws-cli) | `shad0wz7/maven-aws-cli` | Maven + AWS deployment tooling |
| [`docker-maven/azure-cli`](docker-maven/azure-cli) | `shad0wz7/maven-azure-cli` | Maven + Azure deployment tooling |
| [`docker-maven/gcloud-cli`](docker-maven/gcloud-cli) | `shad0wz7/maven-gcloud-cli` | Maven + Google Cloud deployment tooling |
| [`docker-nodeJS`](docker-nodeJS) | `shad0wz7/node-aws-terraform` | Node.js build image + deployment tooling |
| [`docker-nodeJS/aws-cli`](docker-nodeJS/aws-cli) | `shad0wz7/node-aws-cli` | Node.js + AWS deployment tooling |
| [`docker-nodeJS/azure-cli`](docker-nodeJS/azure-cli) | `shad0wz7/node-azure-cli` | Node.js + Azure deployment tooling |
| [`docker-nodeJS/gcloud-cli`](docker-nodeJS/gcloud-cli) | `shad0wz7/node-gcloud-cli` | Node.js + Google Cloud deployment tooling |

## CI/CD behavior

The GitHub Actions workflow in [`.github/workflows/docker-image.yml`](.github/workflows/docker-image.yml) does the following:

1. detects which image folder changed in the commit;
2. builds only the impacted image(s), as a single matrix job driven by the image
   catalogue embedded in the workflow;
3. pushes to Docker Hub on `main` only;
4. publishes `amd64` + `arm64` images for `main` pushes, and a single-arch validation build for pull requests;
5. publishes `latest`, the commit SHA and a `YYYYMMDD` tag;
6. smoke-tests every built image by running its CLIs, so a tool that installs but
   does not actually run fails the build;
7. runs a Trivy scan and uploads the findings to the repository Security tab.

Because `kubectl`, Helm and Terraform are resolved to the latest upstream release
at build time, the workflow also rebuilds every image cache-less once a week
(Mondays, 04:00 UTC). Without that, the layer cache would freeze those tools at
whatever version was current when the layer was first built. `workflow_dispatch`
exposes a `no_cache` input to force the same behaviour on demand.

## Required secrets

Create these repository secrets in GitHub before enabling pushes:

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

## Usage

### Maven image

```bash
docker pull shad0wz7/maven-terraform:latest
```

### Maven AWS image

```bash
docker pull shad0wz7/maven-aws-cli:latest
```

### Maven Azure image

```bash
docker pull shad0wz7/maven-azure-cli:latest
```

### Maven Google Cloud image

```bash
docker pull shad0wz7/maven-gcloud-cli:latest
```

### Java image

```bash
docker pull shad0wz7/java-aws-terraform:latest
```

### DinD image

```bash
docker pull shad0wz7/docker-dind:latest
```

### DinD AWS image

```bash
docker pull shad0wz7/docker-dind-aws-cli:latest
```

### DinD Azure image

```bash
docker pull shad0wz7/docker-dind-azure-cli:latest
```

### Node.js image

```bash
docker pull shad0wz7/node-aws-terraform:latest
```

### Node.js AWS image

```bash
docker pull shad0wz7/node-aws-cli:latest
```

### Node.js Azure image

```bash
docker pull shad0wz7/node-azure-cli:latest
```

### Node.js Google Cloud image

```bash
docker pull shad0wz7/node-gcloud-cli:latest
```

## Make the repository public safely

If you want the repository itself to be public, this is a good baseline. Before switching visibility:

- verify there are no secrets committed in the history;
- configure the Docker Hub secrets above;
- make sure the Docker Hub repositories exist and are public;
- optionally add or review the `LICENSE` file.

## Notes

- `docker-java` is kept for compatibility but should be considered the legacy image.
- The `README.md` inside each folder contains more details about the contents of that image.
- If you want, I can also add GitHub release tags, semantic version tags, or a multi-arch build matrix next.
