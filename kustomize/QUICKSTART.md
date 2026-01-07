# Glance Kustomize Quick Start

This is a quick reference guide for deploying Glance using Kustomize and ArgoCD.

## Prerequisites

- Kubernetes cluster (v1.19+)
- kubectl installed and configured
- ArgoCD installed (for ArgoCD deployments)

## Deploy with kubectl

### Basic Deployment

```bash
# Deploy to glance namespace
kubectl apply -k kustomize/base/

# Check status
kubectl get pods -n glance
kubectl get svc -n glance

# Access the service (port-forward)
kubectl port-forward -n glance svc/glance 8080:8080

# Open browser to http://localhost:8080
```

### Development Environment

```bash
# Deploy to glance-dev namespace
kubectl apply -k kustomize/overlays/development/

# Check status
kubectl get pods -n glance-dev

# Port forward
kubectl port-forward -n glance-dev svc/dev-glance 8080:8080
```

### Production Environment

**Important:** Before deploying to production, customize the ingress hostname in [kustomize/overlays/production/ingress.yaml](overlays/production/ingress.yaml):

```yaml
spec:
  tls:
    - hosts:
        - your-domain.com  # Change this
      secretName: glance-tls
  rules:
    - host: your-domain.com  # Change this
```

Then deploy:

```bash
# Deploy to glance namespace with production settings
kubectl apply -k kustomize/overlays/production/

# Check status
kubectl get pods -n glance
kubectl get ingress -n glance
kubectl get hpa -n glance
```

## Deploy with ArgoCD

### Prerequisites

- ArgoCD installed in your cluster
- kubectl access to argocd namespace

### Update Repository URL

Before deploying, update the repository URL in the ArgoCD application manifests:

```bash
# Edit argocd/application-kustomize.yaml
# Update spec.source.repoURL to your repository
```

### Deploy Applications

#### Base Deployment

```bash
kubectl apply -f argocd/application-kustomize.yaml

# Check application status
argocd app get glance-kustomize

# Sync manually (if needed)
argocd app sync glance-kustomize

# View in UI
# Navigate to your ArgoCD URL
```

#### Development Environment

```bash
kubectl apply -f argocd/application-kustomize-dev.yaml
argocd app get glance-kustomize-dev
```

#### Production Environment

```bash
kubectl apply -f argocd/application-kustomize-prod.yaml
argocd app get glance-kustomize-prod
```

## Customization

### Change Image Version

```bash
cd kustomize/base
kustomize edit set image glanceapp/glance=glanceapp/glance:v0.8.0

# Or edit kustomization.yaml directly
```

### Customize Configuration

Edit [kustomize/base/configmap.yaml](base/configmap.yaml) to modify the Glance configuration:

```yaml
data:
  glance.yml: |
    pages:
      - name: Home
        columns:
          - size: full
            widgets:
              - type: rss
                feeds:
                  - url: https://your-feed.com/rss
```

### Add Environment Variables

Create a patch in the overlay directory:

```yaml
# overlays/production/env-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: glance
spec:
  template:
    spec:
      containers:
      - name: glance
        env:
        - name: TZ
          value: "America/New_York"
```

Add to kustomization.yaml:
```yaml
patches:
  - path: env-patch.yaml
```

## Uninstall

### kubectl

```bash
# Base
kubectl delete -k kustomize/base/

# Development
kubectl delete -k kustomize/overlays/development/

# Production
kubectl delete -k kustomize/overlays/production/
```

### ArgoCD

```bash
# Base
argocd app delete glance-kustomize

# Or with kubectl
kubectl delete -f argocd/application-kustomize.yaml
```

## Troubleshooting

### Check pod logs

```bash
kubectl logs -n glance deployment/glance
```

### Check events

```bash
kubectl get events -n glance --sort-by='.lastTimestamp'
```

### Verify configuration

```bash
# Preview what will be deployed
kubectl kustomize kustomize/base/

# Diff with cluster
kubectl diff -k kustomize/base/
```

### ArgoCD sync issues

```bash
# View sync status
argocd app get glance-kustomize

# Force sync
argocd app sync glance-kustomize --force

# View diff
argocd app diff glance-kustomize
```

## Next Steps

- Read the full [Kustomize README](README.md) for detailed documentation
- Customize the [ConfigMap](base/configmap.yaml) with your widgets
- Set up [Ingress](overlays/production/ingress.yaml) for production
- Configure persistent storage for data persistence
- Set up monitoring and alerts

## Additional Resources

- [Kustomize Documentation](https://kustomize.io/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Glance Documentation](https://github.com/glanceapp/glance)
