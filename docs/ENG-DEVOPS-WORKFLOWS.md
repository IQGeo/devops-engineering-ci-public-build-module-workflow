# DevOps Engineering workflows 

See [README.md](./README.md) for the documentation view and [WORKFLOW-MAPS.md](./WORKFLOW-MAPS.md) for diagrams.

## Summary
We have multiple reusable workflows that we use to create and deploy our products.

### Process workflows.

These top level workflows capture one of our processes

- public-build-platform-workflow\.github\workflows\build-platform.yml
  = The process to build IQGeo Platform
- public-build-module-workflow\.github\workflows\build-module.yml
  - The IQGeo process to build an IQGeo module for IQGeo Platform
- public-build-independent-module-workflow\.github\workflows\build-independent-module.yml
  - A portable process for a Partner to build an IQGeo module for IQGeo Platform

### Supporting workflows

The workflows are used in our processes to do specific multi-step tasks. 

- public-build-platform-workflow\.github\workflows\cut-platform.yml
  - Create and store build artifacts from IQGeo Platform source code.
- public-build-module-workflow\.github\workflows\cut-module.yml
  - Create and store build artifacts from IQGeo module source code.
- public-build-platform-workflow\.github\workflows\build-prereqs-minimal.yml
  - Create prerequisite images for IQGeo Platform.
- public-build-platform-specialised-image-workflow\.github\workflows\build-specialised-images.yml
  - Create the suite of 5 specialised images used to deploy, test, and develop against IQGeo Platform
- public-build-platform-specialised-image-workflow\.github\workflows\build-devdb-qa-images.yml
  - Create an internal test image for IQGeo Platform
- public-build-multi-arch-workflow\.github\workflows\build-multi-arch.yml
  - Create a multi-architecture deliverables for our images
- redeploy-eks-pod\.github\workflows\redeploy-eks-pod.yml
  - Deploy exemplars our our product to an internal, hand-off cluster

### Supporting actions

These actions perform specific tasks

- public-extract-version-action\action.yml
  - Contains logic to determine which tags to use
- public-multi-arch-action\action.yml
  - Create a multi-architecture manifest 
- public-platform-build-push-action\action.yml
  - Pushes IQGeo Platform images to image repositories
- public-build-push-action\action.yml
  - Pushes IQgeo module images to image repositories.