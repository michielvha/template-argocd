# Platform installation and configuration

This repository is used for bootstrapping a complete GitOps platform foundation including Argo CD, External Secrets
Operator (ESO), and AWS Load Balancer Controller.  
It enables self-management of Argo CD using Argo CD itself, along with the essential infrastructure components required
for a production-ready setup.

> [!IMPORTANT]
> The `main` branch of this repository is continually reconciled by the Argo CD Controller. Any changes made to this
> branch will be automatically and immediately applied to the live environment. Please ensure all modifications are
> intentional and reviewed before committing.

## Platform Components

This platform setup includes three essential components that work together:

### Argo CD

The GitOps continuous delivery tool for Kubernetes, configured for self-management and application deployment.

### External Secrets Operator (ESO)

Integrates external secret management systems with Kubernetes, providing secure secret synchronization from external
sources.

### AWS Load Balancer Controller

Manages AWS Application Load Balancers (ALB) and Network Load Balancers (NLB) for Kubernetes ingress resources.

### Reloader

A Kubernetes controller that watches for changes in ConfigMaps and Secrets and triggers pod restarts to apply the new

## Repository Structure

To ensure separation of concerns, three repositories are used to manage the complete platform setup:

* Platform foundation and component
  installation: <TO_ADD>
* Argo CD application
  definitions: <TO_ADD>
* Kubernetes manifests or Helm charts used by Argo CD
  applications: <TO_ADD>

## Usage

Apply the appropriate overlay to the target Kubernetes cluster (development or production).  
Components will be deployed into their respective namespaces:

- Argo CD: `argocd` namespace
- External Secrets Operator: `external-secrets` namespace
- AWS Load Balancer Controller: `kube-system` namespace

```bash
# Development
kubectl apply -k manifests/platform/overlays/dev

# Production
kubectl apply -k manifests/platform/overlays/prd
```

This setup ensures that all platform components are installed and configured to work together, with Argo CD managing its
own resources and the broader application ecosystem within the cluster.


## TODO:

- Add external DNS operator
- Add Cluster Auto Scaler
