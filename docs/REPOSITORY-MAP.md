# Repository Map

## Legend

- Process workflows are the main reusable entry points.
- Supporting workflows are called by the process workflows.
- Composite actions perform the low-level build, tagging, and manifest work.
- Deployment and scanning sit adjacent to the build path.

## Repository Map

| Repository | Type | Main entrypoint | Primary role |
| --- | --- | --- | --- |
| `devops-engineering-ci-public-build-platform-workflow` | Process workflow repo | `.github/workflows/build-platform.yml` | Orchestrates platform builds |
| `devops-engineering-ci-public-build-module-workflow` | Process workflow repo | `.github/workflows/build-module.yml` | Orchestrates standard module builds |
| `devops-engineering-ci-public-build-independent-module-workflow` | Process workflow repo | `.github/workflows/build-independent-module.yml` | Orchestrates independently versioned module builds |
| `devops-engineering-ci-public-build-multi-arch-workflow` | Supporting workflow repo | `.github/workflows/build-multi-arch.yml` | Builds amd64 and arm64 images, then publishes a manifest |
| `devops-engineering-ci-public-build-platform-specialised-image-workflow` | Supporting workflow repo | `.github/workflows/build-specialised-images.yml` | Builds specialised platform image families |
| `devops-engineering-ci-public-build-platform-specialised-image-workflow` | Supporting workflow repo | `.github/workflows/build-devdb-qa-images.yml` | Builds DevDB QA image families |
| `devops-engineering-ci-public-build-platform-workflow` | Supporting workflow repo | `.github/workflows/cut-platform.yml` | Produces platform cut artifacts |
| `devops-engineering-ci-public-build-platform-workflow` | Supporting workflow repo | `.github/workflows/build-prereqs-minimal.yml` | Builds minimal multi-arch pre-req images |
| `devops-engineering-ci-public-build-module-workflow` | Supporting workflow repo | `.github/workflows/cut-module.yml` | Produces standard module cut artifacts |
| `devops-engineering-ci-public-editions-workflow` | Supporting workflow repo | `.github/workflows/build-editions.yml` | Builds edition image families via the shared multi-arch flow |
| `devops-engineering-ci-public-extract-version-action` | Composite action repo | `action.yml` | Normalizes version and tags before build |
| `devops-engineering-ci-public-build-push-action` | Composite action repo | `action.yml` | Builds one module-style architecture-specific image |
| `devops-engineering-ci-public-platform-build-push-action` | Composite action repo | `action.yml` | Builds one platform architecture-specific image |
| `devops-engineering-ci-public-multi-arch-action` | Composite action repo | `action.yml` | Creates and retags multi-arch manifests |
| `devops-engineering-ci-redeploy-eks-pod` | Deployment workflow repo | `.github/workflows/redeploy-eks-pod.yml` | Forces pod rollout on EKS |
| `devops-engineering-ci-public-trivy-scan-action` | Scanning repo | not present in current snapshot | Intended post-build image scanning |

## Visual overview

```mermaid
flowchart TB
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef action fill:#fef3c7,stroke:#b45309,stroke-width:2px,color:#451a03;
    classDef edge fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;
    classDef scan fill:#ffe4e6,stroke:#be123c,stroke-width:2px,color:#4c0519;

    caller[Caller repo or product pipeline]
    extract[Extract version action]

    platform[Build platform workflow]
    module[Build module workflow]
    independent[Build independent module workflow]

    cutPlatform[Cut platform workflow]
    cutModule[Cut module workflow]
    multiArchWF[Build multi-arch workflow]
    specialised[Build specialised images workflow]
    devdbQA[Build DevDB QA images workflow]
    editions[Build editions workflow]

    buildPush[Build push action]
    platformBuildPush[Platform build push action]
    multiArchAction[Multi-arch action]

    redeploy[Redeploy EKS pod workflow]
    trivy[Trivy scan action]

    caller --> extract
    extract --> platform
    extract --> module
    caller --> independent

    platform --> cutPlatform
    platform --> multiArchWF
    platform --> specialised
    platform --> devdbQA
    platform -. optional .-> redeploy

    module --> cutModule
    module --> multiArchWF
    module -. reference only .-> editions
    module -. optional .-> redeploy

    independent --> multiArchWF
    independent -. optional QA chain .-> multiArchWF
    independent -. inactive .-> redeploy

    multiArchWF --> buildPush
    multiArchWF --> multiArchAction
    specialised --> platformBuildPush
    specialised --> multiArchAction
    devdbQA --> platformBuildPush
    devdbQA --> multiArchAction

    platform -. post build .-> trivy
    module -. post build .-> trivy
    independent -. post build .-> trivy

    class platform,module,independent process;
    class cutPlatform,cutModule,multiArchWF,specialised,devdbQA,editions support;
    class buildPush,platformBuildPush,multiArchAction,extract action;
    class redeploy edge;
    class trivy scan;
```

## Top-Level Architecture

```mermaid
flowchart TD
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef action fill:#fef3c7,stroke:#b45309,stroke-width:2px,color:#451a03;
    classDef edge fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;
    classDef scan fill:#ffe4e6,stroke:#be123c,stroke-width:2px,color:#4c0519;

    caller[Caller repo or product pipeline]
    extract[extract-version action]

    subgraph process[Process workflows]
        platform[build-platform.yml]
        module[build-module.yml]
        independent[build-independent-module.yml]
    end

    subgraph support[Supporting workflows]
        cutPlatform[cut-platform.yml]
        cutModule[cut-module.yml]
        multiArchWF[build-multi-arch.yml]
        specialised[build-specialised-images.yml]
        devdbQA[build-devdb-qa-images.yml]
        prereqs[build-prereqs-minimal.yml]
        editions[build-editions.yml]
    end

    subgraph actions[Composite actions]
        buildPush[build-push action]
        platformBuildPush[platform-build-push action]
        multiArchAction[multi-arch action]
    end

    redeploy[redeploy-eks-pod.yml]
    trivy[trivy scan action]

    caller --> extract
    extract --> platform
    extract --> module
    caller --> independent

    platform --> cutPlatform
    platform --> multiArchWF
    platform --> specialised
    platform --> devdbQA
    platform -. may use .-> prereqs
    platform -. optional .-> redeploy

    module --> cutModule
    module --> multiArchWF
    module -. reference exists .-> editions
    module -. optional .-> redeploy

    independent --> multiArchWF
    independent -. optional QA builds .-> multiArchWF
    independent -. redeploy currently disabled .-> redeploy

    multiArchWF --> buildPush
    multiArchWF --> multiArchAction

    specialised --> platformBuildPush
    specialised --> multiArchAction

    devdbQA --> platformBuildPush
    devdbQA --> multiArchAction

    platform -. post build .-> trivy
    module -. post build .-> trivy
    independent -. post build .-> trivy

    class platform,module,independent process;
    class cutPlatform,cutModule,multiArchWF,specialised,devdbQA,prereqs,editions support;
    class buildPush,platformBuildPush,multiArchAction,extract action;
    class redeploy edge;
    class trivy scan;
```

## Notes On Current Gaps

- `devops-engineering-ci-public-trivy-scan-action` is represented as an adjacent component because the current workspace snapshot does not include its `action.yml`.
- `devops-engineering-ci-public-build-module-workflow` exposes an editions-related input and reference, but the current workflow implementation does not invoke the editions workflow.
- `devops-engineering-ci-public-build-independent-module-workflow` contains a commented-out redeploy job, so the redeploy edge is shown as inactive.