# Build Paths

## Platform Build Path

```mermaid
flowchart TD
    classDef stage fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef edge fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    extract[extract-version action]
    platform[build-platform.yml]
    cut[cut-platform.yml]
    injectors[build-multi-arch.yml for injector images]
    base[build-specialised-images.yml for base and base-clear]
    build[build-specialised-images.yml for build and build-clear]
    appTools[build-specialised-images.yml for appserver and tools]
    devenv[build-specialised-images.yml for devenv-2 and devenv-2-clear]
    devdb[build-devdb-qa-images.yml]
    redeploy[redeploy-eks-pod.yml]

    caller --> extract --> platform
    platform --> cut
    platform --> injectors
    cut --> injectors

    platform --> base --> build --> appTools --> devenv
    injectors --> devdb
    appTools --> devdb
    devdb -. optional .-> redeploy

    class platform process;
    class cut,injectors,base,build,appTools,devenv,devdb stage;
    class redeploy edge;
```

### Platform workflow dependency detail

```mermaid
flowchart LR
    base[platform-base and platform-base-clear]
    build[platform-build and platform-build-clear]
    appserver[platform-appserver]
    tools[platform-tools]
    devenv[platform-devenv-2 and platform-devenv-2-clear]

    base --> build
    build --> appserver
    build --> tools
    appserver --> devenv
```

## Standard Module Build Path

```mermaid
flowchart TD
    classDef stage fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    extract[extract-version action]
    module[build-module.yml]
    cut[cut-module.yml]
    injector[build-multi-arch.yml for injector images]
    qaBuild[build-multi-arch.yml for platform-module build image]
    qaComponents[build-multi-arch.yml for appserver and tools images]
    qaApp[build-multi-arch.yml for QA appserver image]
    redeploy[redeploy-eks-pod.yml]
    editions[build-editions.yml]

    caller --> extract --> module
    module --> cut --> injector
    injector --> qaBuild --> qaComponents --> qaApp
    qaComponents -. optional redeploy after QA chain .-> redeploy
    module -. editions input and workflow reference exist, but current implementation does not invoke it .-> editions

    class module process;
    class cut,injector,qaBuild,qaComponents,qaApp stage;
    class redeploy,editions optional;
```

## Independent Module Build Path

```mermaid
flowchart TD
    classDef stage fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    independent[build-independent-module.yml]
    prep[Checkout caller repo, workflow repo, and core repo]
    cut[Generic or custom cut script]
    injector[build-multi-arch.yml for injector image]
    qaBuild[build-multi-arch.yml for platform build image]
    qaComponents[build-multi-arch.yml for appserver and tools images]
    qaApp[build-multi-arch.yml for QA appserver image]
    redeploy[redeploy-eks-pod.yml]

    caller --> independent --> prep --> cut --> injector
    injector --> qaBuild --> qaComponents --> qaApp
    independent -. redeploy job is currently commented out .-> redeploy

    class independent process;
    class prep,cut,injector,qaBuild,qaComponents,qaApp stage;
    class redeploy optional;
```

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