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

Create the HelmRelease and export it.

```
flux create hr crossplane \
    --target-namespace=crossplane-system \
    --create-target-namespace=true \
    --source=HelmRepository/crossplane \
    --chart=crossplane --export
```

The result was added to the repo with some changes.

## Create the Flux Config

You can use the portal to add a GitOps configuration:
- add the repo with username and PAT as password (if private)
- create a kustomization that uses the infra folder
- add extra kustomizations if needed

When you do this via the portal, Flux will be installed on the cluster automatically.

From the command line:

```
RG=rg-aks
CLUSTER=clu-crossplane
USER=gbaeke
PAT=<PAT>

az k8s-configuration flux create -g $RG -c $CLUSTER \
  -n cluster-config --namespace config-infra -t managedClusters \
  --scope cluster -u https://github.com/gbaeke/crossplane \
  --branch main  \
  --https-user $USER --https-key $PAT \
  --kustomization name=infra path=./infra prune=true \
  --kustomization name=secrets path=./secrets prune=true dependsOn=["infra"] \
  --kustomization name=apps path=./crossplane-apps prune=true dependsOn=["secrets"]
```

Above, the k8s configuration has 3 kustomizations:
- infra: installs Crossplane and Azure Key Vault to Kubernetes; it also installs Crossplane providers
- secrets: creates a secret that contains credentials to configure the Crossplane providers; the secret comes from a Key Vault
- apps: a Kustomization where I can drop custom resourses to create Azure resources to play with like resource groups, storage accounts, AKS clusters, Kubernetes resources

Note: you can remove --https-user and --https-key if the repo is public


## Setup Azure Provider

These are the manual steps. The Azure Key Vault to Kubernetes controller and secret sync from Key Vault via the secrets kustomization should take care of this.

```
az ad sp create-for-rbac --sdk-auth --role Owner > "creds.json"
kubectl create secret generic azure-creds -n crossplane-system --from-file=creds=./creds.json
```



