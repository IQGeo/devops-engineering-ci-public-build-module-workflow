# Build Paths

This file now contains the shared build-component diagrams used by multiple process workflows.

For the process-specific visuals, use:

- [BUILD-PLATFORM-WORKFLOW.md](./BUILD-PLATFORM-WORKFLOW.md)
- [BUILD-MODULE-WORKFLOW.md](./BUILD-MODULE-WORKFLOW.md)
- [BUILD-INDEPENDENT-MODULE-WORKFLOW.md](./BUILD-INDEPENDENT-MODULE-WORKFLOW.md)

## Shared Multi-Arch Build Path

```mermaid
flowchart LR
    wf[build-multi-arch.yml]
    lower[Normalize module name]
    arm[build-push action on arm64 runner]
    amd[build-push action on x64 runner]
    manifest[multi-arch action]

    wf --> lower
    lower --> arm
    lower --> amd
    arm --> manifest
    amd --> manifest
```

## Specialised Platform Image Build Path

```mermaid
flowchart LR
    wf[build-specialised-images.yml]
    params[Resolve dockerfile, obfuscation, and platform base]
    push[platform-build-push action]
    manifest[multi-arch action]

    wf --> params --> push --> manifest
```

## Build Output Flow

```mermaid
flowchart TD
    source[Source repositories]
    cut[Cut workflows or cut scripts]
    artifact[GitHub artifact: binaries]
    azure[Azure File Share: cut-binaries]
    archImages[Architecture-specific images in ACR]
    manifests[Multi-arch manifests]
    harbor[Harbor engineering and optional releases paths]
    redeploy[Optional EKS redeploy]

    source --> cut
    cut --> artifact
    cut --> azure
    artifact --> archImages
    archImages --> manifests
    manifests --> harbor
    manifests -. may trigger .-> redeploy
```