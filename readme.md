# ArgoCD

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes.

Changes both in the cluster and in the Git repository are detected and applied to the cluster.
This way we can be sure that the cluster is always in the desired state.

This repository serves as a base template for managing ArgoCD configurations across different environments (development and production) using a kustomize-based approach.

All the platform specific stuff will have to be added to an overlay based on your needs.

## Repository Structure

Our template ArgoCD configuration follows a kustomize-based approach with the following structure:

```
manifests/
  argocd/
    base/                  # Base components for all environments
        aks-clusters/      # Base AKS cluster configurations
        repos/             # Base repository configurations
    overlays/
      dev/                 # Development environment specific configs
        projects/          # ArgoCD projects for dev
        repos/             # Repository configurations for dev
      prd/                 # Production environment specific configs
        clusters/          # Managed clusters in production
        projects/          # ArgoCD projects for production
        repos/             # Repository configurations for production
    version/
      dev/                 # ArgoCD version definition for dev
      prd/                 # ArgoCD version definition for prod
```

## Configuration Details

### Base Configuration
The base configuration of ArgoCD is done in the `base` folder.
This folder contains all the general basic deployment configuration of ArgoCD.

### Base Clusters
The base clusters configuration is done in the `base/clusters` folder.
This folder contains all the general basic cluster configuration of ArgoCD.

### Base Repos
The base repo configuration is done in the `base/repos` folder.
This folder contains all the general basic repo configuration of ArgoCD.

### Overlays
The overlays folder contains all the overlays for the different environments.
Currently, we have:
- `dev` : Development environment
- `prd` : Production environment

You can add more overlays for other environments as needed.

### Version Management
The version folder contains the `kustomization.yaml` files that pull in the manifests from GitHub. When upgrading ArgoCD, update the URL to the desired version.
- `dev` folder: Contains configuration for development environment versioning
- `prd` folder: Contains configuration for production environment versioning

## Additional Components

### Declarative Cluster Configuration

Declarative Cluster configurations ensures easy deployment using azure workload identities. If cluster has to be redeployed, it will be automatically be re-added to ArgoCD.

To enable this the `azure-workload-identity.yaml` file should be patched onto the configuration. A template has been provided but it can be customized as needed.

Refer to this Guide to figure out how to do this

### Projects
All projects & RBAC can be configured in the projects directory.

### Declarative Repositories Configuration
Declarative Repositories Configuration ensures easy deployment using external secrets operator.

### Configmaps
The configmaps patches can be found in configmaps.yaml file.
Refer to the [ArgoCD documentation](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/) for more information.

### Applications
Each team should have their own application directories:
- `argocd-apps`: Contains all applications managed by ArgoCD (deployed with an app of apps pattern)
- `k8s-resources`: Contains all resources deployed by the ArgoCD application

## Installation and Management

### Installation

Initial installation of ArgoCD is done with the following command:

```bash
kubectl apply -k manifests/argocd
```
After this command is executed, ArgoCD will be installed in the `argocd` namespace.

### Upgrade
Before upgrading ArgoCD, please refer to the [tested version](https://argo-cd.readthedocs.io/en/stable/operator-manual/installation/#supported-versions).

To upgrade, update the version reference in the appropriate `version/{env}/kustomization.yaml` file and apply the changes.

### Recovery
If the ArgoCD installation is corrupted or the cluster is redeployed, it can be recovered with the following command:

```bash
kubectl apply -k manifests/argocd
```
When the main cluster is redeployed, we only need to redeploy our argocd application and all other applications will be deployed automatically.

## References

For more information, refer to the [ArgoCD documentation](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/).
