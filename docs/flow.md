# End-to-End Flow

## 1. AWS Infrastructure Repo

Run the Terraform workflow in `aws_infra`.

Recommended order:

1. `networking apply`
2. `iam apply`
3. `config apply`
4. `compute apply`
5. `bootstrap apply`

The `bootstrap` layer installs Argo CD on the platform EKS cluster and creates the `platform-root` Argo CD Application. No local `kubectl apply` is required.

## 2. Platform Repo

The platform repo contains the Argo CD child apps, Crossplane providers, XRD, Composition, and `PlatformStack` claims.

After Terraform bootstrap, Argo CD owns the platform lifecycle:

```text
platform-root
  -> crossplane
  -> crossplane-config
  -> platform-api
  -> platform-stacks
```

A change to this repo is reviewed through pull request. After merge, Argo CD can auto-sync, or the `Platform Components` workflow can run `argocd app sync platform-root` through the Argo CD API.

## 3. Workload Cluster Add-ons

The `PlatformStack` claim tells Crossplane to configure the workload EKS cluster with:

- workload Argo CD
- Argo Rollouts
- Istio
- Kyverno
- Calico/Tigera operator and baseline NetworkPolicies
- Twistlock/Prisma Cloud Defender when private chart details are provided

## 4. Microservices Repo

Application teams should use a separate microservices repo. That repo can contain Helm charts, Kustomize overlays, or raw manifests.

The workload Argo CD instance should point to that microservices repo. Argo Rollouts and Istio then handle canary or blue/green deployments.

## Required Secrets For Platform Workflow

Add these to the platform repo GitHub Environment or repository secrets:

```text
ARGOCD_SERVER
ARGOCD_AUTH_TOKEN
```

Use an Argo CD project/token with permission to read and sync `platform-root`, not a personal admin token.
