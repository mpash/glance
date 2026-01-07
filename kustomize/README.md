# Kustomize Deployment for Glance

This directory contains Kustomize configurations for deploying Glance to Kubernetes with ArgoCD.

## Structure

```
kustomize/
├── base/                      # Base Kubernetes manifests
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── serviceaccount.yaml
│   ├── configmap.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/                  # Environment-specific configurations
    ├── development/           # Development environment
    │   ├── kustomization.yaml
    │   └── deployment-patch.yaml
    └── production/            # Production environment
        ├── kustomization.yaml
        ├── deployment-patch.yaml
        ├── ingress.yaml
        ├── hpa.yaml
        └── pvc.yaml
```

## Quick Start

### Deploy with kubectl and Kustomize

#### Base Deployment
```bash
# Deploy base configuration
kubectl apply -k kustomize/base/

# Verify deployment
kubectl get pods -n glance
```

#### Development Environment
```bash
# Deploy development overlay
kubectl apply -k kustomize/overlays/development/

# Verify deployment
kubectl get pods -n glance-dev
```

#### Production Environment
```bash
# Deploy production overlay
kubectl apply -k kustomize/overlays/production/

# Verify deployment
kubectl get pods -n glance
kubectl get ingress -n glance
kubectl get hpa -n glance
```

### Deploy with ArgoCD

#### Base Deployment
```bash
# Apply ArgoCD application manifest
kubectl apply -f argocd/application-kustomize.yaml

# Check sync status
argocd app get glance-kustomize
argocd app sync glance-kustomize
```

#### Development Environment
```bash
kubectl apply -f argocd/application-kustomize-dev.yaml
```

#### Production Environment
```bash
kubectl apply -f argocd/application-kustomize-prod.yaml
```

## Configuration

### Base Configuration

The base configuration includes:
- **Namespace**: `glance`
- **ServiceAccount**: Basic service account with auto-mount enabled
- **ConfigMap**: Default Glance configuration with sample widgets
- **Deployment**: Single replica with security context and resource limits
- **Service**: ClusterIP service on port 8080

**Key Features:**
- Security hardened (non-root user, read-only filesystem)
- Health checks (liveness and readiness probes)
- Resource limits (CPU: 500m, Memory: 256Mi)
- Temporary storage for writable directories

### Development Overlay

The development overlay modifies the base with:
- **Namespace**: `glance-dev`
- **Image**: `latest` tag with Always pull policy
- **Resources**: Reduced limits (CPU: 250m, Memory: 128Mi)
- **Replicas**: 1

### Production Overlay

The production overlay includes:
- **Replicas**: 2 (minimum)
- **HPA**: Autoscaling from 2 to 10 replicas based on CPU/memory
- **Ingress**: NGINX ingress with TLS support
- **PVC**: 5Gi persistent storage for data
- **Resources**: Increased limits (CPU: 1000m, Memory: 512Mi)

## Customization

### Customize Glance Configuration

Edit the ConfigMap in `base/configmap.yaml` to customize your Glance dashboard:

```yaml
data:
  glance.yml: |
    server:
      port: 8080
      host: 0.0.0.0
    
    pages:
      - name: Home
        columns:
          - size: small
            widgets:
              - type: calendar
              # Add more widgets here
```

### Customize Image Version

Edit `kustomization.yaml` in the appropriate directory:

```yaml
images:
  - name: glanceapp/glance
    newTag: v0.7.0  # Change to desired version
```

Or use the command line:

```bash
cd kustomize/base
kustomize edit set image glanceapp/glance=glanceapp/glance:v0.8.0
```

### Add Environment Variables

Create a patch file or add to the deployment:

```yaml
# In overlays/production/env-patch.yaml
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

Add to `kustomization.yaml`:
```yaml
patches:
  - path: env-patch.yaml
```

### Customize Ingress

Edit `overlays/production/ingress.yaml`:

```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - your-domain.com  # Change this
      secretName: glance-tls
  rules:
    - host: your-domain.com  # Change this
