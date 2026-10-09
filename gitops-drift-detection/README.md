# GitOps Drift Detection

## Overview

This project demonstrates GitOps drift detection and reconciliation using Argo CD and declarative Kubernetes manifests.

Argo CD continuously compares the desired state defined in Git with the live state of the Kubernetes cluster. When differences are detected, Argo CD reports the drift and can reconcile the cluster with the desired state according to the configured synchronization policy.

The Argo CD Application is configured to demonstrate automated synchronization, pruning, and self-healing.

## Objective

- Detect differences between Git-managed manifests and live Kubernetes resources.
- Demonstrate automated synchronization and self-healing with Argo CD.
- Demonstrate pruning of resources removed from the Git repository.
- Apply Kubernetes security and resource-management configurations.
- Establish a repeatable workflow for managing Kubernetes deployments through GitOps.

## Technologies

- Kubernetes
- Argo CD
- Kustomize
- YAML

## Project Structure

```text
Gitops-Drift-Detection/
├── README.md
├── implementation.md
├── application.yaml
└── manifests/
    └── app.yaml
```

- `application.yaml` — Defines the Argo CD Application, including the Git repository source, deployment destination, and synchronization policy.
- `manifests/app.yaml` — Contains the Kubernetes workload resources and related configuration.
- `README.md` — Documents project setup, configuration, validation, and operations.
- `implementation.md` — Provides supplementary implementation notes and operational guidance.

## Prerequisites

Before running this project, ensure you have:

- A Kubernetes cluster with appropriate administrative or deployment permissions.
- Argo CD installed and configured to access the target cluster.
- `kubectl` installed and configured for the intended cluster.
- The Argo CD CLI (`argocd`) installed if you want to inspect application status from the terminal.
- A Git repository containing the project manifests and accessible to Argo CD.
- Metrics Server installed if the workload uses a Horizontal Pod Autoscaler.
- A CNI plugin that enforces Kubernetes NetworkPolicy resources if network isolation is required.

## Setup and Configuration

### 1. Configure the Argo CD Application

Open `application.yaml` and review the following settings:

- **Repository URL:** Replace the example repository URL with your actual Git repository.
- **Target revision:** Set the branch, tag, or commit Argo CD should track.
- **Manifest path:** Ensure the path points to the directory containing the workload manifests.
- **Destination cluster:** Configure the intended Kubernetes cluster.
- **Destination namespace:** Set the namespace where the workload should be deployed.
- **Synchronization policy:** Review automated sync, pruning, and self-healing settings before deployment.

Ensure the repository URL, manifest path, and target revision match the actual repository structure.

### 2. Review the Kubernetes Manifests

Inspect the files under `manifests/` before deployment.

The workload configuration includes production-oriented Kubernetes controls such as:

- Namespace Pod Security enforcement.
- Resource quotas.
- A service account with automatic token mounting disabled.
- Default-deny network policies.
- Deployment health probes and resource requests/limits.
- Horizontal Pod Autoscaling (HPA).
- Pod Disruption Budget (PDB).

Confirm that the cluster supports these configurations and that the resource limits, namespace policies, network rules, and replica settings are appropriate for your environment.

### 3. Validate the Configuration

From the project directory, inspect the Application manifest and validate the Kubernetes resources.

```bash
kubectl apply --dry-run=client -f application.yaml
kubectl apply --dry-run=client -f manifests/app.yaml
```

These commands perform client-side validation; they do not guarantee that the resources will pass server-side validation or be successfully reconciled by Argo CD.

If the workload directory contains Kustomize configuration, validate the rendered resources as well:

```bash
kubectl kustomize manifests/
```

Run this command only if `manifests/` contains a valid Kustomize configuration, such as `kustomization.yaml`.

## Deployment

### 1. Apply the Argo CD Application

After reviewing the configuration and confirming the target cluster, apply the Application manifest:

```bash
kubectl apply -f application.yaml
```

This creates or updates the Argo CD Application resource. Argo CD must be installed in the cluster where the Application resource is submitted.

