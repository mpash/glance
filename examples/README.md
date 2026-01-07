# Glance Examples

This directory contains example configurations and personal customizations.

## Personal Customizations

The `personal/` directory contains user-specific customizations that are meant to be copied to personal repositories:

- **[mitchpash/](personal/mitchpash/)** - Example personal deployment with custom services, domain, and configuration

These examples demonstrate how to create your own overlay while keeping the upstream repository clean.

## Using These Examples

### Option 1: Copy to Your Own Repo

```bash
# Copy the example
cp -r examples/personal/mitchpash ~/my-homelab-repo/glance/

# Update kustomization to reference upstream
cd ~/my-homelab-repo/glance/
vim kustomization.yaml
# Change: resources: - https://github.com/glanceapp/glance//kustomize/base?ref=main
```

### Option 2: Fork and Customize

```bash
# Fork the glance repo
# Clone your fork
git clone https://github.com/yourusername/glance.git

# Keep personal/ as your customization area
cd glance/examples/personal/
cp -r mitchpash myname/
vim myname/configmap.yaml
```

### Option 3: Use as Template

Use the personal examples as templates for creating your own overlay structure anywhere.

## What's in Personal Examples

Each personal example includes:
- `kustomization.yaml` - Overlay configuration
- `configmap.yaml` - Custom Glance dashboard config
- `deployment-patch.yaml` - Resource customizations
- `ingress.yaml` - Custom domain configuration
- `hpa.yaml` - Autoscaling configuration
- `pvc.yaml` - Storage configuration
- `application-kustomize-*.yaml` - ArgoCD application
- `README.md` - Specific instructions
- `CONFIGURATION_GUIDE.md` - How to customize

## Creating Your Own

To create your own personal overlay:

1. Copy an existing example
2. Update the domain in `ingress.yaml`
3. Customize `configmap.yaml` with your widgets
4. Update `kustomization.yaml` if needed
5. Apply: `kubectl apply -k examples/personal/yourname/`

## Why Personal Examples?

Personal customizations shouldn't be in the upstream repository because:
- They contain specific domains, services, and configurations
- They're not useful to other users
- They should live in your infrastructure repository
- Keeps the upstream repo clean and focused

The examples here show **how** to customize, not specific personal configs.

## See Also

- [Kustomize Documentation](../kustomize/README.md)
- [ArgoCD Getting Started](../argocd/GETTING_STARTED.md)
- [Helm vs Kustomize](../kustomize/HELM_VS_KUSTOMIZE.md)
