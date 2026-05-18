# devops-engineering-ci-public-build-module-workflow

Reusable GitHub Actions workflow for building IQGeo modules and, when required, producing platform-with-module QA images.

## Overview

This repository provides the standard IQGeo module build process. It is intended to be called from a product or module repository after version metadata has already been derived, typically by `devops-engineering-ci-public-extract-version-action`.

The workflow does four main things:

1. Normalizes the requested module list into a build matrix.
2. Cuts module binaries from source using `cut-module.yml`.
3. Builds module injector images through the shared multi-architecture workflow.
4. Optionally builds platform-with-module QA images.

For inbound and outbound dependency relationships, see [docs/WHO-CALLS-WHAT.md](docs/WHO-CALLS-WHAT.md).

The wider reusable workflow documentation for this workspace lives under [docs/README.md](docs/README.md).

## Workflows

### `.github/workflows/build-module.yml`

This is the main reusable entry point.

Key responsibilities:

- Accepts the target version, module list, build id, tags, and release intent.
- Calls `.github/workflows/cut-module.yml` to package source artifacts.
- Calls `IQGeo/devops-engineering-ci-public-build-multi-arch-workflow/.github/workflows/build-multi-arch.yml@main` to build injector images.
- Optionally builds platform build, appserver, tools, and QA appserver images for the module.
- Can call the EKS redeploy workflow when configured.
- Exposes an `include_editions` input, but the current workflow implementation does not invoke the editions reusable workflow.

Typical flow:

```text
caller workflow
	-> build-module.yml
		 -> cut-module.yml
		 -> build-multi-arch.yml for injector images
		 -> build-multi-arch.yml for QA images
		 -> optional redeploy-eks-pod.yml
```

### `.github/workflows/cut-module.yml`

Packages module artifacts for later Docker builds.

Key responsibilities:

- Checks out the caller repository and `IQGeo/myworld-product-core`.
- Places the module source into the expected platform modules directory.
- Runs the module cut script from the module repository.
- Uploads the generated binaries both to Azure File Share and as a short-lived GitHub artifact.

## Important inputs

The main workflow expects these notable inputs:

- `version`: module version being built.
- `modules`: comma-separated module list. The first module is also used when naming QA images.
- `release_modules`: modules eligible for release repository retagging.
- `shortened_version`: version segment used for language pack lookup.
- `tags`: comma-separated tags to apply to the final multi-arch images.
- `build_id`: unique identifier used for architecture-specific image tags.
- `is_release`: controls whether release repositories are updated.
- `build_qa_images`: toggles platform-with-module QA image creation.
- `namespace` and `pod_name`: optional inputs for post-build redeploy.

## How this repo fits the wider build stack

- Upstream: caller workflows and the extract-version action decide the version and tags.
- Downstream: this workflow relies on the shared multi-arch workflow and build actions to produce images.
- Adjacent: the central system view is documented in `docs/README.md`.

## When to use this workflow

Use this repository when the module follows the standard IQGeo module layout and should be built as part of the normal platform-aligned module process.

For independently versioned partner modules or modules that need a generic/custom cut script wrapper, use `devops-engineering-ci-public-build-independent-module-workflow` instead.