### 2. Inspect Application Status

Check the Application's synchronization and health status:

```bash
argocd app get gitops-drift-detection
```

If necessary, confirm the Application name in `application.yaml` and use that name in the command.

Review the reported sync status, health status, resource differences, and any reconciliation errors.

## Drift Detection and Reconciliation

To demonstrate drift detection:

1. Deploy the application and wait for Argo CD to report a healthy, synchronized state.
2. Make a controlled change to a managed resource directly in the Kubernetes cluster.
3. Inspect the Application in the Argo CD UI or CLI to identify the difference between the live and desired states.
4. Observe whether self-healing restores the resource to its Git-defined state.
5. Remove a test resource from Git and observe the effect of pruning after the updated revision is synchronized.

Use a disposable, non-production environment for these tests. Avoid manually changing production resources or deleting resources that may be shared with other workloads.

**Important:** Automated pruning can delete live resources that are no longer present in Git. Self-healing can also revert intentional manual changes. Review the synchronization policy and resource ownership before enabling these features in a production environment.

## Security and Operational Considerations

- Never commit passwords, access tokens, private keys, or other secrets to Git.
- Use an appropriate secret-management solution, workload identity, or platform-supported credential mechanism.
- Apply least-privilege access controls to Kubernetes, Argo CD, Git repositories, and any external services.
- Review namespace security policies, quotas, network exposure, and resource limits before deployment.
- Protect the tracked Git branch and require appropriate reviews for manifest changes.
- Pin container images to trusted versions or immutable digests where practical.
- Keep credentials, generated state, and local configuration outside version control.
- Review all resource changes before allowing automated synchronization or pruning.
- Monitor application health, synchronization failures, and relevant Kubernetes events.

## Troubleshooting

- **Missing executable:** Install the required tool and confirm it is available on your `PATH`.
- **Repository connection failure:** Verify the repository URL, credentials, repository permissions, and network connectivity from Argo CD.
- **Application not synchronizing:** Inspect the Application's sync status, target revision, manifest path, and Argo CD logs.
- **Permission or credential error:** Verify the active Kubernetes context, service account, RBAC permissions, and configured credentials.
- **Manifest validation failure:** Check YAML syntax, API versions, resource references, and cluster support for the requested resources.
- **HPA not functioning:** Confirm that Metrics Server is installed and that resource requests are defined for the relevant containers.
- **Network policies appear ineffective:** Confirm that the cluster's CNI plugin enforces Kubernetes NetworkPolicy resources and that the selected rules allow the required traffic.
- **Workload health or rollout failure:** Inspect pod events, container logs, readiness and liveness probes, resource limits, and rollout status.
- **Unexpected resource deletion or reversion:** Review the Application's automated sync, prune, and self-heal settings before making further changes.

## Cleanup

Before removing any resources, confirm that they belong exclusively to this project and are not shared with other applications.

To remove the Argo CD Application:

```bash
kubectl delete -f application.yaml
```

Review the Application's deletion and resource-finalizer behavior before running this command. Depending on the configuration, deleting the Application may also delete managed resources.

If you want to preserve the deployed workload, review the Argo CD resource finalizer and deletion settings first. Do not assume that deleting the Application removes only the Argo CD configuration.

## Future Improvements

- Add automated manifest validation and integration tests.
- Introduce policy-as-code checks for Kubernetes security and compliance.
- Pin image versions and automate dependency updates.
- Add monitoring, alerting, and operational dashboards.
- Integrate protected CI/CD workflows for manifest validation and controlled GitOps changes.
- Document environment-specific configuration and rollback procedures before adopting the project in production.

## Implementation Notes

This project provides a demonstration of GitOps-based Kubernetes management. Applying the Application manifest does not automatically configure external cloud accounts, publish container images, push changes to Git, or provision an otherwise unavailable cluster.

Validate the manifests, confirm the deployment target, and review the security and synchronization settings before using the project outside a local or disposable validation environment.