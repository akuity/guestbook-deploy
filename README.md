# Guestbook Deploy Repository

## Overview

This repository is a reference GitOps deployment repository containing multiple deploy environments. This example utilizes the technique of "rendered YAML branches" for GitOps deployment.

## Details

The technique of using "rendered YAML branches" removes the responsibility of config templating from Argo CD, to the CI/CD pipeline. In this example, a [GitHub Action](https://github.com/akuity/guestbook-deploy/blob/main/.github/workflows/render-manifests.yml) automates the config management templating (e.g. `kustomize build`) such that fully rendered Kubernetes manifests are outputted to an environment specific branch (e.g. `env/stage`, `env/prod`). Argo CD applications are configured to deploy the manifests from the **environment branch**, as opposed to a directory in the `main` branch.

The application source repository is located at https://github.com/akuity/guestbook and has a [CI/CD Pipeline](https://github.com/akuity/guestbook/blob/main/.github/workflows/ci-cd.yml) which builds new container images and automatically commits the new image tags to the [kustomize environments](https://github.com/akuity/guestbook-deploy/tree/main/env) contained in this repository.

## Why this approach?

### Advantages:
* Easily understandable change history/diff - change is not obfuscated by config tooling
* Use different policies per environment - e.g. automated commit/deployment to dev/stage, PR approval process and protected branch for prod
* Upgrading Argo CD + baked-in toolchain (kustomize) is no longer a risk - templating done in CI, not by Argo CD
* Better security - No longer at risk from vulnerabilities in tooling (helm, kustomize)
* Safer change management - Change to a kustomize base will not immediately affect all environments
* Improved Argo CD performance - expensive templating process (kustomize build) is no longer performed by Argo CD 

### Disadvantages:
* Additional CI automation requirements (e.g. GitHub action)
* Does not support tools which render plain-text secrets (e.g. Kustomize + SOPS)

## Environments

| Environment | Status |
|----|----|
| [Dev](https://github.com/akuity/guestbook-deploy/tree/env/dev) | [![App Status](https://cd.demo.akuity.io/api/badge?name=guestbook-dev&revision=true)](https://cd.demo.akuity.io/applications/guestbook-dev) |
| [Stage](https://github.com/akuity/guestbook-deploy/tree/env/stage) | [![App Status](https://cd.demo.akuity.io/api/badge?name=guestbook-stage&revision=true)](https://cd.demo.akuity.io/applications/guestbook-stage) |
| [Prod](https://github.com/akuity/guestbook-deploy/tree/env/prod) | [![App Status](https://cd.demo.akuity.io/api/badge?name=guestbook-prod&revision=true)](https://cd.demo.akuity.io/applications/guestbook-prod) |

## Configuration Management

This respository utilizes kustomize for configuration management of multiple environments. A common kustomize base is shared between all environments.  Environments are organized into individual `env` directories, structured in the following manner:

```
.
├── base
│   ├── guestbook-deploy.yaml
│   ├── guestbook-ing.yaml
│   ├── guestbook-svc.yaml
│   └── kustomization.yaml
└── env
    ├── dev
    │   └── kustomization.yaml
    ├── prod
    │   └── kustomization.yaml
    └── stage
        └── kustomization.yaml
```

Any changes to the kustomize configuration in `main` branch will result in the following:
* For the `env/dev` and `env/stage` branches, the change will be automatically pushed to the environment branch resulting in immediate deployment
* For the `env/prod` branch, a PR will be created against the branch for manual approval

Details of how this is accomplished can be seen in the [GitHub Action](https://github.com/akuity/guestbook-deploy/blob/main/.github/workflows/render-manifests.yml).

The source code for this repository is located at https://github.com/akuity/guestbook-deploy.
