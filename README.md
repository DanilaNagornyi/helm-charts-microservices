# Deploy Online Boutique Microservices to Kubernetes

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-0080FF?style=for-the-badge&logo=digitalocean&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

</div>

This capstone project contains the Kubernetes configuration used to deploy the Online Boutique microservices application. The original exercise targeted Linode LKE, but I could not complete the Linode account verification, so I adapted and deployed the project to a managed DigitalOcean Kubernetes cluster.

The application source and container images come from the [Google Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) project. This repository focuses on the Kubernetes manifests, service configuration, image updates, and Helm-based deployment created during the bootcamp project.

## Architecture

The application includes the following components:

- frontend;
- cart service with Redis;
- product catalog and recommendation services;
- currency, payment and shipping services;
- checkout and email services;
- advertisement service.

Each application component has a Kubernetes Deployment and Service. Internal services use `ClusterIP` and Kubernetes DNS for communication. The frontend uses a `LoadBalancer` Service to provide external access through the cloud provider.

## Deployment Configuration

Two deployment approaches are included:

1. **Raw Kubernetes manifests** in `config.yaml` for all services.
2. **Helm charts and Helmfile** with a reusable microservice chart and separate values for every service.

The manifests include:

- labels and selectors connecting Services to Pods;
- environment variables for service discovery;
- multiple replicas for the application services;
- liveness and readiness probes;
- resource requests and limits for selected workloads;
- Redis storage using an `emptyDir` volume;
- an external LoadBalancer for the frontend.

## Project Structure

```text
.
├── config.yaml                    # Raw Kubernetes manifests for the complete application
├── charts/
│   ├── microservice/              # Reusable chart for application services
│   └── redis/                     # Redis-specific chart
├── values/                        # Configuration for each microservice
├── helmfile.yaml                  # All Helm releases in one declarative file
├── install.sh                     # Install all releases with Helm
└── uninstall.sh                   # Remove all Helm releases
```

No kubeconfig or cloud credentials are stored in this repository.

## Prerequisites

- access to a Kubernetes cluster;
- `kubectl` configured for the cluster;
- Helm 3;
- Helmfile when using `helmfile.yaml`.

The project can also be tested locally with Minikube. For local access, use `minikube tunnel` for the LoadBalancer or port-forward the frontend Service.

## Deploy with Raw Manifests

```bash
kubectl apply -f config.yaml
kubectl get pods
kubectl get services
```

Get the public endpoint after the cloud LoadBalancer is ready:

```bash
kubectl get service frontend
```

Remove the deployment:

```bash
kubectl delete -f config.yaml
```

## Deploy with Helmfile

```bash
helmfile sync
helmfile status
```

Remove all releases:

```bash
helmfile destroy
```

## What I Learned

Kubernetes was one of my favorite and most challenging modules. Deploying a complete microservices application helped me understand how Deployments, Services, labels, selectors, environment variables, health probes, resource settings, and internal DNS work together. I also gained practical troubleshooting experience when the original cloud provider and older image links could not be used. I located the current upstream image paths, updated the deployment configuration, and successfully adapted the project to DigitalOcean Kubernetes. Minikube was also very useful for testing and learning Kubernetes locally.
