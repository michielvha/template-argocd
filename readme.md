# GitOps Platform Template

**TODO: Turn into our main template that has functionalities for all cloud platforms we've used. We should allow to easily enable and disable these features via kustomize.**

This repository is used for bootstrapping a complete GitOps platform foundation including Argo CD, External Secrets Operator (ESO), and AWS Load Balancer Controller.  

This repository serves as a base template for managing a GitOps platform configurations across different environments (development and production) using a kustomize-based approach.

All the environment specific stuff will have to be added to an overlay based on your needs.

We include the Platform Engineering and GitOps Paradigms to ensure a robust, scalable, and secure foundation for managing Kubernetes clusters and applications.

> [!IMPORTANT]
> By default the `main` branch of this repository is continually reconciled by the Argo CD Controller. Any changes made to this
> branch will be automatically applied to the live environment. Please ensure all modifications are
> intentional and reviewed before committing.

## General Architecture

What makes a great platform is mostly depending on your use case and requirements. It will depend on which type of cloud provider you are using, or maybe you are on prem, who knows.
The purpose of this repository is to provide a basic template that can be used as a starting point for your own platform no matter which infrastructure you are on.
Given our requirements we have decided to follow the gitops separation of concerns, this repository will only contain the platform related application components and not the application workload related components.
It will also not include any infrastructure related components, as these will be managed outside argocd by an IaC tool.

We allow developers to create their own infrastructure related components via crossplane, but only if these are app related and not infra related.

### GitOps Definition

We consider GitOps to be:

- Git as the single source of truth for declarative infrastructure and applications.
- Folder per environment (e.g., `dev`, `prd`) containing kustomize overlays for environment-specific configurations. NO Branches per environment.
  - Automated deployment of changes (through pull requests) on main branch using "trunk-based" development.
- Clear separation of concerns between platform setup, argocd applications and K8S application configuration.

To ensure separation of concerns, three repositories are used to manage the complete platform setup:

* Platform foundation and component
  installation: <TO_ADD>
* Argo CD application
  definitions: <TO_ADD>
* Kubernetes manifests or Helm charts used by Argo CD
  applications: <TO_ADD>

## Essential Platform Components

The essential components of this GitOps platform are required to provide a robust and secure foundation for managing Kubernetes clusters and applications.

Ofcourse this is just a template, so you can remove/add any components you (don't) need. However, I would recommend to keep these components as a pure base that is fully cloud-agnostic.

- **Secret Management Tool:** for secure handling of sensitive information.
  - **External Secrets Operator** (ESO): Integrates external secret management systems with Kubernetes, providing secure secret synchronization from external sources.
- **Continuous Delivery Tool:** Automates the deployment of applications and infrastructure changes.
  - **Argo CD** for GitOps-based continuous delivery, configured for self-management and application deployment.
- **Essential Add-ons:**
  - **Reloader**: A Kubernetes controller that watches for changes in ConfigMaps and Secrets and triggers pod restarts to apply the new

## Optional Platform Components

The optional components can be included based on specific requirements and use cases. 
These are the cloud specific components you are still best of using in most case and thus cannot escape.
These components enhance the platform's capabilities but are not strictly necessary for what we might consider basic operation.

- Load Balancer Management: 
  - **AWS Load Balancer Controller** for managing load balancers (ALB/NLB) in AWS environments via ingress sources.
  - **Azure Application Gateway Ingress Controller** for managing Azure Application Gateway in Azure environments via ingress sources.

    
## Repository Structure

Our template Platform configuration follows a kustomize-based approach with the following structure:

**TODO: update below to latest structure**

```
manifests/
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

**TODO: explain only the main platform overlay folders here, rest of the info should be in a readme in the several app folders under manifests/base.**

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