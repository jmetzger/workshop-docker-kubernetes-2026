## Docker / Container Security

**Image-Security**
- Minimal Base Images (distroless, Chainguard, Wolfi, Alpine vs. Debian-slim Trade-offs)
- Multi-Stage Builds, kein Build-Toolchain im Runtime-Image
- Image Scanning: Trivy, Grype, Snyk – CI-Integration
- SBOM-Generierung (Syft) und Vulnerability-Tracking
- Supply Chain: Image Signing mit cosign/Sigstore, Verifikation per Policy

**Dockerfile Best Practices**
- `USER` non-root, fixe UID/GID
- `COPY --chown`, keine Secrets in Layern (Buildkit Secrets/SSH)
- `.dockerignore`, reproducible builds, Pinning per Digest statt Tag
- HEALTHCHECK, sinnvoller ENTRYPOINT (kein Shell-Form)

**Runtime**
- Capabilities droppen (`--cap-drop=ALL`, gezielt zurückgeben)
- `--read-only` root fs + tmpfs Mounts
- seccomp/AppArmor Profile, no-new-privileges
- User Namespaces, Rootless Docker / Podman als Alternative
- Resource Limits (cgroups) als Security-Maßnahme (DoS-Schutz)

**Container Escapes – konkrete CVE-Beispiele**
- CVE-2019-5736 (runC overwrite)
- CVE-2022-0185 (fsconfig)
- CVE-2024-21626 "Leaky Vessels" (runC fd leak)
- Klassiker: privileged container, Docker socket mount, hostPath
