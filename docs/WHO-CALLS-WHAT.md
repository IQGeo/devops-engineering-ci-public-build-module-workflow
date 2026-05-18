# Who Calls What

## Scope

This document shows the inbound and outbound workflow dependencies for `devops-engineering-ci-public-build-module-workflow`.

## Matrix

| Direction | Component | Relationship | Notes |
| --- | --- | --- | --- |
| Called by | Caller workflow or product pipeline | Inbound | Usually receives `version`, `tags`, `build_id`, and `is_release` after `devops-engineering-ci-public-extract-version-action` runs upstream |
| Calls | `.github/workflows/cut-module.yml` | Direct | Produces module cut artifacts |
| Calls | `devops-engineering-ci-public-build-multi-arch-workflow/.github/workflows/build-multi-arch.yml` | Direct | Builds injector images for the requested module set |
| Calls | `devops-engineering-ci-public-build-multi-arch-workflow/.github/workflows/build-multi-arch.yml` | Direct | Reused again for platform build, appserver, tools, and QA appserver images when `build_qa_images=true` |
| Calls | `devops-engineering-ci-redeploy-eks-pod/.github/workflows/redeploy-eks-pod.yml` | Optional | Invoked when `namespace`, `pod_name`, and `build_qa_images` are all set |
| References | `devops-engineering-ci-public-editions-workflow/.github/workflows/build-editions.yml` | Inactive | The workflow defines an editions reference and input, but does not currently invoke it |

## Notes

- The same shared multi-arch workflow is used for both injector builds and the module-specific QA image chain.
- Editions are represented here as a reference rather than an active dependency because the current workflow file does not invoke that reusable workflow.