# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is **not an application** — it's a collection of standalone `Dockerfile`s that produce ready-to-use CI/CD tool images (Maven/Node.js build environments bundled with cloud CLIs, `kubectl`, `helm`, `terraform`, etc.), published to Docker Hub under the `shad0wz7` namespace. There is no shared build system, package manifest, or source code to compile — each folder is an independent image definition.

## Repository layout

```
docker-dind/                  # Docker-in-Docker + Node.js + Terraform (no cloud CLI)
docker-dind/aws-cli/          # DinD + AWS CLI
docker-dind/azure-cli/        # DinD + Azure CLI
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

**Important nuance:** the root `docker-maven/Dockerfile` and `docker-nodeJS/Dockerfile` ("kitchen sink" images with all three cloud CLIs) exist in the repo but are marked `"disabled": true` in the image catalogue inside `.github/workflows/docker-image.yml`, so they are never built or pushed. Only the single-cloud variants (`aws-cli`, `azure-cli`, `gcloud-cli` subfolders) and `docker-dind` (plus its `aws-cli`/`azure-cli` variants) are actively published. `docker-java` is disabled in the catalogue too, to push users onto `docker-maven`: its last published tags stay on Docker Hub but no longer receive updates. Don't assume the root images are being published without checking that catalogue.

`docker-java` is explicitly deprecated (see `docker-java/README.md`) — the maintained replacement is `docker-maven`. Don't add new functionality there.

## Conventions used across Dockerfiles

- Build args `BUILD_DATE`, `VCS_REF`, `VERSION` feed OCI image labels (`org.opencontainers.image.*`) — preserve these when editing a Dockerfile.
- Tool versions for `kubectl`, `helm` and `terraform` are resolved dynamically at build time (`dl.k8s.io/release/stable.txt`, `get.helm.sh/helm-latest-version`, `checkpoint-api.hashicorp.com`), not pinned — intentional per the project's floating-latest strategy. Because the GHA layer cache would otherwise freeze these versions forever, the workflow rebuilds everything cache-less on a monthly schedule (1st of the month); keep that job when touching the workflow. Urgent rebuilds (critical CVE) go through `workflow_dispatch` with `no_cache`.
- **Every downloaded binary must be checksum-verified** against the publisher's own manifest (`kubectl.sha256`, `<tarball>.sha256sum` for Helm, `terraform_<v>_SHA256SUMS`). Don't add a `curl | bash` installer or an unverified download — the previous `get-helm-3 | bash` was replaced for exactly this reason. Note this puts the images on **Helm 4**, not Helm 3.
- Cloud CLIs come from a signed apt repository where the vendor publishes one (Azure CLI and Google Cloud CLI on the Debian-based Maven images) and from the official self-contained bundle for AWS CLI v2. The Alpine variants have no vendor musl build for AWS/Azure, so there they stay on pip — which is why those images need a `.build-deps` virtual package (`cargo`, `gcc`, `musl-dev`, `linux-headers`, …) that is `apk del`-ed in the same layer.
- Architecture-aware installs use `uname -m` + a `case` statement to select `amd64`/`arm64` (or `x86_64`/`arm` for the Google Cloud SDK tarball) — multi-arch (`linux/amd64,linux/arm64`) builds only happen on `main` pushes; PR builds are single-arch (`amd64`) validation only.
- Python-based CLIs that remain on pip (`azure-cli` and `awscli` on Alpine, `awsebcli` everywhere) are installed into a venv with `PATH` extended, to avoid conflicting with the system Python. Only create that venv if something is actually installed into it: an empty `/opt/venv` at the head of `PATH` shadows the system python for no benefit.
- Alpine-based images (`docker-dind*`, `docker-nodeJS*`) use `apk`; Debian/Ubuntu-based images (`docker-maven*`) use `apt-get`; `docker-java` (Amazon Corretto/Alpine) uses `apk`.

## Building and testing locally

There's no test suite. Validation is building the image and running its smoke command — the list of tools each image must be able to execute lives in the `smoke` field of the image catalogue in `.github/workflows/docker-image.yml`, and CI runs it against every built image. When you add a tool to a Dockerfile, add it to that image's `smoke` string too.

```bash
# Build a single variant locally
docker build -t test-image ./docker-maven/aws-cli

# Build for a specific platform (mirrors CI's PR validation build)
docker build --platform linux/amd64 -t test-image ./docker-nodeJS/azure-cli

# Sanity-check the tools installed in the built image
docker run --rm test-image bash -c "aws --version && kubectl version --client && terraform -version"
```

## CI/CD (`.github/workflows/docker-image.yml`)

- Uses `dorny/paths-filter` to detect which image folder(s) changed, then generates a build matrix from a JSON image catalogue embedded in the `detect-changes` job. To add a variant: add one catalogue entry (`key`, `context`, `image`, `description`, `smoke`) plus a matching paths-filter entry. There is one `build` job for all images — don't reintroduce per-image jobs.
- Pushes to Docker Hub only on `push` to `main` (requires `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` repo secrets); `pull_request` runs build a single-arch validation image without pushing.
- Tags and OCI labels come from `docker/metadata-action`; pushes to `main` publish `latest`, `${{ github.sha }}` and a `YYYYMMDD` tag. `BUILD_DATE`/`VCS_REF` are passed as build args so the Dockerfile labels are actually populated.
- `workflow_dispatch` and the monthly `schedule` force every image to build regardless of changed paths. A change to the workflow file itself rebuilds every image on PRs (validation only), but on `main` only the images whose folder changed are republished — unless a pushed commit's subject line (or the PR title line of a `Merge pull request` commit) contains `[rebuild-all]`. A `[rebuild: dind, node_aws]` marker (same lines on `main`, PR title on PRs) adds the named images to the selection without removing any; names are comma-separated catalogue keys (`_` or `-`), image names, or a family (`maven`, `node` = all their enabled variants), and an unknown name fails the run. Commit bodies are deliberately ignored: a commit that merely documented `[rebuild: ...]` in its body once broke a `main` run by being parsed as a marker. `workflow_dispatch` takes a `no_cache` input; the scheduled run always builds cache-less.
- Trivy scans each image for `CRITICAL,HIGH` OS/library vulnerabilities and uploads SARIF to the repository Security tab (one `category` per image). The scan is still non-blocking, but findings now surface as code-scanning alerts and PR annotations rather than only in the job log.
- `concurrency` cancels in-progress runs **for pull requests only**; runs on `main` are never cancelled so a rapid second push cannot drop a publish.

## Other notes

- `CODEOWNERS` (`.github/CODEOWNERS`) requires review from `@ShadowZ11` and `@Asma-B-M` on all files.
- `dependabot.yml` only tracks `github-actions` updates (weekly) — Dockerfile base image/tool versions are not managed by Dependabot and must be bumped manually.
- Licensed under MIT.
