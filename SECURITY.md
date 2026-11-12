# Security policy

## Supported images

Only the images that are still built by the CI workflow receive security fixes, and
only on their most recent tag. Images are rebuilt cache-less on the 1st of every month
(picking up the latest base image, `kubectl`, Helm and Terraform), and on demand when a
critical vulnerability requires it.

| Docker Hub image | Supported |
| --- | --- |
| `shad0wz7/docker-dind`, `docker-dind-aws-cli`, `docker-dind-azure-cli` | :white_check_mark: `latest` / newest `YYYYMMDD` tag |
| `shad0wz7/maven-aws-cli`, `maven-azure-cli`, `maven-gcloud-cli` | :white_check_mark: `latest` / newest `YYYYMMDD` tag |
| `shad0wz7/node-aws-cli`, `node-azure-cli`, `node-gcloud-cli` | :white_check_mark: `latest` / newest `YYYYMMDD` tag |
| Older `YYYYMMDD` or commit-SHA tags of the images above | :x: pull the newest tag instead |
| `shad0wz7/java-aws-terraform` (deprecated) | :x: use `shad0wz7/maven-*-cli` |
| `shad0wz7/maven-terraform`, `shad0wz7/node-aws-terraform` (disabled) | :x: use the single-cloud variants |

## Reporting a vulnerability

**Please do not open a public issue for a vulnerability that is not already public.**

Report it privately through GitHub instead:
**Security** tab → **Report a vulnerability**
(<https://github.com/ShadowZ11/Docker/security/advisories/new>).

This applies in particular to anything specific to this repository, for example:

- a Dockerfile step that downloads or installs something without verifying it
  (every binary must be checksum-verified against its publisher's manifest);
- a way to tamper with what the CI workflow builds or pushes to Docker Hub;
- secrets or credentials exposed in an image, a build log or the workflow.

Please include the affected image(s) and tag or digest, the steps to reproduce, and
the impact you expect.

### Already-public CVEs in bundled packages

A known CVE in a base image or a bundled tool (Debian/Alpine packages, Python
packages, the cloud CLIs, `kubectl`, Helm, Terraform, …) is already public, so it can be
reported as a regular issue using the **Bug report** template. Before doing so, please
check that it still affects the **newest** tag: many of these are fixed by the monthly
rebuild. The repository's Trivy scan results (`CRITICAL` and `HIGH`) are also
published in the Security tab.

Note that a CVE can only be fixed here once the upstream publisher (base image
maintainer or tool vendor) has released a fix.

## What to expect

- Acknowledgement of a private report within **7 days**.
- A fix, or an explanation of why the report will not be fixed, as soon as possible
  depending on severity. Critical issues trigger an out-of-cycle rebuild and push of
  the affected images.
- Credit in the published security advisory, unless you prefer to stay anonymous.
