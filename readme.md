# GitOps Platform Template

This repository is used for bootstrapping a **complete GitOps platform foundation** including Continous Delivery, Secret Management, and load balancing controllers.  

It serves as a base template for managing a GitOps platform configurations across different environments (development and production) using a kustomize-based approach.

All the environment specific stuff will have to be added to an overlay based on your needs.

We include the Platform Engineering and GitOps Paradigms to ensure a robust, scalable, and secure foundation for managing Kubernetes clusters and applications.

> [!IMPORTANT]
> By default the `main` branch of this repository is continually reconciled by the Argo CD Controller. Any changes made to this
> branch will be automatically applied to the live environment. Please ensure all modifications are intentional and reviewed before committing.

## General Architecture

What makes a great platform is pretty subjective and really comes down to your own requirements.
It depends on the cloud provider you are using, or maybe you are on prem, who knows.

The purpose of this repository is to provide a basic template that can be used as a starting point for any platform no matter which infrastructure you are on. 
While the main focus of this template is cloud we will include an on-prem component as well (for all my home labbers and true kubernetes warriors).

Given our requirements we have decided to follow the gitops separation of concerns, this repository will only contain the platform related application components and not the application workload related components.
It will also not include any infrastructure related components (CSI Drivers, Cloud-Specific controllers,...), as these should be managed outside argocd by an IaC tool (enabled via add-ons).

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

The essential components of this GitOps platform are the minimum required to provide a robust and secure foundation for managing Kubernetes clusters and applications.

Ofcourse this is just a template, so you can remove/add any components you (don't) need. However, I would recommend to keep these components as a pure base.

- **Secret Management Tool:** for secure handling of sensitive information.
  - **External Secrets Operator:** (ESO): Integrates external secret management systems with Kubernetes, providing secure secret synchronization from external sources.
- **Continuous Delivery Tool:** Automates the deployment of applications and infrastructure changes.
  - **Argo CD:** for GitOps-based continuous delivery, configured for self-management and application deployment.
- **Essential Add-ons:**
  - **Reloader:** A Kubernetes controller that watches for changes in ConfigMaps and Secrets and triggers pod restarts to apply the new

## Optional Platform Components

The optional components can be included based on specific requirements and use cases. 
These are often the cloud specific components, you are still best of using in most case and thus cannot escape.
These components enhance the platform's capabilities but are not strictly necessary for what we might consider basic operation.

- **Load Balancer Management:** 
  - **Envoy Proxy:** as a high-performance proxy server for load balancing and service mesh capabilities, fully cloud-agnostic, however unfortunately no longer actively maintained on EKS since it creates a classic load balancer, this is thus not suitable for AWS.
  - **AWS Load Balancer Controller:** for managing load balancers (ALB/NLB) in AWS environments via ingress sources, currently only option for proper aws integration.
  - **Metal-lb:** as a simple, lightweight load balancer for bare-metal Kubernetes clusters.

> [!NOTE]
> You might notice that we do not include many tools, this is fully intentional.
> Any application that is not strictly required for the platform's core functionality (e.g. additional policies, auto-scaling, monitoring,...) has been left out.
> While not strictly required, they are often applications that must run on every platform cluster and should instead be added to a ``post-deploy`` application,
> which is managed by **Argo CD** through the ``k8s-resources``` workload manifests repository.

## Repository Structure

Our template Platform configuration follows a kustomize-based approach with the following structure:

```
manifests/
  base/                             # Base platform components (see individual app readme files)
    argocd/                         # ArgoCD base setup (see manifests/base/argocd/readme.md)
    envoy/                          # Envoy base setup (see manifests/base/envoy/readme.md)
    aws-load-balancer-controller/   # AWS LB Controller base setup
    external-secrets-operator/      # ESO base setup
    reloader/                       # Reloader base setup
  overlays/
    dev/                            # Development environment overlay
    prd/                            # Production environment overlay
```

> Details for each base app (argocd, envoy, etc.) are documented in their respective readme files under manifests/base.

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
- **Turn into our main template that has functionalities for all cloud platforms we've used. We should allow to easily enable and disable these features via kustomize.**
- **Add references to the infrastructure repos for azure, eks and use ansible with proxmox for on-prem.**
