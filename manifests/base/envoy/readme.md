# Envoy Gateway

Envoy Gateway is a cloud-agnostic Gateway controller that leverages the Envoy Proxy to provide advanced traffic management capabilities. It is designed to be a high-performance, extensible, and secure solution for managing ingress traffic in Kubernetes environments.

> [!IMPORTANT]
> Please note that Envoy Gateway is no longer actively maintained on EKS due to its reliance on creating classic load balancers, which are not optimal for AWS environments. 
> For AWS users, it is recommended to use the AWS Load Balancer Controller, which is specifically designed to manage load balancers (ALB/NLB) in AWS environments via ingress sources.

#### Example Kubernetes `Gateway` CRD with Static IP and Port:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: kafka-tcp-gateway
  namespace: envoy-gateway
spec:
  gatewayClassName: envoy-gateway-class
  listeners:
  - name: kafka-listener
    protocol: TCP
    port: 9094
    allowedRoutes:
      namespaces:
        from: All
  infrastructure:
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-ipv4: "10.199.249.232"  # Use static IP for all LB services
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"  # Internal LB for corporate network
      service.beta.kubernetes.io/port_9094_health-probe_protocol: "Tcp"  # TCP health check protocol
      service.beta.kubernetes.io/port_9094_health-probe_port: "9094"  # Health probe on port 9094
      service.beta.kubernetes.io/port_9094_health-probe_interval: "5"  # Health check interval in seconds
      service.beta.kubernetes.io/port_9094_health-probe_num-of-probe: "3"  # Number of unhealthy checks before marking the service as unhealthy
```

### Key Points:
- **Static IP Assignment**: The annotation `service.beta.kubernetes.io/azure-load-balancer-ipv4` ensures that the Azure Load Balancer uses the IP address `10.199.249.232` for all services. This prevents the creation of new IPs with each new service deployment.
- **Single Port Exposure**: By setting `port: 9094`, the service exposes only port `9094`, making network management simpler and more secure within your corporate network.
- **Health Checks**: The health probe annotations are optional but provide an extra layer of monitoring to ensure the service remains healthy.

### Network Security Group (NSG) Configuration:
To manage the exposed ports effectively, ensure that you contact Azure-Aws team to:
1. **Open port `9094`** in the NSG for the internal load balancer IP.
2. **Restrict unnecessary ports**: Since the service forwards only traffic on `9094`, other NodePorts do not need to be managed or exposed.

### Reference

- [All Annotations azure sigs](https://cloud-provider-azure.sigs.k8s.io/topics/loadbalancer/#loadbalancer-annotations)
- [Annotations Microsoft Learn](https://learn.microsoft.com/en-us/azure/aks/load-balancer-standard#customizations-via-kubernetes-annotations)

---
