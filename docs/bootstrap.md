# Bootstrap Runbook

This platform repo is not bootstrapped with local `kubectl apply`.

The bootstrap entry point is the Terraform `infra/bootstrap` layer in the separate `aws_infra` repo. That layer:

1. Connects to the platform EKS cluster.
2. Installs Argo CD by Helm.
3. Creates the `platform-root` Argo CD Application.
4. Points `platform-root` at `argocd/apps` in this platform repository.

After that, Argo CD owns platform reconciliation.

## Prerequisites

- Platform EKS cluster exists.
- Workload EKS cluster exists.
- `aws_infra/variables/<env>/bootstrap.tfvars` points to this platform repository.
- The Terraform runner can reach the platform EKS API endpoint.
- Argo CD child Applications in `argocd/apps` point to this repository URL.

## Automated Bootstrap

Run the Terraform workflow from `aws_infra` with:

```text
action = apply
layers = bootstrap
```

For a full environment build, use:

```text
action = apply
layers = all
```

The layer order is:

```text
networking -> iam -> config -> compute -> bootstrap
```

## Target Cluster Credential

Crossplane needs credentials to manage each workload cluster. Do not commit real kubeconfigs.

Use your approved secret process to create a secret shaped like:

```text
examples/target-cluster-secret.example.yaml
```

In production, prefer a dedicated scoped identity over a personal admin kubeconfig.

## PlatformStack

Argo CD syncs `examples/dev-platform-stack.yaml` through the `platform-stacks` Application. Crossplane then reconciles the workload cluster add-ons.

## Verification

From CI or an approved operator workstation, verify through Argo CD first:

```powershell
argocd app get platform-root
argocd app get crossplane
argocd app get crossplane-config
argocd app get platform-api
argocd app get platform-stacks
```

Use direct cluster access only for break-glass troubleshooting.
