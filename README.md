# lab-ci-workflows

Reusable GitHub Actions workflows (`on: workflow_call`) shared by every lab repository. Each repository keeps a thin workflow that calls the ones defined here.

## Status

Callers pin the major version tag: `uses: CarlosFerOrduna/lab-ci-workflows/.github/workflows/<workflow>.yml@v1`. Third-party actions inside the workflows are pinned by commit SHA.

| Workflow              | Used by                                       | What it does                                                                                                                                                                                        |
| --------------------- | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `powershell-lint.yml` | lab-infra                                     | Runs PSScriptAnalyzer on a folder of PowerShell scripts. Inputs: `path`, `settings`, `analyzer-version`.                                                                                            |
| `container-image.yml` | lab-processor, lab-producer-a, lab-producer-b | Builds the caller's `Dockerfile` and pushes `ghcr.io/<owner>/<repo>:<tree hash>` with the caller's `GITHUB_TOKEN`. Skips the build if that tag exists. Labels the image with its source repository. |

### container-image.yml

The caller decides when it runs and grants the permissions; a reusable workflow can only keep or reduce them:

```yaml
on:
  push:
    branches: [development]
  workflow_dispatch:

permissions:
  contents: read
  packages: write

jobs:
  image:
    uses: CarlosFerOrduna/lab-ci-workflows/.github/workflows/container-image.yml@v1
```

- The tag is the tree hash (`git rev-parse HEAD^{tree}`): it identifies the content, not the commit, so the same files always map to the same image.
- Inside the reusable workflow, `github.repository` and `GITHUB_TOKEN` belong to the caller: each service publishes its own package, never another one's.
- Values reach the shell through `env`, never as `${{ }}` inside `run`, so no expression can inject commands.
- No personal token publishes images.

Planned for the service pipeline:

- Run the npm scripts (`lint`, `test`) before publishing, so CI and local work use the same commands.
- Build with BuildKit.
- On `stage` and `main`, reuse the image built for `development`. Fail if it does not exist and never rebuild.
- Update the image tag in the matching lab-gitops overlay.

## Lab repositories

| Repository                                                              | Role                                                                                                         |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| [lab-infra](https://github.com/CarlosFerOrduna/lab-infra)               | Builds the infrastructure: Hyper-V VMs today, Ansible and k3s next. Holds the repository settings checklist. |
| [lab-gitops](https://github.com/CarlosFerOrduna/lab-gitops)             | Desired state of the clusters. ArgoCD applies whatever is in this repo.                                      |
| [lab-ci-workflows](https://github.com/CarlosFerOrduna/lab-ci-workflows) | Reusable GitHub Actions workflows shared by every lab repo.                                                  |
| [lab-processor](https://github.com/CarlosFerOrduna/lab-processor)       | Shared processor service, namespace `platform`.                                                              |
| [lab-producer-a](https://github.com/CarlosFerOrduna/lab-producer-a)     | Producer service, namespace `producer-a`.                                                                    |
| [lab-producer-b](https://github.com/CarlosFerOrduna/lab-producer-b)     | Producer service, namespace `producer-b`.                                                                    |
