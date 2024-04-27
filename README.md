# SSL Hijack demo
Illustrate the process of creating the illusion of accessing a credible website.
The user interface seamlessly mimics that of a legitimate website, effectively concealing the fact
that it's interacting with a locally hosted backend system.
The illusion is achieved by generating a fake self-signed certificate for the domain of the target
HTTPS site and modifying the user's system to accept and trust the falsified certificate.

## Setup
### Pre-requisites:
- Kubernetes cluster
- Helm
- Publicly accessible node if accessed from outside (e.g. via tailscale)

### Install ingress controller and MetalLB
1. Inorder for ingress to work, an ingress controller must be installed.
```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

2. On an on-premise cluster lacking a network load balancer, MetalLB needs to be deployed and the
IP address pool configured.
  2.1 Install metallb
  ```
  helm upgrade --install metallb metallb \
    --repo https://metallb.github.io/metallb \
    --namespace metallb --create-namespace
  ```
  2.2 Once metallb-controller pod is ready, configure metallb IP address pool
  ```
  kubectl apply -f metallb-crd/ipaddresspool.yaml
  ```
  2.3 Verify the external IP of the ingress controller LoadBalancer service
  ```
  kubectl -n ingress-nginx get svc
  ```
