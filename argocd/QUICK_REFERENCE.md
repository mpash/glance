# ArgoCD Quick Reference for Glance

A quick reference card for deploying and managing Glance with ArgoCD.

## 🚀 Quick Deploy

### Using ArgoCD UI
1. Click **+ NEW APP**
2. Fill in:
   - **Name:** glance
   - **Project:** default
   - **Repo:** `https://github.com/glanceapp/glance`
   - **Path:** `helm/glance` or `kustomize/base`
   - **Namespace:** glance
3. Click **CREATE** → **SYNC**

### Using kubectl
```bash
kubectl apply -f https://raw.githubusercontent.com/glanceapp/glance/main/argocd/application.yaml
```

### Using ArgoCD CLI
```bash
argocd app create glance \
  --repo https://github.com/glanceapp/glance.git \
  --path helm/glance \
  --dest-namespace glance \
  --sync-policy automated \
  --sync-option CreateNamespace=true
```

## 📋 Available Manifests

| Manifest | Type | Environment | Use Case |
|----------|------|-------------|----------|
| `application.yaml` | Helm | Base | Simple deployment with defaults |
| `application-kustomize.yaml` | Kustomize | Base | Native K8s manifests |
| `application-kustomize-dev.yaml` | Kustomize | Development | Dev environment with latest image |
| `application-kustomize-prod.yaml` | Kustomize | Production | Full production setup with HPA/Ingress |
| `examples/production.yaml` | Helm | Production | Production-ready Helm deployment |
| `examples/development.yaml` | Helm | Development | Dev environment Helm deployment |

## 🔧 Common Commands

### Application Management
```bash
# Create application
argocd app create glance --repo URL --path PATH --dest-namespace glance

# Get application status
argocd app get glance

# Sync application
argocd app sync glance

# Delete application
argocd app delete glance

# List all applications
argocd app list
```

### Sync Operations
```bash
# Sync with prune (remove deleted resources)
argocd app sync glance --prune

# Force sync (override sync windows)
argocd app sync glance --force

# Sync specific resource
argocd app sync glance --resource apps:Deployment:glance

# Dry run
argocd app sync glance --dry-run
```

### Configuration
```bash
# Set auto-sync
argocd app set glance --sync-policy automated

# Disable auto-sync
argocd app set glance --sync-policy none

# Set target revision
argocd app set glance --revision v0.7.0

# Set Helm values
argocd app set glance --values values.yaml
```

### Monitoring
```bash
# Watch sync status
argocd app get glance --watch

# View sync history
argocd app history glance

# View diff
argocd app diff glance

# View manifests
argocd app manifests glance
```

### Troubleshooting
```bash
# Get application YAML
argocd app get glance -o yaml

# View sync errors
argocd app get glance

# View application events
kubectl describe application glance -n argocd

# Check application logs
kubectl logs -n glance deployment/glance
```

## 🎯 Common Tasks

### Deploy with Custom Configuration

**Helm:**
```bash
# Create custom values file
cat > my-values.yaml <<EOF
replicaCount: 2
ingress:
  enabled: true
  hosts:
    - host: glance.example.com
EOF

# Deploy with custom values
argocd app create glance \
  --repo https://github.com/glanceapp/glance.git \
  --path helm/glance \
  --dest-namespace glance \
  --values-literal-file my-values.yaml
```

**Or edit Application manifest:**
```bash
kubectl edit application glance -n argocd
# Add under spec.source.helm:
#   values: |
#     replicaCount: 2
#     ingress:
#       enabled: true
```

### Update to New Version
```bash
# Set specific version
argocd app set glance --revision v0.8.0

# Sync the update
argocd app sync glance
```

### Enable Ingress
```bash
kubectl edit application glance -n argocd
# Add under spec.source.helm.values:
#   ingress:
#     enabled: true
#     className: nginx
#     hosts:
#       - host: glance.yourdomain.com
```

### Scale Application
```bash
kubectl edit application glance -n argocd
# Add under spec.source.helm.values:
#   replicaCount: 3
```

### View Application in UI
```bash
# Port forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Open browser to https://localhost:8080
```

### Access Glance
```bash
# Port forward Glance service
kubectl port-forward -n glance svc/glance 8080:8080

# Open browser to http://localhost:8080
```

## 🔍 Health Checks

### Application Health
```bash
# Check overall health
argocd app get glance | grep Health

# Expected: "Health Status: Healthy"
```

### Sync Status
```bash
# Check sync status
argocd app get glance | grep Sync

# Expected: "Sync Status: Synced"
```

### Resource Status
```bash
# Check all resources
kubectl get all -n glance

# Check pods
kubectl get pods -n glance
# Expected: STATUS = Running

# Check service
kubectl get svc -n glance
# Expected: TYPE = ClusterIP

# Check ingress (if enabled)
kubectl get ingress -n glance
```

## ⚙️ Sync Policies

### Automated Sync
```yaml
syncPolicy:
  automated:
    prune: true      # Delete resources not in Git
    selfHeal: true   # Sync when cluster state drifts
    allowEmpty: false
```

### Manual Sync
```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| Application not syncing | Check `argocd app get glance` for errors |
| Pods not starting | Check `kubectl describe pod -n glance` |
| Out of sync | Run `argocd app diff glance` to see changes |
| Permission errors | Check service account and RBAC |
| Ingress not working | Verify ingress controller is installed |
| Configuration not applied | Restart deployment: `kubectl rollout restart deployment/glance -n glance` |

### Quick Diagnostics
```bash
# Full diagnostic
argocd app get glance
kubectl get all -n glance
kubectl describe pod -n glance $(kubectl get pod -n glance -o name | head -1)
kubectl logs -n glance deployment/glance --tail=50
```

## 📊 Status Indicators

### Application Status
- 🟢 **Healthy** - All resources are healthy
- 🟡 **Progressing** - Deployment in progress
- 🟡 **Degraded** - Some resources unhealthy
- 🔴 **Missing** - Resources not found

### Sync Status
- 🟢 **Synced** - Git and cluster match
- 🟡 **OutOfSync** - Git and cluster differ
- 🟡 **Unknown** - Sync status unknown

## 🔗 Quick Links

- **Full Guide:** [GETTING_STARTED.md](GETTING_STARTED.md)
- **Kustomize Guide:** [../kustomize/README.md](../kustomize/README.md)
- **Glance Docs:** [../docs/configuration.md](../docs/configuration.md)
- **ArgoCD Docs:** https://argo-cd.readthedocs.io/

## 💡 Tips

1. **Use automated sync** for non-production environments
2. **Use manual sync** for production environments  
3. **Pin versions** in production (don't use HEAD or latest)
4. **Test in dev** before deploying to production
5. **Monitor sync status** regularly
6. **Use Git** for all configuration changes (GitOps)
7. **Set resource limits** to prevent resource exhaustion
8. **Enable ingress** for external access
9. **Backup ConfigMaps** before major changes
10. **Use ArgoCD notifications** for alerts

## 🎓 Learning Resources

- [ArgoCD Tutorial](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [GitOps Guide](https://www.gitops.tech/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kustomize Docs](https://kustomize.io/)

---

**Need help?** See the [Getting Started Guide](GETTING_STARTED.md) for detailed instructions.
