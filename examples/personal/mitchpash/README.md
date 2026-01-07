# Personal Glance Customizations

This directory contains personal customizations for Glance that can be moved to your own repository.

## Structure

```
examples/personal/mitchpash/
├── application-kustomize-mitchpash.yaml  # ArgoCD application
├── configmap.yaml                         # Custom Glance configuration
├── deployment-patch.yaml                  # Resource limits and volumes
├── hpa.yaml                              # HorizontalPodAutoscaler
├── ingress.yaml                          # Ingress with custom domain
├── kustomization.yaml                    # Kustomize overlay
├── pvc.yaml                              # PersistentVolumeClaim
└── CONFIGURATION_GUIDE.md                # How to customize

## Moving to Your Homelab Repo

To use these customizations in your own repository:

1. **Copy this entire directory** to your homelab repo:
   ```bash
   cp -r examples/personal/mitchpash ~/homelab-repo/glance-custom/
   ```

2. **Update the kustomization.yaml** to reference the upstream base:
   ```yaml
   resources:
     # Option 1: Reference from GitHub
     - https://github.com/glanceapp/glance//kustomize/base?ref=main
     
     # Option 2: Reference local clone
     - ../../../glance/kustomize/base
   ```

3. **Update the ArgoCD application**:
   ```yaml
   source:
     repoURL: https://github.com/yourusername/homelab-repo.git
     path: glance-custom
   ```

4. **Apply to your cluster**:
   ```bash
   kubectl apply -f application-kustomize-mitchpash.yaml
   ```

## Using Remote Kustomize Base

For a fully portable setup that doesn't require a local copy of the Glance repo:

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: glance

resources:
  # Reference upstream directly from GitHub
  - https://github.com/glanceapp/glance//kustomize/base?ref=v0.7.0
  - ingress.yaml
  - hpa.yaml
  - pvc.yaml

patches:
  - path: deployment-patch.yaml
    target:
      kind: Deployment
      name: glance
  - path: configmap.yaml
    target:
      kind: ConfigMap
      name: glance-config

images:
  - name: glanceapp/glance
    newTag: v0.7.0
```

## Quick Test

Test the configuration before moving:

```bash
# From the glance repo root
kubectl apply -k examples/personal/mitchpash/ --dry-run=client

# Or with kustomize
kubectl kustomize examples/personal/mitchpash/
```

## What's Included

- **Custom domain**: glance.mitchpash.com with Traefik ingress
- **Monitor widget**: Status dashboard for all your services
- **Markets widget**: Crypto and tech stocks
- **RSS feeds**: Hacker News, Kubernetes, Glance releases
- **Clock widget**: Denver timezone
- **Weather widget**: Denver, US
- **HPA**: Auto-scaling 2-10 replicas
- **Storage**: 5Gi PVC for persistent data

## Files You Can Safely Delete from Upstream

Once you've copied this to your homelab repo, you can delete:
- `examples/personal/mitchpash/` (this entire directory)

This keeps the upstream Glance repo clean and generic for all users.

## See Also

- [Configuration Guide](CONFIGURATION_GUIDE.md) - Full customization guide
- [Kustomize README](../../../kustomize/README.md) - Base documentation
- [ArgoCD Getting Started](../../../argocd/GETTING_STARTED.md) - Deployment guide
