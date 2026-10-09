# Security Policy

## Supported Versions

This project publishes a rolling container image built from the `main` branch. Only the most
recently published image is supported with security updates: the weekly scheduled build bakes
a fresh disk with the current Ubuntu security updates and re-runs the vulnerability gate, and
the pinned NVIDIA driver is bumped when NVIDIA publishes a security bulletin for the 580 branch.

| Version | Supported |
| ------- | --------- |
| `latest` / `main` (newest digest) | :white_check_mark: |
| `sha-*` builds, `v*` tags, older digests | :x: |

Kernel and driver fixes reach a VM only by pulling a newer image: the kernel is pinned inside
the image so the VM can never boot a kernel that lacks the NVIDIA module. Userland packages
inside a running VM keep receiving Ubuntu security updates through `unattended-upgrades`.

## Reporting a Vulnerability

Please report suspected vulnerabilities **privately** — do not open a public issue.

Use GitHub's private vulnerability reporting for this repository:
<https://github.com/igladun/ubuntu-nvidia-containerdisk/security/advisories/new>

Include the affected image digest or tag, a description of the issue, and steps to reproduce.
You can expect an acknowledgement within 7 days. Once a fix is available a new image is
published and the advisory is disclosed publicly.

## Supply-chain assurances

Every published image ships with:

- A signed SLSA build provenance attestation proving the image was built by this
  repository's workflow.
- A CycloneDX SBOM attested against the image digest. The image is pushed by digest first
  and tags are applied only after both attestations have been pushed.
- Trivy vulnerability scanning of the baked disk — publishing is blocked on fixable
  CRITICAL/HIGH CVEs in OS packages and bundled binaries. Scoped, auto-expiring exceptions
  would live in [`.trivyignore`](.trivyignore); there are currently none.
- A weekly driver-freshness check that fails when NVIDIA's apt repo carries a newer 580.x
  driver than the pinned one (the vulnerability scanner has no advisory data for NVIDIA's
  packages, so this is the only signal).
- A fresh bake on every scheduled build and on every `v*` tag — the Actions cache is never
  trusted for a release artifact, and a failed scheduled build opens an issue.
- GitHub Actions pinned to full-length commit SHAs and the Dockerfile base image pinned by
  digest, both kept current by Dependabot.
- The upstream Ubuntu checksum file verified against Canonical's cloud-image signing key, and
  the cloud image verified against that checksum.
- No SSH host keys, users, passwords or authorized keys baked into the image.
- CodeQL analysis of the CI workflows and OpenSSF Scorecard monitoring.

Consumers can verify both attestations against the registry; see the
[README's Security section](README.md#security) for the `gh attestation` commands.
