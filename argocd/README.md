# ArgoCD Application Manifests for Glance

This directory contains [ArgoCD](https://argoproj.github.io/cd/) Application manifests for deploying Glance.

## 📖 New to ArgoCD?

**Start here:** [Getting Started Guide](GETTING_STARTED.md) - Complete step-by-step tutorial for deploying Glance with ArgoCD.

## Deployment Options

Glance can be deployed using either:

1. **Helm Chart** (recommended for most users) - `application.yaml`
2. **Kustomize** (for those preferring native Kubernetes manifests) - `application-kustomize*.yaml`

See the [Kustomize README](../kustomize/README.md) for detailed Kustomize documentation.

## Prerequisites

- ArgoCD installed in your Kubernetes cluster
- Access to the ArgoCD namespace
- kubectl configured to access your cluster

## Quick Start

### Helm Deployment (Recommended)

Apply the basic ArgoCD Application manifest:

```bash
kubectl apply -f argocd/application.yaml
```

This will:
- Create the `glance` namespace
- Deploy Glance using the Helm chart from this repository
- Enable automatic sync and self-healing

### Kustomize Deployment

For Kustomize-based deployment:

```bash
# Base deployment
kubectl apply -f argocd/application-kustomize.yaml

# Development environment
kubectl apply -f argocd/application-kustomize-dev.yaml

# Production environment
kubectl apply -f argocd/application-kustomize-prod.yaml
```

See the [Kustomize README](../kustomize/README.md) for more details.

### View Application Status

```bash
# Using kubectl
kubectl get application glance -n argocd

# Using ArgoCD CLI
argocd app get glance

# View in ArgoCD UI
# Navigate to https://your-argocd-url and find the 'glance' application
```

## Available Manifests

### Helm-based Manifests

### `application.yaml` (Basic)
A basic ArgoCD Application manifest with sensible defaults. Good starting point for most deployments.

**Features:**
- Automated sync with prune and self-heal enabled
- Creates namespace automatically
- Uses default values from the Helm chart

**Customization needed:**
- Update `spec.source.repoURL` to your repository URL
- Optionally update `spec.source.targetRevision` to a specific branch/tag
- Uncomment and modify the `values` section for custom configuration

### `examples/production.yaml`
Production-ready configuration with:
- 2 replicas with autoscaling (2-10 pods)
- Ingress with TLS enabled
- Persistent storage (5Gi)
- Increased resource limits
- Environment-specific configuration

### `examples/development.yaml`
Development environment configuration with:
- Single replica
- Latest image tag with Always pull policy
- Staging TLS certificates
- Reduced resource limits
- Separate namespace (glance-dev)

### `examples/git-repository.yaml`
Example using Helm parameters instead of values file:

```yaml
parameters:
  - name: image.tag
    value: v0.7.0
  - name: replicaCount
    value: "1"
```

### `examples/helm-repository.yaml`
Example for using a Helm repository (when the chart is published to a chart repository).

### Kustomize-based Manifests

### `application-kustomize.yaml` (Base)
Base Kustomize deployment with default configuration.

**Features:**
- Automated sync with prune and self-heal enabled
- Deploys to `glance` namespace
- Uses base Kustomize configuration
- Single replica with standard resource limits

### `application-kustomize-dev.yaml` (Development)
Development environment using Kustomize overlay.

**Features:**
- Deploys to `glance-dev` namespace
- Uses `latest` image tag
- Reduced resource limits
- Automatic sync enabled

### `application-kustomize-prod.yaml` (Production)
Production environment with full features.

**Features:**
- HorizontalPodAutoscaler (2-10 replicas)
- Ingress with TLS support
- Persistent storage (5Gi PVC)
- Increased resource limits
- Manual sync recommended (prune disabled)

See the [Kustomize README](../kustomize/README.md) for detailed configuration options.

## Usage Examples

### Deploy Specific Version

1. Copy `application.yaml` to `glance-v0.6.0.yaml`
2. Modify the manifest:

```yaml
spec:
  source:
    targetRevision: v0.6.0  # or specific commit SHA
    helm:
      values: |
        image:
          tag: v0.6.0
```

3. Apply:

```bash
kubectl apply -f glance-v0.6.0.yaml
```

### Deploy with Custom Values

Create a custom Application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: glance-custom
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/glanceapp/glance.git
    targetRevision: main
    path: helm/glance
    helm:
      releaseName: glance
      values: |
        replicaCount: 2
        ingress:
          enabled: true
          hosts:
            - host: my-glance.example.com
              paths:
                - path: /
                  pathType: Prefix
        config:
          pages:
            - name: My Dashboard
              columns:
                - size: small
                  widgets:
                    - type: calendar
  destination:
    server: https://kubernetes.default.svc
    namespace: glance-custom
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Deploy to Multiple Environments

Deploy to different namespaces for different environments:

```bash
# Development
kubectl apply -f argocd/examples/development.yaml

# Production
kubectl apply -f argocd/examples/production.yaml
```

### Using External Values File

If you have a separate values file in your repository:

```yaml
spec:
  source:
    helm:
      valueFiles:
        - values.yaml
        - environments/production-values.yaml
```

### Using Secrets for Sensitive Data

For sensitive configuration like API tokens:

1. Create a Kubernetes Secret:

```bash
kubectl create secret generic glance-secrets \
  --from-literal=github-token=ghp_your_token \
  -n glance
```

2. Reference in your Application:

```yaml
spec:
  source:
    helm:
      values: |
        env:
          - name: GITHUB_TOKEN
            valueFrom:
              secretKeyRef:
                name: glance-secrets
                key: github-token
        config:
          pages:
            - name: Home
              columns:
                - size: full
                  widgets:
                    - type: releases
                      token: ${GITHUB_TOKEN}
                      repositories:
                        - glanceapp/glance
```

## Sync Policies

### Automated Sync

Automatically sync when changes are detected in Git:

```yaml
syncPolicy:
  automated:
    prune: true      # Remove resources that are no longer defined
    selfHeal: true   # Automatically sync when cluster state deviates
```

### Manual Sync

Require manual sync approval:

```yaml
syncPolicy:
  automated: null  # Remove automated section
  syncOptions:
    - CreateNamespace=true
```

Then sync manually:

```bash
argocd app sync glance
```

### Selective Sync

Sync only when specific paths change:

```yaml
spec:
  source:
    path: helm/glance
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Advanced Configuration

### Ignore Differences

Ignore differences in certain fields (e.g., when using HPA):

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
      - /spec/replicas  # Ignore replica count (managed by HPA)
```

### Multi-Source Applications

Use multiple sources (e.g., Helm chart + external values):

```yaml
sources:
  - repoURL: https://github.com/glanceapp/glance.git
    targetRevision: main
    path: helm/glance
  - repoURL: https://github.com/myorg/helm-values.git
    targetRevision: main
    path: glance
    helm:
      valueFiles:
        - production-values.yaml
```

### Health Checks

Custom health checks for resources:

```yaml
spec:
  source:
    helm:
      values: |
        # Enable health checks in Helm values
        livenessProbe:
          httpGet:
            path: /
            port: http
          initialDelaySeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: http
          initialDelaySeconds: 5
```

## Monitoring and Debugging

### Check Application Status

```bash
# Get application details
argocd app get glance

# View application logs
argocd app logs glance

# View sync history
argocd app history glance
```

### Troubleshooting

**Application Not Syncing:**

```bash
# Check application status
kubectl describe application glance -n argocd

# Check ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Force refresh
argocd app get glance --refresh
```

**Sync Failures:**

```bash
# View sync errors
argocd app get glance

# Retry sync
argocd app sync glance --retry-limit 3
```

**Resource Differences:**

```bash
# View differences between Git and cluster
argocd app diff glance
```

## Best Practices

1. **Version Control**: Keep all ArgoCD Application manifests in Git
2. **Separate Environments**: Use different namespaces and Application names for different environments
3. **Automated Sync**: Enable automated sync for development, consider manual sync for production
4. **Prune with Care**: Be cautious with `prune: true` in production to avoid accidental deletions
5. **Use Specific Revisions**: Pin to specific Git tags or commits for production deployments
6. **Health Checks**: Configure proper health checks in your Helm values
7. **Secrets Management**: Use Kubernetes Secrets or external secret managers (e.g., Sealed Secrets, External Secrets Operator)
8. **Resource Limits**: Always set appropriate resource limits and requests

## Integration with CI/CD

### GitOps Workflow

1. **Update Application Manifest**: Modify the ArgoCD Application in Git
2. **Commit Changes**: Commit and push to your repository
3. **ArgoCD Detects Changes**: ArgoCD automatically detects the changes
4. **Automatic Sync**: If automated sync is enabled, ArgoCD applies the changes
5. **Verification**: Monitor the sync status in ArgoCD UI or CLI

### Example CI/CD Pipeline

```yaml
# .github/workflows/deploy.yaml
name: Deploy Glance
on:
  push:
    branches:
      - main
    paths:
      - 'argocd/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Update ArgoCD Application
        run: |
          kubectl apply -f argocd/application.yaml
      
      - name: Wait for Sync
        run: |
          argocd app wait glance --sync --health --timeout 300
```

## Uninstalling

To remove the Glance application:

```bash
# Delete the ArgoCD Application (this will also delete all resources)
kubectl delete application glance -n argocd

# Or using ArgoCD CLI
argocd app delete glance
```

To keep the resources but remove from ArgoCD management:

```bash
argocd app delete glance --cascade=false
```

## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [ArgoCD Application Specification](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml)
- [Glance Helm Chart Documentation](../helm/glance/README.md)
- [Glance Configuration Guide](https://github.com/glanceapp/glance/blob/main/docs/configuration.md)

## Support

For issues related to:
- **ArgoCD manifests**: Open an issue in this repository
- **Helm chart**: See the [Helm chart README](../helm/glance/README.md)
- **Glance application**: Visit the [Glance repository](https://github.com/glanceapp/glance)
