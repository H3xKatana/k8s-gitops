# K8s GitOps Setup for k8s-micro.0xkatana.site

This repository contains the GitOps configuration for deploying the demo microservice with HTTPS using HTTP challenge validation.

## Prerequisites

Before applying the configuration, ensure you have:

1. **k3s cluster running**
2. **kubectl configured** to connect to your cluster
3. **Kustomize** installed (`kustomize` or `kubectl kustomize`)
4. **Cert-Manager** installed for HTTPS certificates
5. **Traefik Ingress Controller** (comes with k3s by default)

## Installing Cert-Manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml
```

Wait for cert-manager to be ready:

```bash
kubectl wait --for=condition=ready pod -l app.kubernetes.io/instance=cert-manager -n cert-manager --timeout=90s
```

Then create the ClusterIssuer for Let's Encrypt:

```bash
kubectl apply -f https://gist.githubusercontent.com/traefik/ed786349843453335d77814e43e3f3c2/raw/0a04e4e9fb0b15aa0e29a969885e4c2c3f5e61c9/cluster-issuer.yml
```

Or create a custom ClusterIssuer by applying the following:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com  # Replace with your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik
```

## DNS Configuration

1. Configure your DNS provider to create an A record for `k8s-micro.0xkatana.site` pointing to your cluster's external IP address.

2. This setup uses HTTP challenge validation (not DNS), so no API tokens or DNS automation is required.

## Applying the Configuration

To apply the configuration to your k3s cluster:

```bash
kubectl kustomize base/ | kubectl apply -f -
```

Or using kubectl:

```bash
kubectl apply -k base/
```

## Testing the Setup

1. Check all resources are running:
   ```bash
   kubectl get pods,svc,ingress,certificate,certificaterequest -n default
   ```

2. Check if the certificate is issued:
   ```bash
   kubectl describe certificate k8s-micro-certificate -n default
   ```

3. Monitor the ingress:
   ```bash
   kubectl describe ingress demo-microservice-api-ingress -n default
   ```

## Expected Result

Once everything is set up correctly:
- DNS record for `k8s-micro.0xkatana.site` points to your k3s cluster
- HTTPS certificate is issued and valid for `k8s-micro.0xkatana.site`
- The service is accessible via `https://k8s-micro.0xkatana.site`

## Troubleshooting

### If certificates are not issued:
1. Check cert-manager logs:
   ```bash
   kubectl logs -l app.kubernetes.io/name=cert-manager -n cert-manager
   ```

2. Check certificate status:
   ```bash
   kubectl describe certificate k8s-micro-certificate -n default
   ```

### If service is not accessible:
1. Check if ingress has an external IP:
   ```bash
   kubectl get ingress -n default
   ```

2. Verify the service is running:
   ```bash
   kubectl get svc demo-microservice-api-svc -n default
   ```

3. Verify your DNS A record is properly pointing to your cluster's IP address