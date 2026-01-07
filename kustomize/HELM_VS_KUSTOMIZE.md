# Helm vs Kustomize for Glance Deployment

This document helps you choose between Helm and Kustomize for deploying Glance with ArgoCD.

## Quick Comparison

| Feature | Helm | Kustomize |
|---------|------|-----------|
| **Complexity** | Higher | Lower |
| **Templating** | Go templates | Patch-based |
| **Package Management** | Yes (Helm charts) | No |
| **Learning Curve** | Steeper | Gentler |
| **Native to Kubernetes** | No | Yes (built into kubectl) |
| **ArgoCD Support** | Excellent | Excellent |
| **Configuration** | values.yaml | Patches & overlays |
| **Upgrades** | Managed by Helm | Manual with Kustomize |
| **Rollback** | Built-in | Via kubectl/ArgoCD |

## When to Use Helm

Choose Helm if you:
- Want a complete package management solution
- Need version management and rollback capabilities
- Prefer centralized configuration (values.yaml)
- Want to use official Helm repositories
- Need complex templating logic
- Are already familiar with Helm

### Helm Pros
- ✅ Built-in release management
- ✅ Easy version upgrades
- ✅ Rollback functionality
- ✅ Chart repositories
- ✅ Mature ecosystem
- ✅ Centralized configuration

### Helm Cons
- ❌ More complex syntax (Go templates)
- ❌ Additional tooling required
- ❌ Steeper learning curve
- ❌ Can be harder to debug

## When to Use Kustomize

Choose Kustomize if you:
- Prefer native Kubernetes manifests
- Want simpler, patch-based configuration
- Like environment-specific overlays
- Don't need package management
- Prefer declarative over templated configs
- Want built-in kubectl support

### Kustomize Pros
- ✅ Native Kubernetes manifests
- ✅ Built into kubectl
- ✅ Simpler syntax
- ✅ Easy to understand and debug
- ✅ Overlay pattern for environments
- ✅ No external dependencies

### Kustomize Cons
- ❌ No built-in package management
- ❌ No version/release tracking
- ❌ Manual rollback management
- ❌ Less mature for complex scenarios

## Configuration Examples

### Helm Configuration

```yaml
# values.yaml
replicaCount: 2
image:
  repository: glanceapp/glance
  tag: v0.7.0
resources:
  limits:
    cpu: 500m
    memory: 256Mi
ingress:
  enabled: true
  hosts:
    - host: glance.example.com
```

### Kustomize Configuration

```yaml
# kustomization.yaml
resources:
  - ../../base
images:
  - name: glanceapp/glance
    newTag: v0.7.0
patches:
  - path: deployment-patch.yaml
```

```yaml
# deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: glance
spec:
  replicas: 2
```

## ArgoCD Integration

Both work excellently with ArgoCD:

### Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: glance
spec:
  source:
    repoURL: https://github.com/glanceapp/glance.git
    path: helm/glance
    helm:
      values: |
        replicaCount: 2
```

### Kustomize Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: glance-kustomize
spec:
  source:
    repoURL: https://github.com/glanceapp/glance.git
    path: kustomize/overlays/production
```

## Common Scenarios

### Scenario 1: Simple Deployment

**Winner: Kustomize** - Simpler setup, fewer moving parts

```bash
kubectl apply -k kustomize/base/
```

vs

```bash
helm install glance helm/glance/
```

### Scenario 2: Multiple Environments

**Tie** - Both handle this well

**Helm:** Multiple values files
```bash
helm install glance-prod helm/glance/ -f values-prod.yaml
helm install glance-dev helm/glance/ -f values-dev.yaml
```

**Kustomize:** Overlays
```bash
kubectl apply -k kustomize/overlays/production/
kubectl apply -k kustomize/overlays/development/
```

### Scenario 3: Complex Configuration

**Winner: Helm** - Better for complex templating

Helm can handle complex logic:
```yaml
{{- if .Values.ingress.enabled }}
  {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
  {{- end }}
{{- end }}
```

### Scenario 4: Quick Customization

**Winner: Kustomize** - Easier to patch specific fields

```bash
cd kustomize/base
kustomize edit set image glanceapp/glance:v0.8.0
kustomize edit set replicas glance=3
```

### Scenario 5: Production with Rollback

**Winner: Helm** - Built-in rollback

```bash
helm upgrade glance helm/glance/
# If something goes wrong:
helm rollback glance
```

## Migration

### From Helm to Kustomize

1. Export current Helm values:
   ```bash
   helm get values glance > my-values.yaml
   ```

2. Create Kustomize patches matching your values

3. Test in a separate namespace

4. Switch over when ready

### From Kustomize to Helm

1. Review your patches and overlays

2. Convert to Helm values.yaml format

3. Test with `helm template`

4. Deploy with Helm

## Recommendation

### For Glance Specifically

**Start with Kustomize if:**
- You're new to Kubernetes
- You want simple, straightforward deployment
- You're comfortable with YAML
- You don't need complex templating

**Start with Helm if:**
- You need version management
- You want rollback capabilities
- You're already using Helm
- You need complex configuration logic

### Best Practice

Many organizations use **both**:
- Helm for complex, third-party applications
- Kustomize for simple, internal applications

For Glance, **either works great**. Choose based on your team's expertise and requirements.

## Getting Help

### Helm Resources
- [Helm Documentation](https://helm.sh/docs/)
- [Glance Helm Chart](../helm/glance/)
- [ArgoCD Helm Applications](../argocd/)

### Kustomize Resources
- [Kustomize Documentation](https://kustomize.io/)
- [Glance Kustomize Setup](../kustomize/)
- [Quick Start Guide](QUICKSTART.md)

## Try Both!

The beauty of having both options is you can:

1. **Start simple** with Kustomize base
2. **Learn the basics** of Kubernetes
3. **Evaluate** if you need Helm's features
4. **Switch** if/when needed

Both are fully supported with ArgoCD, so you can't go wrong!
