# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is **not an application** — it's a collection of standalone `Dockerfile`s that produce ready-to-use CI/CD tool images (Maven/Node.js build environments bundled with cloud CLIs, `kubectl`, `helm`, `terraform`, etc.), published to Docker Hub under the `shad0wz7` namespace. There is no shared build system, package manifest, or source code to compile — each folder is an independent image definition.

## Repository layout

```
docker-dind/                  # Docker-in-Docker + Azure/AWS CLI + Terraform (all-in-one)
docker-java/                  # DEPRECATED — kept for compatibility only, do not extend
docker-maven/                 # "Kitchen sink" Maven image: AWS + Azure + GCloud CLI + kubectl/Helm/Terraform
docker-maven/aws-cli/         # Maven + AWS CLI only
docker-maven/azure-cli/       # Maven + Azure CLI only
docker-maven/gcloud-cli/      # Maven + Google Cloud CLI only
docker-nodeJS/                # "Kitchen sink" Node.js image: AWS + Azure + GCloud CLI + kubectl/Helm/Terraform
docker-nodeJS/aws-cli/        # Node.js + AWS CLI only
docker-nodeJS/azure-cli/      # Node.js + Azure CLI only
docker-nodeJS/gcloud-cli/     # Node.js + Google Cloud CLI only
```

Each folder is self-contained: Dockerfiles are not chained with `FROM` off one another, so the tool-install `RUN` blocks are intentionally duplicated per variant rather than shared. When updating a tool version or fixing an install step in one variant, check whether the same block exists in sibling variants (e.g. `docker-maven/*-cli/Dockerfile` all install `kubectl`/`helm`/`terraform` the same way) and decide whether the fix applies there too — don't assume a shared base will pick it up.

**Important nuance:** the root `docker-maven/Dockerfile` and `docker-nodeJS/Dockerfile` ("kitchen sink" images with all three cloud CLIs) exist in the repo but their CI build jobs (`build-maven`, `build-node`) are currently **commented out** in `.github/workflows/docker-image.yml`. Only the single-cloud variants (`aws-cli`, `azure-cli`, `gcloud-cli` subfolders) and `docker-dind`/`docker-java` are actively built and pushed. Don't assume the root images are being published without checking the workflow file.

`docker-java` is explicitly deprecated (see `docker-java/README.md`) — the maintained replacement is `docker-maven`. Don't add new functionality there.

## Conventions used across Dockerfiles

- Build args `BUILD_DATE`, `VCS_REF`, `VERSION` feed OCI image labels (`org.opencontainers.image.*`) — preserve these when editing a Dockerfile.
- Tool versions for `kubectl` and `terraform` are resolved dynamically at build time (latest stable via `dl.k8s.io`/`releases.hashicorp.com`), not pinned — this is intentional per the project's floating-latest strategy, except where a CLI is explicitly version-pinned (e.g. `awscli==1.29.85`, `awsebcli==3.27.2` in the Maven-based images).
- Architecture-aware installs use `uname -m` + a `case` statement to select `amd64`/`arm64` (or `x86_64`/`arm` for the Google Cloud SDK tarball) — multi-arch (`linux/amd64,linux/arm64`) builds only happen on `main` pushes; PR builds are single-arch (`amd64`) validation only.
- Python-based CLIs (`awscli`, `azure-cli`, `awsebcli`) are installed into a venv at `/opt/venv` with `PATH` extended, to avoid conflicting with the system Python.
- Alpine-based images (`docker-dind`, `docker-nodeJS*`) use `apk`; Debian/Ubuntu-based images (`docker-maven*`) use `apt-get`; `docker-java` (Amazon Corretto/Alpine) uses `apk`.

## Building and testing locally

There's no test suite — validation is building the image and exercising the final version-check commands each Dockerfile ends with (e.g. `docker --version && az --version && aws --version && node --version` in `docker-dind`).

```bash
# Build a single variant locally
docker build -t test-image ./docker-maven/aws-cli

# Build for a specific platform (mirrors CI's PR validation build)
docker build --platform linux/amd64 -t test-image ./docker-nodeJS/azure-cli

# Sanity-check the tools installed in the built image
docker run --rm test-image bash -c "aws --version && kubectl version --client && terraform -version"
```

## CI/CD (`.github/workflows/docker-image.yml`)

- Uses `dorny/paths-filter` to detect which image folder(s) changed, then only builds the affected job(s). Add a new filter entry + job pair here when adding a new image variant.
- Pushes to Docker Hub only on `push` to `main` (requires `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` repo secrets); `pull_request` runs build a single-arch validation image without pushing.
- Every push to `main` publishes both a `latest` tag and a `${{ github.sha }}` tag.
- `workflow_dispatch` forces all jobs to run regardless of changed paths — use this to rebuild everything (e.g. after a shared base image or tool-version change upstream).
- Each build job runs a non-blocking Trivy scan (`continue-on-error: true`) against `CRITICAL,HIGH` OS/library vulnerabilities — scan failures don't fail the build, so check the job's Trivy step output manually when reviewing a PR.
- `concurrency` cancels in-progress runs for the same ref, so rapid pushes to the same branch/PR won't queue redundant builds.

## Other notes

- `CODEOWNERS` (`.github/CODEOWNERS`) requires review from `@ShadowZ11` and `@Asma-B-M` on all files.
- `dependabot.yml` only tracks `github-actions` updates (weekly) — Dockerfile base image/tool versions are not managed by Dependabot and must be bumped manually.
- Licensed under MIT.