```

### Add Persistent Storage

The production overlay already includes a PVC. To modify:

```yaml
# In overlays/production/pvc.yaml
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi  # Adjust size
  storageClassName: fast-ssd  # Add storage class if needed
```

Then add volume mount to deployment patch:

```yaml
# In overlays/production/deployment-patch.yaml
spec:
  template:
    spec:
      containers:
      - name: glance
        volumeMounts:
        - name: data
          mountPath: /app/data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: glance-data
```

## ArgoCD Integration

### Application Manifests

Three ArgoCD applications are provided:

1. **application-kustomize.yaml** - Base deployment
2. **application-kustomize-dev.yaml** - Development environment
3. **application-kustomize-prod.yaml** - Production environment (manual sync)

### Sync Policies

#### Base and Development
- **Automated sync**: Enabled
- **Prune**: Enabled (removes resources not in Git)
- **Self-heal**: Enabled (corrects manual changes)

#### Production
- **Automated sync**: Enabled with prune disabled
- **Self-heal**: Enabled
- **Manual approval**: Recommended (comment out automated section)

### Update Repository URL

Before deploying, update the `repoURL` in ArgoCD application manifests:

```yaml
spec:
  source:
    repoURL: https://github.com/your-org/glance.git  # Update this
```

### Multiple Environments

Deploy multiple environments by applying different overlays:

```bash
# Development
kubectl apply -f argocd/application-kustomize-dev.yaml

# Production
kubectl apply -f argocd/application-kustomize-prod.yaml
```

## Validation

### Preview Changes

Before applying, preview what will be deployed:

```bash
# View rendered manifests
kubectl kustomize kustomize/base/
kubectl kustomize kustomize/overlays/production/

# Diff with current cluster state
kubectl diff -k kustomize/overlays/production/
```

### Validate ArgoCD Application

```bash
# Dry run
argocd app create glance-kustomize \
  --repo https://github.com/glanceapp/glance.git \
  --path kustomize/base \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace glance \
  --dry-run

# View diff before sync
argocd app diff glance-kustomize
```

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n glance
kubectl describe pod <pod-name> -n glance
kubectl logs <pod-name> -n glance
```

### Check ArgoCD Sync Status
```bash
argocd app get glance-kustomize
argocd app sync glance-kustomize --dry-run
```

### Common Issues

**Image Pull Errors:**
- Verify image tag exists: `docker pull glanceapp/glance:v0.7.0`
- Check imagePullSecrets if using private registry

**Configuration Errors:**
- Validate YAML: `kubectl kustomize kustomize/base/ | kubectl apply --dry-run=client -f -`
- Check ConfigMap: `kubectl get configmap glance-config -n glance -o yaml`

**Permission Errors:**
- Verify ServiceAccount exists: `kubectl get sa -n glance`
- Check security context settings in deployment

**Ingress Not Working:**
- Verify ingress controller is installed: `kubectl get pods -n ingress-nginx`
- Check ingress resource: `kubectl describe ingress glance -n glance`
- Verify DNS is pointing to ingress controller

## Migration from Helm

If migrating from Helm to Kustomize:

1. Export current Helm values:
   ```bash
   helm get values glance -n glance > my-values.yaml
   ```

2. Transfer configuration to Kustomize ConfigMap

3. Update image version in `kustomization.yaml`

4. Apply Kustomize manifests (consider using a different namespace first)

5. Test thoroughly before removing Helm release

## Best Practices

1. **Version Control**: Keep all customizations in Git
2. **Overlays**: Use overlays for environment-specific configs
3. **Secrets**: Use sealed-secrets or external-secrets for sensitive data
4. **Testing**: Always test in development before production
5. **Monitoring**: Add monitoring and alerting for production deployments
6. **Backup**: Backup persistent volumes regularly
7. **Updates**: Use specific image tags, avoid `latest` in production

## Additional Resources

- [Kustomize Documentation](https://kustomize.io/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Glance Documentation](https://github.com/glanceapp/glance)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
