# Docker tool images

This repository contains ready-to-use Docker images for CI/CD pipelines that need common build and deployment tools.

- automatic Docker Hub publishing only when a given image folder changes;
- predictable image tags (`latest`, commit SHA and `YYYYMMDD`);
- clearer image metadata and build instructions;

## Published images

| Folder | Docker Hub image | Purpose |
| --- | --- | --- |
| [`docker-dind`](docker-dind) | `shad0wz7/docker-dind` | Docker-in-Docker + Node.js + Terraform |
| [`docker-dind/aws-cli`](docker-dind/aws-cli) | `shad0wz7/docker-dind-aws-cli` | Docker-in-Docker + AWS deployment tooling |
| [`docker-dind/azure-cli`](docker-dind/azure-cli) | `shad0wz7/docker-dind-azure-cli` | Docker-in-Docker + Azure deployment tooling |
| [`docker-maven/aws-cli`](docker-maven/aws-cli) | `shad0wz7/maven-aws-cli` | Maven + AWS deployment tooling |
| [`docker-maven/azure-cli`](docker-maven/azure-cli) | `shad0wz7/maven-azure-cli` | Maven + Azure deployment tooling |
| [`docker-maven/gcloud-cli`](docker-maven/gcloud-cli) | `shad0wz7/maven-gcloud-cli` | Maven + Google Cloud deployment tooling |
| [`docker-nodeJS/aws-cli`](docker-nodeJS/aws-cli) | `shad0wz7/node-aws-cli` | Node.js + AWS deployment tooling |
| [`docker-nodeJS/azure-cli`](docker-nodeJS/azure-cli) | `shad0wz7/node-azure-cli` | Node.js + Azure deployment tooling |
| [`docker-nodeJS/gcloud-cli`](docker-nodeJS/gcloud-cli) | `shad0wz7/node-gcloud-cli` | Node.js + Google Cloud deployment tooling |

### No longer published

These folders are still in the repository but are disabled in the workflow's image
catalogue: they are neither built nor pushed, and their existing tags on Docker Hub
no longer receive updates (base image, tools or security fixes).

| Folder | Docker Hub image | Status | Use instead |
| --- | --- | --- | --- |
| [`docker-java`](docker-java) | `shad0wz7/java-aws-terraform` | Deprecated | `shad0wz7/maven-*-cli` |
| [`docker-maven`](docker-maven) | `shad0wz7/maven-terraform` | Disabled (all three cloud CLIs in one image) | `shad0wz7/maven-aws-cli`, `maven-azure-cli` or `maven-gcloud-cli` |
| [`docker-nodeJS`](docker-nodeJS) | `shad0wz7/node-aws-terraform` | Disabled (all three cloud CLIs in one image) | `shad0wz7/node-aws-cli`, `node-azure-cli` or `node-gcloud-cli` |

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
at build time, the workflow also rebuilds every image cache-less once a month
(1st of the month, 04:00 UTC). Without that, the layer cache would freeze those
tools at whatever version was current when the layer was first built.

### Forcing a rebuild

By default, a push to `main` only republishes the images whose folder changed
(a change to the workflow file alone revalidates every image on the PR, but
republishes nothing extra on `main`). To rebuild more than that:

| Need | How |
| --- | --- |
| Rebuild **every** image now (e.g. critical CVE in a base image or tool) | Run the workflow manually (**Actions → Publish Docker images to Docker Hub → Run workflow**) and tick `no_cache` so `kubectl`, Helm and Terraform are re-resolved to their latest release. |
| Rebuild **every** image as part of a change | Put `[rebuild-all]` in the **first line** of a commit pushed to `main` — typically the PR title, which GitHub puts in the merge/squash commit. |
| Rebuild **specific** images as part of a change | Put `[rebuild: name1, name2]` in the PR title (checked on the PR, and carried into the merge/squash commit) or in the first line of a commit pushed to `main`. |

Names accepted by `[rebuild: ...]` (comma-separated, case-insensitive, `_` or `-`):

- a catalogue key: `dind`, `dind_aws`, `dind_azure`, `maven_aws`, `maven_azure`,
  `maven_gcloud`, `node_aws`, `node_azure`, `node_gcloud`;
- a Docker Hub image name, e.g. `node-aws-cli`;
- a family: `maven` or `node` = all their enabled variants.

`[rebuild: dind]` targets only the base `docker-dind` image, since `dind` is
itself a key; list `dind_aws` / `dind_azure` explicitly to include them. Markers
only **add** images to the selection, never remove any. An unknown name fails the
run (the error lists the valid names) rather than silently skipping a rebuild.

Markers are only read from commit **subject lines** (and the PR title line of a
`Merge pull request` commit), never from commit bodies, so a commit that only
describes the syntax — like this README — does not trigger a rebuild.

Example PR title: `fix(node): bump base image [rebuild: node, dind_aws]`.

## Usage

### Maven image

> No longer published — see [No longer published](#no-longer-published).

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

> No longer published — see [No longer published](#no-longer-published).

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

> No longer published — see [No longer published](#no-longer-published).

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

## Notes

- `docker-java`, and the root `docker-maven` / `docker-nodeJS` images, are kept in the repository but no longer published (see [No longer published](#no-longer-published)).
- The `README.md` inside each folder contains more details about the contents of that image.
