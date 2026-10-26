# box1-workloads-dev

Development workload desired state for Box1. Extracted from
`box1-org/box1-gitops` commit `f682e73b70a6bd56ac99c2f4787340b3afd40c19` on October 25, 2026.
Historical workload changes remain available in that repository's Git history.

- `applications/`: Argo CD Applications for development services.
- `workloads/<service>/`: namespace, runtime resources, onboarding permissions,
  and immutable image digest deployed by each Application.
- `.tekton/`: pull request validation using platform-owned Tasks in `box1-ci`.

`box1-gitops/applications/workloads-dev.yaml` installs the parent Application
`box1-workloads-dev`, which discovers `applications/` here. Platform operators,
shared Tekton Tasks, Developer Hub, and cluster bootstrap stay in `box1-gitops`.

Service repositories keep their original `deploy/` manifests as source templates.
Argo CD deploys this repository's workload state. Onboarding creates one
Application plus its workload files here; image promotion changes only the image
digest and build provenance here after required checks succeed.

## Validation

Run `kubectl kustomize workloads/<service>` locally. Pull requests run
`box1-phase1-smoke-` and `box1-gitops-validate-` checks through Pipelines as Code.
Promotion verifies successful checks from the `box1-pipelines` GitHub App on the
exact proposed commit before automatically merging development changes.

## Future environments

Stage and production can use `box1-workloads-stage` and `box1-workloads-prod`
with the same `applications/` and `workloads/` layout and independent parent
Applications, namespaces, access, and review policies. They are not provisioned.
Promotion should copy a tested immutable image digest and its source/build
provenance between environment repositories through reviewed pull requests;
it should not rebuild the image. Record the upstream environment commit and
verification evidence in each promotion. Require stage/prod approval and tests
before merging; use Git reverts to restore a previous tested digest.

## Access

The repository is public, matching the original platform repository. Argo CD
reads it anonymously over HTTPS. The existing Box1 GitHub Apps handle onboarding
and CI; application PAC
tokens are explicitly scoped to their source repository and this repository.
No GitHub App key or token is stored in Git.
