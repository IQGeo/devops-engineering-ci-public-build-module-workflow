# Build Independent Module Workflow Visuals

## Scope

This document shows the visual overview and top-level architecture for `devops-engineering-ci-public-build-independent-module-workflow/.github/workflows/build-independent-module.yml`.

## Visual overview

```mermaid
flowchart LR
    classDef process fill:#d8f3dc,stroke:#1b4332,stroke-width:2px,color:#081c15;
    classDef support fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#172554;
    classDef optional fill:#f3e8ff,stroke:#7e22ce,stroke-width:2px,color:#3b0764;

    caller[Caller workflow]
    independent[Build independent module workflow]
    cut[Generic or custom cut step]
    injector[Build multi-arch workflow\nfor injector image]
    qa[Build multi-arch workflow\nfor QA image chain]
    redeploy[Redeploy EKS pod workflow]

    caller --> independent --> cut --> injector --> qa
    qa -. optional .-> redeploy

    class independent process;
    class cut,injector,qa support;
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

    subgraph processFlow[Process workflow]
        independent[build-independent-module.yml]
    end

    subgraph supportFlow[Internal setup and reusable workflows]
        cut[generic or custom cut script]
        multiArchInjector[build-multi-arch.yml\nfor injector image]
        multiArchQA[build-multi-arch.yml\nfor platform build, appserver, tools, and QA appserver images]
    end

    subgraph actions[Supporting actions]
        buildPush[build-push action]
        multiArchAction[multi-arch action]
    end

    redeploy[redeploy-eks-pod.yml]

    caller --> independent --> cut
    independent --> multiArchInjector
    independent --> multiArchQA
    multiArchQA -. redeploy job currently commented out .-> redeploy

    multiArchInjector --> buildPush
    multiArchInjector --> multiArchAction
    multiArchQA --> buildPush
    multiArchQA --> multiArchAction

    class independent process;
    class cut,multiArchInjector,multiArchQA support;
    class buildPush,multiArchAction action;
    class redeploy optional;
```