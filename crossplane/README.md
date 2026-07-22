# Crossplane Platform API

This folder contains the Crossplane configuration for the platform API.

## Files

- `config/functions.yaml`: composition functions used by the platform composition.
- `config/providers.yaml`: provider-kubernetes and provider-helm packages.
- `../examples/target-cluster-secret.example.yaml`: example shape for a target cluster kubeconfig secret.
- `apis/platformstack-xrd.yaml`: the `PlatformStack` API schema.
- `apis/platformstack-composition.yaml`: the implementation that renders provider configs, namespaces, Helm releases, and baseline policies.

## PlatformStack

`PlatformStack` is a claim. Platform engineers can create one object per target cluster instead of writing many Helm and Kubernetes manifests manually.

The claim supports:

- Target cluster kubeconfig secret reference.
- Argo CD chart settings.
- Argo Rollouts chart settings.
- Istio sidecar-mode chart settings.
- Optional Twistlock/Prisma Defender chart settings.
- Calico baseline policy namespace list.


