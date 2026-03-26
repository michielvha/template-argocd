## Declarative Approach to Adding Clusters to ArgoCD

To leverage Azure Workload Identity for automatically adding AKS clusters to ArgoCD, several preparatory steps are required. Below is a guide using Kustomize to overlay the necessary configurations within this repository structure.

> [!NOTE]
> A full guide with reference examples can be found on [my personal website](https://vhco.pro/blog/platform/aks/declarative-cluster-onboarding-argocd.html)

### Prerequisites

1. **Azure User-Managed Identities (UMIs):** Set up UMIs in Azure with federated credentials and appropriate rights to access AKS cluster credentials. The UMI must be able to read secrets from the Key Vault where cluster CA cert and server URL are stored.

2. **Azure Key Vault Secrets:** Store the following secrets in your Key Vault (e.g. `argocd-prd-akv`):
   - `<cluster-name>-ca-cert`: Base64-encoded CA certificate from the target cluster
   - `<cluster-name>-server-url`: Private endpoint URL of the target cluster's API server

### 1. Enable Azure Workload Identity for ArgoCD

The `azure-patches.yaml` file in the prd overlay contains the patches for Azure Workload Identity. Uncomment it in `manifests/base/argocd/overlays/prd/kustomization.yaml`:

```yaml
patches:
- path: patches.yaml
- path: azure-patches.yaml   # Uncomment this line
```

Update the placeholders in `azure-patches.yaml`:
- `<ARGOCD_UMI_CLIENT_ID>`: Client ID of your Azure Workload Identity
- `<YOUR_TENANT_ID>`: Azure tenant ID

This patches the `argocd-server` and `argocd-application-controller` ServiceAccounts with workload identity annotations, and sets `azure.workload.identity/use: "true"` on their deployments.

### 2. Configure ClusterSecretStore

The External Secrets Operator uses a `ClusterSecretStore` named `argocd-prd-store` (see `manifests/base/external-secrets-operator/overlays/azure/cluster-secret-store.yaml`). Ensure:
- The Key Vault URL is correct
- The `argocd` namespace is included in the store's `conditions.namespaces`

### 3. Add a New Cluster

To add a new AKS cluster to ArgoCD:

**a) Create a cluster overlay folder**

Create a new folder under `clusters/` named after your cluster (e.g. `aks-int-we`):

```
manifests/base/argocd/overlays/prd/clusters/
├── kustomization.yaml
├── readme.md
└── <your-cluster-name>/          # e.g. aks-int-we
    └── kustomization.yaml
```

**b) Create the cluster kustomization**

Create `kustomization.yaml` in your cluster folder:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../../../base/aks-clusters

patches:
- target:
    name: cluster-external-secret
    kind: ExternalSecret
  patch: |-
    - op: replace
      path: /metadata/name
      value: <cluster-name>-cluster-external-secret
    - op: replace
      path: /spec/target/name
      value: <cluster-name>-cluster-secret
    - op: replace
      path: /spec/target/template/data/name
      value: <cluster-name>
    - op: replace
      path: /spec/data/0/remoteRef/key
      value: secret/<cluster-name>-ca-cert
    - op: replace
      path: /spec/data/1/remoteRef/key
      value: secret/<cluster-name>-server-url
```

Replace `<cluster-name>` with your cluster identifier (e.g. `aks-int-we`). This value will appear as the cluster name in the ArgoCD UI.

**c) Register the cluster in the clusters kustomization**

Add your cluster folder to `clusters/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- <your-cluster-name>/
```

**d) Include clusters in the prd overlay**

Ensure `clusters` is listed as a resource in `manifests/base/argocd/overlays/prd/kustomization.yaml`:

```yaml
resources:
- ../../base
- https://raw.githubusercontent.com/argoproj/argo-cd/v3.1.6/manifests/ha/install.yaml
- projects
- clusters
```

### How It Works

The base ExternalSecret at `manifests/base/argocd/base/aks-clusters/external-secret.yaml` defines the template. It:
- References the `argocd-prd-store` SecretStore
- Uses `argocd-k8s-auth` with Azure Workload Identity for cluster authentication
- Fetches `caCert` and `serverUrl` from Key Vault

Each cluster overlay patches the base with cluster-specific values. The External Secrets Operator syncs the resulting secrets, which ArgoCD automatically discovers via the `argocd.argoproj.io/secret-type: cluster` label.

### Manual Secret (Alternative to External Secrets)

If you prefer not to use External Secrets, you can create a Kubernetes Secret manually:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <cluster-name>-cluster-secret
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: <cluster-name>
  server: <private-endpoint-of-kubernetes-api-server>
  config: |
    {
      "execProviderConfig": {
        "command": "argocd-k8s-auth",
        "env": {
          "AZURE_CLIENT_ID": "<UMI_CLIENT_ID>",
          "AZURE_TENANT_ID": "<TENANT_ID>",
          "AZURE_FEDERATED_TOKEN_FILE": "/var/run/secrets/azure/tokens/azure-identity-token",
          "AZURE_AUTHORITY_HOST": "https://login.microsoftonline.com/",
          "AAD_ENVIRONMENT_NAME": "AzurePublicCloud",
          "AAD_LOGIN_METHOD": "workloadidentity"
        },
        "args": ["azure"],
        "apiVersion": "client.authentication.k8s.io/v1beta1"
      },
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<base64-ca-data-from-kubeconfig>"
      }
    }
```
