# Platform Architecture

## Responsibilities

Terraform provisions AWS resources and EKS clusters. It should not become the long-term owner of every Kubernetes add-on because that creates noisy plans, large state, and awkward drift handling for resources that are naturally reconciled inside Kubernetes.

Argo CD is the GitOps engine. It watches this repository and applies the desired platform state to the platform EKS cluster.

Crossplane is the platform API layer. It creates Kubernetes and Helm provider resources that point to workload clusters and then reconciles add-ons there.

## Control Plane Cluster

The platform cluster is the management plane. It hosts:

- Argo CD for GitOps sync.
- Crossplane core.
- Crossplane functions for templating and readiness.
- provider-kubernetes for raw Kubernetes objects.
- provider-helm for Helm releases.
- XRDs and Compositions that define the platform API.

## Workload Cluster

The workload cluster hosts the actual application platform add-ons:

- `argocd` namespace for tenant/application GitOps if you want app teams to use local Argo CD.
- `argo-rollouts` namespace for progressive delivery.
- `istio-system` namespace for mesh control plane and ingress gateway.
- `twistlock` namespace for Prisma Cloud Compute Defender.
- Calico `NetworkPolicy` objects for default-deny and DNS allowances.

## Request Flow

```text
Developer changes PlatformStack in Git
  |
Pull request review
  |
Merge
  |
Argo CD syncs PlatformStack to platform cluster
  |
Crossplane sees PlatformStack claim
  |
Composition renders ProviderConfig, Object, and Release resources
  |
provider-kubernetes and provider-helm connect to workload cluster
  |
Namespaces, charts, and policies are reconciled on workload EKS
```

## Why Not Only Terraform

Terraform is still the right tool for AWS foundations: VPC, IAM, EKS, KMS, ECR, backend, and cluster-level AWS add-ons. For in-cluster add-ons, Kubernetes controllers give you continuous reconciliation, health, and drift correction after the initial deployment. That is why this folder places Crossplane and Argo CD above Terraform rather than inside each Terraform root module.


