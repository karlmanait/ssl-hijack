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

    2.1. Install metallb
    ```
    helm upgrade --install metallb metallb \
      --repo https://metallb.github.io/metallb \
      --namespace metallb --create-namespace
    ```
    2.2. Once metallb-controller pod is ready, configure metallb IP address pool
    ```
    kubectl apply -f metallb-crd/ipaddresspool.yaml
    ```
    2.3. Verify the external IP of the ingress controller LoadBalancer service
    ```
    kubectl -n ingress-nginx get svc
    ```

### Deploy spoofsite service locally
```
kubectl apply -f spoofsite/service.yaml
kubectl apply -f spoofsite/deployment.yaml
kubectl apply -f spoofsite/ingress.yaml
```

### Override DNS resolution
Make sure to update the hosts file with the external IP assigned to the ingress controller service.
* Linux - `/etc/hosts`
* Windows - `C:\Windows\System32\drivers\etc\hosts`

### Generate SSL certificate and create secret
1. Create SSL certificate
```
openssl req -x509 -newkey rsa:4096 -sha256 -nodes \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com" -days 365
```
2. Create k8s secret based on SSL certificate
```
kubectl create secret tls demo-tls --cert=tls.crt --key=tls.key
```

### Accept self-signed certificate in the browser
When accessing the site for the first time, you may encounter a warning about the risk associated
with the self-signed certificate. To proceed, simply accept the risk and continue.
This action will add the certificate as an exception in your browser settings.

If the browser redirects you to the original site, clear the DNS cache records in both your browser
and the machine you're using or use "Private/Incognito Mode".
