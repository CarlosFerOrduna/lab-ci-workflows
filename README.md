# lab-ci-workflows

Reusable GitHub Actions workflows (`on: workflow_call`) shared by every lab repository. Each repository keeps a thin workflow that calls the ones defined here.

## Status

Today: only this README.

Planned:

- PowerShell linting with PSScriptAnalyzer, for the scripts in lab-infra.
- The service pipeline, for lab-processor, lab-producer-a and lab-producer-b:
  - Run the npm scripts (`lint`, `test`) so CI and local work use the same commands.
  - Build the image with BuildKit and push it to the lab registry.
  - Tag the image with the tree hash (`git rev-parse HEAD^{tree}`).
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
