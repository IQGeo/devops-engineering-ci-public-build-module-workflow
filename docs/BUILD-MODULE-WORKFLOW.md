# Build Module Workflow Visuals

## Scope

This document shows the visual overview and top-level architecture for `devops-engineering-ci-public-build-module-workflow/.github/workflows/build-module.yml`.

## Visual overview

```mermaid
flowchart LR
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    extract[Extract version action]
    module[Build module workflow]
    cut[Cut module workflow]
    injector[Build multi-arch workflow\nfor injector images]
    qa[Build multi-arch workflow\nfor QA image chain]
    redeploy[Redeploy EKS pod workflow]
    editions[Build editions workflow]

    caller --> extract --> module
    module --> cut
    module --> injector
    module --> qa
    qa -. optional .-> redeploy
    module -. reference only .-> editions

    class module process;
    class cut,injector,qa support;
    class redeploy,editions optional;
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
        module[build-module.yml]
    end

    subgraph supportFlow[Supporting workflows]
        cut[cut-module.yml]
        multiArchInjector[build-multi-arch.yml\nfor injector images]
        multiArchQA[build-multi-arch.yml\nfor platform build, appserver, tools, and QA appserver images]
        editions[build-editions.yml]
    end

    subgraph actions[Supporting actions]
        buildPush[build-push action]
        multiArchAction[multi-arch action]
    end

    redeploy[redeploy-eks-pod.yml]

    caller --> extract --> module
    module --> cut
    module --> multiArchInjector
    module --> multiArchQA
    module -. reference exists, not currently invoked .-> editions
    multiArchQA -. optional .-> redeploy

    multiArchInjector --> buildPush
    multiArchInjector --> multiArchAction
    multiArchQA --> buildPush
    multiArchQA --> multiArchAction

    class module process;
    class cut,multiArchInjector,multiArchQA,editions support;
    class extract,buildPush,multiArchAction action;
    class redeploy optional;
```