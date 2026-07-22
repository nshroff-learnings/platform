# PlatformStack Examples

`dev-platform-stack.yaml` is an example claim for a workload EKS cluster named `eks-dev-01`.

Before syncing it:

1. Replace placeholder chart versions if your Helm repositories use different versions.
2. Create the target kubeconfig secret in `crossplane-system`.
3. Decide whether Twistlock should be enabled. Keep it disabled until your private chart repository and credentials are ready.
4. Add each namespace that should receive baseline Calico policies under `calico.protectedNamespaces`.

The example intentionally uses a claim, not direct provider resources. That is the platform API experience you are trying to learn.


