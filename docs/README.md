# DevOps Engineering workflows

## Executive summary

This workspace contains the reusable GitHub Actions components used to build IQGeo Platform, standard IQGeo modules, and independently versioned partner modules.

The build system is organized into layers:

1. Caller repositories decide when to run a build and usually determine version metadata first.
2. Process workflows orchestrate complete build paths.
3. Supporting workflows handle reusable multi-step jobs such as cutting source artifacts and building multi-architecture images.
4. Composite actions perform the low-level Docker build, push, manifest, and retag operations.
5. Optional deployment and scanning components sit adjacent to the core build graph.

For visual views of the same relationships, use [WORKFLOW-MAPS.md](./WORKFLOW-MAPS.md).
For the short outline view, use [ENG-DEVOPS-WORKFLOWS.md](./ENG-DEVOPS-WORKFLOWS.md).

## Summary

This workspace contains the reusable GitHub Actions building blocks used to build IQGeo Platform, IQGeo modules, independently versioned partner modules, and the images that support deployment and testing.

The repositories fit together as a layered system:

1. Caller repositories decide when to run a build and usually derive version metadata.
2. Process workflows orchestrate a full product build.
3. Supporting workflows handle repeated multi-step jobs such as cutting binaries or building multi-architecture images.
4. Supporting actions perform the lowest-level image build, tagging, and retagging work.
5. An optional deployment workflow redeploys EKS pods after new QA images are available.

## How the pieces fit together

### End-to-end flow

```text
Caller repo / product pipeline
  -> extract-version action
  -> process workflow
     -> cut workflow
     -> build-multi-arch workflow
        -> build-push action
        -> multi-arch action
     -> specialised image workflow or editions workflow
        -> platform-build-push action or build-multi-arch workflow
     -> optional redeploy-eks-pod workflow
```

### Common responsibilities by layer

- Caller repositories decide the trigger, source ref, and release intent.
- `devops-engineering-ci-public-extract-version-action` normalizes tags, release state, namespace hints, and downstream image tags.
- Top-level reusable workflows orchestrate build order, dependency sequencing, and which images are produced.
- Supporting reusable workflows package artifacts or coordinate fan-out across amd64 and arm64 builders.
- Composite actions perform the actual image build/push and manifest/retag operations.

## Recommended documentation entry points

- Use this document for the system view.
- Use [WORKFLOW-MAPS.md](./WORKFLOW-MAPS.md) as the visual index.
- Use [REPOSITORY-MAP.md](./REPOSITORY-MAP.md) for component and dependency maps.
- Use [BUILD-PATHS.md](./BUILD-PATHS.md) for the main build path diagrams.
- Use each repository README for inputs, outputs, and usage examples of that specific workflow or action.