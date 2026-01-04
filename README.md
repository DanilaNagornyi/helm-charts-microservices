<p align="center">
<img src="https://helm.sh/img/helm.svg" width="200" alt="Helm Charts" />
</p>

![Kubernetes](https://img.shields.io/badge/kubernetes-v1.24+-326ce5?logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-v3.0+-0F1689?logo=helm&logoColor=white)
![Go](https://img.shields.io/badge/Go-00ADD8?logo=go&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![gRPC](https://img.shields.io/badge/gRPC-4285F4?logo=google&logoColor=white)

**Helm Charts for Online Boutique Microservices**

This repository contains **Helm charts and configurations** for deploying Google's [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) microservices demo application. It provides a simplified, production-ready approach to deploying the 11-tier e-commerce application using Helm and Helmfile.

> 💡 **Perfect for learning:** This project demonstrates Helm best practices, microservices deployment patterns, and infrastructure-as-code principles for Kubernetes.

If you're using these charts, please **★Star** this repository to show your interest!

## 📋 Table of Contents

- [What's Inside](#whats-inside)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Deployment Options](#deployment-options)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Learning Resources](#learning-resources)

---

## What's Inside

This repository provides:

- **Reusable Helm Charts**: Generic microservice chart for deploying any of the 11 services
- **Service-specific Values**: Pre-configured values files for each microservice
- **Redis Chart**: Dedicated chart for the Redis cache service
- **Helmfile Integration**: Declarative deployment management for all services
- **Install Scripts**: Simple bash scripts for quick deployment
- **Raw Manifests**: Traditional Kubernetes YAML configurations as reference

### Key Features

- ✅ **DRY Principle**: Single generic chart reused across multiple microservices
- ✅ **Easy Customization**: Service-specific configurations via values files
- ✅ **Multiple Deployment Methods**: Helm, Helmfile, or kubectl
- ✅ **Production Patterns**: Resource limits, health checks, and best practices
- ✅ **Version Control**: Explicit versioning for all container images

---

## Architecture

The application consists of **11 microservices** communicating via gRPC:

| Service | Language | Port | Description |
|---------|----------|------|-------------|
| **frontend** | Go | 8080 | HTTP server serving the web UI (LoadBalancer) |
| **cartservice** | C# | 7070 | Shopping cart management with Redis backend |
| **productcatalogservice** | Go | 3550 | Product catalog and search functionality |
| **currencyservice** | Node.js | 7000 | Currency conversion service |
| **paymentservice** | Node.js | 50051 | Payment processing (mock) |
| **shippingservice** | Go | 50051 | Shipping cost calculation |
| **emailservice** | Python | 5000 | Order confirmation emails (mock) |
| **checkoutservice** | Go | 5050 | Orchestrates cart, payment, shipping, email |
| **recommendationservice** | Python | 8080 | Product recommendations based on cart |
| **adservice** | Java | 9555 | Contextual advertisement service |
| **redis-cart** | Redis | 6379 | Cache for shopping cart data |

**Service Dependencies:**

```
frontend → productcatalog, currency, cart, recommendation, shipping, checkout, ad
checkout → productcatalog, payment, email, currency, shipping, cart
cart → redis-cart
recommendation → productcatalog
```

---

## Prerequisites

Before you begin, ensure you have:

- **Kubernetes Cluster** (v1.24+)
  - Local: [Minikube](https://minikube.sigs.k8s.io/), [Kind](https://kind.sigs.k8s.io/), [Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Cloud: [GKE](https://cloud.google.com/kubernetes-engine), [EKS](https://aws.amazon.com/eks/), [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/), [DigitalOcean Kubernetes](https://www.digitalocean.com/products/kubernetes)

- **Tools Installed:**
  - [kubectl](https://kubernetes.io/docs/tasks/tools/) (configured with cluster access)
  - [Helm 3.x](https://helm.sh/docs/intro/install/)
  - [Helmfile](https://helmfile.readthedocs.io/) (optional, for declarative deployments)
  - `git` (to clone the repository)

- **Cluster Resources:**
  - Minimum: 2 nodes with 2GB RAM each
  - Recommended: 3 nodes with 2-4GB RAM each

---

## Quick Start

### Option 1: Deploy with Helmfile (Recommended)

The simplest way to deploy all services at once:

```bash
# Clone the repository
git clone <repository-url>
cd helm-chart-microservices/

# Deploy all services
helmfile sync

# Check deployment status
kubectl get pods
kubectl get services
```

### Option 2: Deploy with Bash Script

Use the provided install script:

```bash
# Make scripts executable
chmod +x install.sh uninstall.sh

# Deploy all services
./install.sh

# Wait for pods to be ready
kubectl get pods -w
```

Press `Ctrl+C` when all pods show `Running` status.

### Option 3: Deploy with Helm (Manual)

Deploy services individually:

```bash
# Deploy Redis cache first
helm install -f values/redis-values.yaml rediscart charts/redis

# Deploy microservices
helm install -f values/email-service-values.yaml emailservice charts/microservice
helm install -f values/cart-service-values.yaml cartservice charts/microservice
helm install -f values/currency-service-values.yaml currencyservice charts/microservice
helm install -f values/payment-service-values.yaml paymentservice charts/microservice
helm install -f values/recommendation-service-values.yaml recommendationservice charts/microservice
helm install -f values/productcatalog-service-values.yaml productcatalogservice charts/microservice
helm install -f values/shipping-service-values.yaml shippingservice charts/microservice
helm install -f values/ad-service-values.yaml adservice charts/microservice
helm install -f values/checkout-service-values.yaml checkoutservice charts/microservice
helm install -f values/frontend-values.yaml frontendservice charts/microservice
```

### Option 4: Deploy with kubectl (Raw Manifests)

Use the raw Kubernetes manifests:

```bash
kubectl apply -f config.yaml
```

---

## Access the Application

After deployment, get the frontend external IP:

```bash
# Wait for LoadBalancer to provision
kubectl get service frontend -w

# Get the external IP
EXTERNAL_IP=$(kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Online Boutique URL: http://$EXTERNAL_IP"
```

Visit the URL in your browser! 🎉

> **Note:** On cloud providers (GKE, EKS, AKS, DigitalOcean), LoadBalancer provisioning takes 2-5 minutes. On local clusters (Minikube, Kind), you may need to use `kubectl port-forward` or `minikube tunnel`.

### Local Development Access

If LoadBalancer is not available:

```bash
# Port forward the frontend service
kubectl port-forward service/frontend 8080:80

# Access at http://localhost:8080
```

---

## Project Structure

```
helm-chart-microservices/
├── charts/                          # Helm charts
│   ├── microservice/               # Generic microservice chart
│   │   ├── Chart.yaml              # Chart metadata
│   │   ├── values.yaml             # Default values
│   │   └── templates/              # Kubernetes templates
│   │       ├── deployment.yaml     # Deployment template
│   │       └── service.yaml        # Service template
│   └── redis/                      # Redis-specific chart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
├── values/                         # Service-specific configurations
│   ├── email-service-values.yaml
│   ├── cart-service-values.yaml
│   ├── currency-service-values.yaml
│   ├── payment-service-values.yaml
│   ├── recommendation-service-values.yaml
│   ├── productcatalog-service-values.yaml
│   ├── shipping-service-values.yaml
│   ├── ad-service-values.yaml
│   ├── checkout-service-values.yaml
│   ├── frontend-values.yaml
│   └── redis-values.yaml
├── helmfile.yaml                   # Helmfile configuration
├── config.yaml                     # Raw Kubernetes manifests
├── install.sh                      # Quick install script
└── uninstall.sh                    # Cleanup script
```

---

## Configuration

### Generic Microservice Chart

The `charts/microservice` chart is designed to be reusable across all microservices. It accepts the following values:

```yaml
appName: servicename                # Name of the microservice
appImage: image-registry/image      # Container image
appVersion: v0.10.4                 # Image tag/version
appReplicas: 2                      # Number of replicas
containerPort: 8080                 # Container port
containerEnvVars:                   # Environment variables
  - name: ENV_VAR_NAME
    value: "value"
servicePort: 8080                   # Service port
serviceType: ClusterIP              # Service type (ClusterIP/LoadBalancer)
```

### Example: Frontend Service Configuration

`values/frontend-values.yaml`:

```yaml
appName: frontend
appImage: us-central1-docker.pkg.dev/google-samples/microservices-demo/frontend
appVersion: v0.10.4
appReplicas: 2
containerPort: 8080
containerEnvVars:
  - name: PORT
    value: "8080"
  - name: PRODUCT_CATALOG_SERVICE_ADDR
    value: "productcatalogservice:3550"
  - name: CURRENCY_SERVICE_ADDR
    value: "currencyservice:7000"
  # ... more environment variables

servicePort: 80
serviceType: LoadBalancer
```

### Helmfile Configuration

The `helmfile.yaml` provides declarative deployment management:

```yaml
releases:
  - name: rediscart
    chart: charts/redis
    values:
      - values/redis-values.yaml
      
  - name: emailservice
    chart: charts/microservice
    values:
      - values/email-service-values.yaml
      
  # ... more releases
```

**Helmfile Benefits:**
- Declarative deployment of multiple releases
- Dependency management
- Environment-specific configurations
- Automated release ordering

---

## Usage

### Deploy All Services

```bash
# Using Helmfile
helmfile sync

# Using install script
./install.sh

# Using raw manifests
kubectl apply -f config.yaml
```

### Update a Single Service

```bash
# Update with Helm
helm upgrade -f values/frontend-values.yaml frontendservice charts/microservice

# Update specific release with Helmfile
helmfile -l name=frontendservice sync
```

### Scale a Service

```bash
# Scale using kubectl
kubectl scale deployment frontend --replicas=3

# Or update values file and redeploy
# Edit values/frontend-values.yaml: appReplicas: 3
helm upgrade -f values/frontend-values.yaml frontendservice charts/microservice
```

### View Deployed Releases

```bash
# List all Helm releases
helm list

# Check Helmfile status
helmfile status
```

### Monitor Services

```bash
# Watch pod status
kubectl get pods -w

# Check all services
kubectl get services

# View logs for a service
kubectl logs -l app=frontend --tail=50 -f

# Check resource usage
kubectl top nodes
kubectl top pods
```

### Cleanup

```bash
# Using Helmfile
helmfile destroy

# Using uninstall script
./uninstall.sh

# Using Helm manually
helm uninstall rediscart emailservice cartservice currencyservice paymentservice \
  recommendationservice productcatalogservice shippingservice adservice \
  checkoutservice frontendservice

# Using kubectl
kubectl delete -f config.yaml
```

---

## Deployment Options

### Helmfile (Recommended for Production)

**Advantages:**
- Declarative configuration
- Manages multiple releases as a single unit
- Environment management (dev, staging, prod)
- Easier rollbacks
- Better version control integration

```bash
helmfile sync                    # Deploy/update all releases
helmfile diff                    # Show changes before applying
helmfile destroy                 # Remove all releases
helmfile -l name=frontend sync   # Deploy specific service
```

### Helm (Manual Control)

**Advantages:**
- Fine-grained control over each release
- Standard Helm tooling
- Direct interaction with releases

```bash
helm install -f values/frontend-values.yaml frontendservice charts/microservice
helm upgrade -f values/frontend-values.yaml frontendservice charts/microservice
helm uninstall frontendservice
```

### kubectl (Raw Manifests)

**Advantages:**
- No additional tools required
- Direct Kubernetes resource management
- Useful for troubleshooting

```bash
kubectl apply -f config.yaml
kubectl delete -f config.yaml
```

---

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl get pods

# Describe problematic pod
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

**Common issues:**
- Insufficient cluster resources (scale up nodes)
- Image pull errors (check image registry and version)
- Configuration errors (verify values files)

### Service Communication Errors

```bash
# Test connectivity between services
kubectl run test --rm -it --image=busybox -- wget -O- frontend:80

# Check service endpoints
kubectl get endpoints

# View service logs for connection errors
kubectl logs -l app=cartservice --tail=100
```

### LoadBalancer Stuck in Pending

```bash
kubectl get service frontend
```

**Solutions:**
- **Cloud providers**: Wait 2-5 minutes for provisioning
- **Local clusters**: Use `minikube tunnel` or `kubectl port-forward`
- **Check quotas**: Verify LoadBalancer quota in cloud account

### Redis Connection Errors

```bash
# Check Redis pod
kubectl get pods -l app=redis-cart

# View Redis logs
kubectl logs -l app=redis-cart

# Restart Redis
kubectl delete pod -l app=redis-cart
```

### Resource Exhaustion

```bash
# Check resource usage
kubectl top nodes
kubectl top pods

# Scale down non-essential services
kubectl scale deployment adservice --replicas=0
```

**Solutions:**
- Increase node size or add more nodes
- Adjust resource limits in values files
- Scale down replicas during development

### Debugging Commands

```bash
# Get all resources
kubectl get all

# Execute commands in a pod
kubectl exec -it <pod-name> -- /bin/sh

# Port forward for local testing
kubectl port-forward service/frontend 8080:80

# Check cluster info
kubectl cluster-info
kubectl get nodes -o wide

# View Helm release history
helm history frontendservice

# Rollback a release
helm rollback frontendservice 1
```

---

## Learning Resources

### Understanding This Repository

1. **Helm Charts Basics**
   - Study `charts/microservice/templates/` to understand templating
   - Compare `values.yaml` with service-specific values files
   - Learn how values override default configurations

2. **Helmfile Concepts**
   - Review `helmfile.yaml` structure
   - Understand release definitions and dependencies
   - Practice with `helmfile diff` before applying changes

3. **Microservices Communication**
   - Analyze environment variables in values files
   - Understand service discovery via DNS (e.g., `cartservice:7070`)
   - Trace request flow: frontend → checkout → payment/shipping/email

### Helm Best Practices

- ✅ **Use values files** for configuration (not hardcoded in templates)
- ✅ **Create reusable charts** (like our generic microservice chart)
- ✅ **Version your charts** (`Chart.yaml` version field)
- ✅ **Define resource limits** (CPU, memory in values)
- ✅ **Include health checks** (liveness and readiness probes)
- ✅ **Use namespaces** for environment isolation

### Official Documentation

- [Helm Documentation](https://helm.sh/docs/)
- [Helmfile Documentation](https://helmfile.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Online Boutique Project](https://github.com/GoogleCloudPlatform/microservices-demo)

### Advanced Topics

- **Helm Hooks**: Pre/post-install actions
- **Chart Dependencies**: Manage chart relationships
- **Kustomize Integration**: Combine with Helm for advanced use cases
- **Secret Management**: Use Sealed Secrets or External Secrets Operator
- **GitOps**: Integrate with ArgoCD or Flux

---

## 🎯 Next Steps

### For Beginners

1. Deploy the application using Helmfile
2. Explore the Helm charts in `charts/` directory
3. Modify a values file and redeploy
4. Scale a service up/down
5. Practice monitoring and troubleshooting

### Intermediate

1. Create a new microservice using the generic chart
2. Customize Helm templates for specific needs
3. Set up different environments (dev, staging, prod) with Helmfile
4. Implement resource quotas and limits
5. Add ConfigMaps and Secrets

### Advanced

1. Build CI/CD pipelines with Helm/Helmfile
2. Implement Helm chart testing
3. Create custom Helm plugins
4. Integrate with service mesh (Istio)
5. Set up monitoring with Prometheus/Grafana Helm charts

---

## 📝 Related Projects

- [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) - The original microservices demo
- [Helm Charts Repository](https://github.com/helm/charts) - Official Helm charts
- [Bitnami Charts](https://github.com/bitnami/charts) - Production-ready charts

---

## 💡 Key Takeaways

This repository demonstrates:

- **Chart Reusability**: One generic chart for multiple microservices
- **Configuration Management**: Separate values files for each service
- **Deployment Automation**: Multiple deployment methods (Helm, Helmfile, kubectl)
- **Best Practices**: Resource limits, health checks, proper service discovery
- **Scalability**: Easy horizontal scaling via replica configuration
- **Version Control**: Explicit container image versions
- **Infrastructure as Code**: Declarative configuration management

---

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project follows the same license as the [Online Boutique demo application](https://github.com/GoogleCloudPlatform/microservices-demo).

---

**Happy Helming! ⎈**
