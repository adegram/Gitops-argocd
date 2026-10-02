# Implementation Notes

Argo CD-managed Helm chart with values stored in Git for repeatable deployment.

## Files and operation

See the project files in this directory for implementation details. The existing `README.md` is intentionally preserved. Update deployment-specific placeholders and environment values before using this project outside a local validation environment.

## Run or validate

```sh
Replace the repository owner in application.yaml. Validate the chart with helm lint chart and helm template demo chart.
```

## Security and operations

- No credentials, private keys, tokens, or passwords are stored in this project. Use your platform's secret store or workload identity.
- Review cloud resource costs, IAM permissions, network exposure, and the generated plan before provisioning infrastructure.
- Use least-privilege credentials and a disposable non-production environment for demonstrations.
- Cloud deployment, infrastructure apply, and Git push are not performed by these implementation files automatically.
