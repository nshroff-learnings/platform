# Platform Control Plane

This folder contains the platform-layer implementation for learning a production-style EKS operating model.

The intended split is:

- Terraform in `infra/` provisions AWS foundations: VPC, IAM, ECR, Secrets Manager metadata, platform EKS, and workload EKS.
- Argo CD runs in the platform EKS cluster and syncs platform configuration from Git.
- Crossplane runs in the platform EKS cluster and exposes a small platform API named `PlatformStack`.
- `PlatformStack` configures a target workload EKS cluster with common add-ons: Argo CD, Argo Rollouts, Istio, Kyverno, Calico, and optional Twistlock/Prisma Cloud Defender.

## Architecture

```text
Git repository
  |
  | Argo CD sync
  v
Platform EKS cluster
  - Argo CD
  - Crossplane
  - provider-kubernetes
  - provider-helm
  - platform.example.com/PlatformStack API
  |
  | Crossplane provider credentials
  v
Workload EKS cluster
  - argocd
  - argo-rollouts
  - istio-system
  - twistlock
  - baseline Calico network policies
```

## Folder Layout

```text
platform/
  argocd/
    root-app.yaml
    apps/
    values/
  crossplane/
    config/
    apis/
  docs/
  examples/
```

## Bootstrap Flow

1. Use the `aws_infra` Terraform workflow to create platform and workload EKS clusters.
2. Run the `bootstrap` Terraform layer. It installs Argo CD on the platform EKS cluster and creates the root Argo CD Application.
3. Argo CD installs Crossplane and syncs Crossplane providers, functions, XRDs, and compositions from this repo.
4. Provide the target workload cluster kubeconfig secret through your approved secret process. Use `examples/target-cluster-secret.example.yaml` only as the shape.
5. Argo CD syncs a `PlatformStack` from `examples/`.
6. Crossplane creates provider configs, namespaces, Helm releases, and baseline policies on the target workload cluster.

See `docs/flow.md` for the full no-manual flow.

## Why This Pattern

This is a good learning setup because it shows the real separation used by many platform teams:

- Terraform handles cloud infrastructure lifecycle and remote state.
- Argo CD handles GitOps delivery and audit history.
- Crossplane exposes higher-level platform APIs and reconciles platform intent.
- Workload clusters receive add-ons without engineers running `helm install` or `kubectl apply` manually against the target cluster.

## Important Notes

- Do not commit real kubeconfigs, Prisma/Twistlock tokens, AWS keys, or Helm repository credentials.
- Pin chart and package versions before using this against a real account.
- The Twistlock chart is vendor/private in most companies. The composition supports it, but you must provide your real chart repository, chart name, chart version, console address, and credentials handling.
- The example uses a kubeconfig secret for readability. For production, prefer short-lived credentials or a tightly scoped management role where your security model supports it.


