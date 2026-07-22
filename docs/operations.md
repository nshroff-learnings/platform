# Operations

## Day-2 Changes

Common changes should happen through Git:

- Add another workload cluster by adding a new kubeconfig secret and a new `PlatformStack`.
- Change chart versions by editing the `PlatformStack` values.
- Enable or disable add-ons by changing `spec.parameters.addons`.
- Add standard namespaces by extending the Composition.
- Add more baseline policies through provider-kubernetes `Object` templates.

## Upgrade Order

Use this order for safer upgrades:

1. Upgrade platform Argo CD.
2. Upgrade Crossplane core.
3. Upgrade Crossplane functions.
4. Upgrade Crossplane providers.
5. Upgrade platform XRD and Composition.
6. Upgrade workload add-ons through `PlatformStack`.

## Drift Handling

Manual changes in the workload cluster are temporary. Crossplane and Helm provider reconciliation will return managed resources to the declared state.

If a manual hotfix is required:

1. Record the change.
2. Patch Git with the intended state.
3. Let Argo CD and Crossplane reconcile.
4. Remove the manual change if it is no longer needed.

## Failure Checks

Useful commands from the platform cluster:

```powershell
kubectl get applications -n argocd
kubectl get providers.pkg.crossplane.io
kubectl get functions.pkg.crossplane.io
kubectl get providerconfigs
kubectl get platformstacks
kubectl get xplatformstacks
kubectl describe platformstack dev-platform
kubectl get events -A --sort-by=.lastTimestamp
```

Check provider logs:

```powershell
kubectl logs -n crossplane-system deploy/provider-kubernetes
kubectl logs -n crossplane-system deploy/provider-helm
```


