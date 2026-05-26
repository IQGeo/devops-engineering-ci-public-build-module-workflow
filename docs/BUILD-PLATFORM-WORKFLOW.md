# Build Platform Workflow Visuals

## Scope

This document shows the visual overview and top-level architecture for `devops-engineering-ci-public-build-platform-workflow/.github/workflows/build-platform.yml`.

## Visual overview

```mermaid
flowchart LR
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    extract[Extract version action]
    platform[Build platform workflow]
    cut[Cut platform workflow]
    injectors[Build multi-arch workflow\nfor injector images]
    specialised[Build specialised images workflow]
    devdb[Build DevDB QA images workflow]
    redeploy[Redeploy EKS pod workflow]

    caller --> extract --> platform
    platform --> cut
    platform --> injectors
    platform --> specialised
    platform --> devdb
    devdb -. optional .-> redeploy

    class platform process;
    class cut,injectors,specialised,devdb support;
    class redeploy optional;
```

## Top-level architecture

```mermaid
flowchart TD
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef action fill:#fef3c7,stroke:#b45309,stroke-width:2px,color:#451a03;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    extract[extract-version action]

    subgraph processFlow[Process workflow]
        platform[build-platform.yml]
    end

    subgraph supportFlow[Supporting workflows]
        cut[cut-platform.yml]
        multiArch[build-multi-arch.yml]
        specialised[build-specialised-images.yml]
        devdb[build-devdb-qa-images.yml]
        prereqs[build-prereqs-minimal.yml]
    end

    subgraph actions[Supporting actions]
        buildPush[build-push action]
        platformBuildPush[platform-build-push action]
        multiArchAction[multi-arch action]
    end

    redeploy[redeploy-eks-pod.yml]

    caller --> extract --> platform
    platform --> cut
    platform --> multiArch
    platform --> specialised
    platform --> devdb
    platform -. same repo support, not directly invoked .-> prereqs
    devdb -. optional .-> redeploy

    multiArch --> buildPush
    multiArch --> multiArchAction
    specialised --> platformBuildPush
    specialised --> multiArchAction
    devdb --> platformBuildPush
    devdb --> multiArchAction

    class platform process;
    class cut,multiArch,specialised,devdb,prereqs support;
    class extract,buildPush,platformBuildPush,multiArchAction action;
    class redeploy optional;
```