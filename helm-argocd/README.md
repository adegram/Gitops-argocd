# Helm Argocd

## Overview

An Argo CD Application that deploys a Helm chart from Git while leaving releases reconciled by Argo CD.

## Objective and design

The chart contains configurable values; the Application declares the Git source, chart path, destination, and sync behavior.

The chart runs an unprivileged workload with restricted Pod Security labels, a namespace quota, disabled service-account token mount, read-only root filesystem, health probes, CPU/memory HPA, a PDB, and a namespace-local NetworkPolicy. Set `image.digest` for immutable image promotion. Metrics Server is required for HPA metrics, and a NetworkPolicy-capable CNI is required for policy enforcement.

## Architecture and flow

The chart contains configurable values; the Application declares the Git source, chart path, destination, and sync behavior.

## Technologies

Helm 3; Kubernetes cluster with Argo CD; configured repository URL and destination namespace

## Project structure

- `application.yaml`
- `chart/Chart.yaml`
- `chart/templates/deployment.yaml`
- `chart/templates/service.yaml`
- `chart/values.yaml`

## Prerequisites

Helm 3; Kubernetes cluster with Argo CD; configured repository URL and destination namespace

## Setup and configuration

Use the commands below from this project directory unless a path is stated. Keep local credentials and generated state outside version control. Review every example value and replace reserved example domains, CIDRs, account IDs, repository owners, and image names before connecting a real environment.

## Validate and install

```bash
helm lint chart
helm template portfolio-web chart --namespace portfolio
kubectl apply -f application.yaml
```

Update the repository URL, target revision, cluster destination and namespace in the Application before adding it to an Argo CD control plane. Inspect `syncPolicy` and pruning behavior first: automated prune can remove live objects that are no longer in Git.

## Security and operations

- No secrets belong in Git. Use environment-specific secret stores and the cloud/CI credential mechanisms described above.
- Review IAM, network exposure, branch protection, and resource ownership before using a real account or cluster.
- Preserve logs and build artifacts only as long as operationally necessary.

## Validation

Run the local checks shown above before opening a pull request. The repository-level implementation notes are supplemental; this README describes the project as it exists now. Cloud deployment, image publication, and cluster rollouts require the external account, agent, registry, or cluster described under prerequisites.

## Troubleshooting

- **Missing executable/tool:** install the named prerequisite and confirm it is on `PATH`.
- **Permission or credential error:** verify the correct account/role/namespace and least-privilege policy; never paste a token into a config file.
- **Example placeholder fails:** substitute a real value in a local ignored file or repository/environment variable.
- **Health/rollout failure:** inspect the exact pod/container events, logs, and readiness state before retrying.

## Cleanup and cost

Remove local containers/processes created by the run command. For cloud or cluster examples, inspect the plan or rendered manifests first and remove only resources owned by this project. Some production-style settings intentionally enable deletion protection or retain state; use the documented recovery and retention policy before cleanup.

## Future improvements

Add project-specific integration tests, pinned image digests and dependency updates, automated policy checks, and operational dashboards/alerts once a real deployment target is configured. Avoid enabling external publish/deploy stages until repository variables, protected environments, IAM trust, branch rules and rollback ownership are in place.
