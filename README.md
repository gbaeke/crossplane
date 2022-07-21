# Crossplane

Crossplane GitOps Repo

## Configuring Crossplane with Flux

Idea is to create a FluxConfig in Azure that creates:
- source
- one or more kustomizations; one of them points to the infra folder to create Helm repos, install charts, etc...

Create the Helm repository with:

```
flux create source helm crossplane --url https://charts.crossplane.io/stable --export
```

Create the HelmRelease with:

```
flux create hr crossplane \
    --target-namespace=crossplane-system \
    --create-target-namespace=true \
    --source=HelmRepository/crossplane \
    --chart=crossplane
```

Create the Flux Config

You can use the portal to add a GitOps configuration:
- add the repo with username and PAT as password
- create a kustomization that uses the infra folder



