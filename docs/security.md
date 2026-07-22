# Security Notes

## Credential Boundaries

The platform cluster becomes highly privileged because Crossplane can connect to target clusters. Treat it like a production management plane.

Recommended controls:

- Restrict access to the platform cluster.
- Use dedicated target-cluster service accounts for Crossplane.
- Limit provider credentials to the namespaces and resource types required by the platform API.
- Store real kubeconfigs and vendor credentials outside Git.
- Rotate target-cluster credentials.
- Enable audit logs on both platform and workload clusters.

## GitOps Safety

Use pull requests for every change to:

- XRD schemas.
- Compositions.
- Argo CD Applications.
- PlatformStack instances.
- Helm chart versions.
- Add-on values.

Do not allow direct pushes to the protected branch that Argo CD tracks.

## Argo CD With Crossplane

Argo CD should use annotation-based resource tracking when managing Crossplane resources. Crossplane creates composed resources with owner references and generated names; annotation tracking avoids ownership confusion between Argo CD and Crossplane.

## Twistlock

Twistlock/Prisma Cloud Compute usually requires private chart access and sensitive console credentials. This repo intentionally does not include those secrets. Use a secret manager, External Secrets Operator, sealed secrets, or a private Helm repository credential process approved by your organization.

## Network Policy

The sample Calico policies are intentionally conservative:

- Default deny ingress and egress in selected workload namespaces.
- Allow DNS egress to kube-dns.
- Allow Istio sidecar/control-plane communication only where needed.

Start with a non-critical namespace and add explicit allow policies based on observed application traffic.


