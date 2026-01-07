# Getting Started with Glance on ArgoCD

This guide walks you through deploying Glance using ArgoCD, from initial setup to production deployment.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start](#quick-start)
3. [Deployment Methods](#deployment-methods)
4. [Step-by-Step Setup](#step-by-step-setup)
5. [Configuration](#configuration)
6. [Verification](#verification)
7. [Common Operations](#common-operations)
8. [Troubleshooting](#troubleshooting)

## Prerequisites

### Required

- **Kubernetes Cluster** (v1.19 or later)
- **ArgoCD Installed** (v2.0 or later)
  ```bash
  # Verify ArgoCD is installed
  kubectl get pods -n argocd
  ```
- **kubectl** configured to access your cluster
- **ArgoCD CLI** (optional but recommended)
  ```bash
  # Install ArgoCD CLI (macOS)
  brew install argocd
  
  # Or download from: https://github.com/argoproj/argo-cd/releases
  ```

### Recommended

- **Access to ArgoCD UI**
  ```bash
  # Port forward to access UI locally
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  
  # Access at: https://localhost:8080
  # Default username: admin
  # Get password: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```

## Quick Start

### Option 1: Using ArgoCD UI (Easiest)

1. **Access ArgoCD UI** at `https://your-argocd-url`

2. **Click "+ NEW APP"**

3. **Fill in the form:**
   - **Application Name:** `glance`
   - **Project:** `default`
   - **Sync Policy:** `Automatic`
   - **Repository URL:** `https://github.com/glanceapp/glance`
   - **Revision:** `HEAD` (or specific version like `v0.7.0`)
   - **Path:** Choose one:
     - `helm/glance` (for Helm deployment)
     - `kustomize/base` (for Kustomize base)
     - `kustomize/overlays/production` (for production)
   - **Cluster:** `https://kubernetes.default.svc` (in-cluster)
   - **Namespace:** `glance`

4. **Click "CREATE"**

5. **Click "SYNC"** to deploy

### Option 2: Using ArgoCD CLI (Recommended)

```bash
# Login to ArgoCD
argocd login localhost:8080

# Create application (Helm)
argocd app create glance \
  --repo https://github.com/glanceapp/glance.git \
  --path helm/glance \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace glance \
  --sync-policy automated \
  --sync-option CreateNamespace=true

# Sync the application
argocd app sync glance

# Watch the sync
argocd app get glance --watch
```

### Option 3: Using kubectl (GitOps Way)

```bash
# Apply the ArgoCD Application manifest
kubectl apply -f https://raw.githubusercontent.com/glanceapp/glance/main/argocd/application.yaml

# Check status
kubectl get application glance -n argocd
```

## Deployment Methods

Glance supports two deployment methods with ArgoCD:

### Method 1: Helm Chart (Recommended for Most Users)

**Best for:** Users who want simple configuration and built-in release management.

```bash
# Using kubectl
kubectl apply -f argocd/application.yaml

# Using ArgoCD CLI
argocd app create glance \
  --repo https://github.com/glanceapp/glance.git \
  --path helm/glance \
  --dest-namespace glance \
  --sync-policy automated
```

**Pros:**
- Simple configuration via values
- Version management
- Easy upgrades
- Rollback support

### Method 2: Kustomize (For Advanced Users)

**Best for:** Users who prefer native Kubernetes manifests and overlay patterns.

```bash
# Base deployment
kubectl apply -f argocd/application-kustomize.yaml

# Development environment
kubectl apply -f argocd/application-kustomize-dev.yaml

# Production environment
kubectl apply -f argocd/application-kustomize-prod.yaml
```

**Pros:**
- Native Kubernetes YAML
- Environment overlays
- Simple patches
- No templating complexity

**See:** [Helm vs Kustomize Comparison](../kustomize/HELM_VS_KUSTOMIZE.md)

## Step-by-Step Setup

### Step 1: Fork or Clone the Repository (Optional)

If you want to customize Glance:

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/glance.git
cd glance
```

### Step 2: Choose Your Configuration

#### For Helm Deployment:

**Option A: Use Default Configuration**

Deploy as-is with defaults (single replica, no ingress):

```bash
kubectl apply -f argocd/application.yaml
```

**Option B: Customize with Values**

1. Download the application manifest:
   ```bash
   curl -O https://raw.githubusercontent.com/glanceapp/glance/main/argocd/application.yaml
   ```

2. Edit the manifest to add custom values:
   ```yaml
   spec:
     source:
       helm:
         values: |
           replicaCount: 2
           
           ingress:
             enabled: true
             className: nginx
             hosts:
               - host: glance.mydomain.com
                 paths:
                   - path: /
                     pathType: Prefix
             tls:
               - secretName: glance-tls
                 hosts:
                   - glance.mydomain.com
           
           resources:
             limits:
               cpu: 1000m
               memory: 512Mi
           
           config:
             pages:
               - name: Home
                 columns:
                   - size: full
                     widgets:
                       - type: calendar
   ```

3. Apply the customized manifest:
   ```bash
   kubectl apply -f application.yaml
   ```

**Option C: Use Example Configurations**

Production-ready example:
```bash
kubectl apply -f argocd/examples/production.yaml
```

Development example:
```bash
kubectl apply -f argocd/examples/development.yaml
```

#### For Kustomize Deployment:

1. **Review the base configuration:**
   ```bash
   kubectl kustomize kustomize/base/
   ```

2. **Choose an overlay:**
   - `base` - Basic deployment
   - `overlays/development` - Dev environment with latest image
   - `overlays/production` - Production with HPA, ingress, PVC

3. **Customize if needed:**
   ```bash
   # Edit the ingress hostname for production
   vim kustomize/overlays/production/ingress.yaml
   
   # Change glance.example.com to your domain
   ```

4. **Apply the ArgoCD application:**
   ```bash
   # For production
   kubectl apply -f argocd/application-kustomize-prod.yaml
   ```

### Step 3: Update Repository URL (If Using Your Fork)

If you forked the repository, update the `repoURL`:

```bash
# Download the manifest
curl -O https://raw.githubusercontent.com/glanceapp/glance/main/argocd/application.yaml

# Edit the file
vim application.yaml

# Change this line:
# repoURL: https://github.com/glanceapp/glance.git
# To:
# repoURL: https://github.com/YOUR_USERNAME/glance.git

# Apply
kubectl apply -f application.yaml
```

### Step 4: Deploy the Application

```bash
# Apply the ArgoCD Application manifest
kubectl apply -f argocd/application.yaml

# Verify it was created
kubectl get application glance -n argocd

# Watch the deployment
kubectl get application glance -n argocd --watch
```

### Step 5: Sync the Application

ArgoCD will automatically sync if you enabled automated sync. Otherwise:

```bash
# Using ArgoCD CLI
argocd app sync glance

# Or click "SYNC" in the ArgoCD UI
```

## Configuration

### Customize Glance Dashboard

The Glance configuration is set in the Helm values or Kustomize ConfigMap.

#### Helm Configuration

Edit your Application manifest:

```yaml
spec:
  source:
    helm:
      values: |
        config:
          pages:
            - name: Home
              columns:
                - size: small
                  widgets:
                    - type: calendar
                    
                    - type: weather
                      location: New York, United States
                
                - size: full
                  widgets:
                    - type: rss
                      feeds:
                        - url: https://news.ycombinator.com/rss
                          title: Hacker News
                        
                        - url: https://github.com/glanceapp/glance/releases.atom
                          title: Glance Releases
                
                - size: small
                  widgets:
                    - type: markets
                      markets:
                        - symbol: SPY
                          name: S&P 500
                        - symbol: QQQ
                          name: NASDAQ
                        - symbol: BTC-USD
                          name: Bitcoin
```

#### Kustomize Configuration

Edit the ConfigMap:

```bash
# Edit the base ConfigMap
vim kustomize/base/configmap.yaml

# Or create an overlay patch
vim kustomize/overlays/production/configmap-patch.yaml
```

### Enable Ingress

#### Helm:

```yaml
spec:
  source:
    helm:
      values: |
        ingress:
          enabled: true
          className: nginx
          annotations:
            cert-manager.io/cluster-issuer: letsencrypt-prod
          hosts:
            - host: glance.yourdomain.com
              paths:
                - path: /
                  pathType: Prefix
          tls:
            - secretName: glance-tls
              hosts:
                - glance.yourdomain.com
```

#### Kustomize:

The production overlay already includes ingress. Just update the hostname:

```bash
vim kustomize/overlays/production/ingress.yaml
```

### Set Resource Limits

#### Helm:

```yaml
spec:
  source:
    helm:
      values: |
        resources:
          limits:
            cpu: 1000m
            memory: 512Mi
          requests:
            cpu: 200m
            memory: 256Mi
```

#### Kustomize:

```bash
vim kustomize/overlays/production/deployment-patch.yaml
```

### Enable Autoscaling

#### Helm:

```yaml
spec:
  source:
    helm:
      values: |
        autoscaling:
          enabled: true
          minReplicas: 2
          maxReplicas: 10
          targetCPUUtilizationPercentage: 80
```

#### Kustomize:

Already included in production overlay. Edit if needed:

```bash
vim kustomize/overlays/production/hpa.yaml
```

## Verification

### Check Application Status

```bash
# Using kubectl
kubectl get application glance -n argocd

# Using ArgoCD CLI
argocd app get glance

# Watch sync progress
argocd app get glance --watch
```

### Check Deployed Resources

```bash
# Check all resources in the namespace
kubectl get all -n glance

# Check specific resources
kubectl get pods -n glance
kubectl get svc -n glance
kubectl get ingress -n glance
kubectl get configmap -n glance
```

### View Application in ArgoCD UI

1. Open ArgoCD UI: `https://your-argocd-url`
2. Find your application in the list
3. Click on it to see the resource tree
4. Check sync status and health

### Test the Application

```bash
# Port forward to access locally
kubectl port-forward -n glance svc/glance 8080:8080

# Open browser to http://localhost:8080
```

Or access via ingress if configured:
```bash
# Check ingress
kubectl get ingress -n glance

# Access via browser at your configured domain
```

### Check Logs

```bash
# View application logs
kubectl logs -n glance deployment/glance

# Stream logs
kubectl logs -n glance deployment/glance -f

# View logs in ArgoCD UI
# Click on the pod in the UI, then "LOGS" tab
```

## Common Operations

### Update to a New Version

#### Using ArgoCD UI:

1. Click on your application
2. Click "APP DETAILS"
3. Click "EDIT"
4. Update "TARGET REVISION" to the new version tag
5. Click "SAVE"
6. Click "SYNC"

#### Using ArgoCD CLI:

```bash
# Update to specific version
argocd app set glance --revision v0.8.0

# Sync the new version
argocd app sync glance
```

#### Using kubectl:

```bash
# Edit the Application manifest
kubectl edit application glance -n argocd

# Update spec.source.targetRevision to the new version
# Save and exit

# ArgoCD will automatically detect and sync the change
```

### Customize Configuration

1. **Edit your Application manifest:**
   ```bash
   kubectl edit application glance -n argocd
   ```

2. **Update the values/patches**

3. **Save and let ArgoCD sync**, or manually sync:
   ```bash
   argocd app sync glance
   ```

### Scale Replicas

#### Helm:

```bash
# Edit application
kubectl edit application glance -n argocd

# Add/update in values:
# replicaCount: 3
```

#### Kustomize:

```bash
# Edit overlay
vim kustomize/overlays/production/deployment-patch.yaml

# Update replicas: 3

# Commit and push if using GitOps
git add .
git commit -m "Scale to 3 replicas"
git push

# ArgoCD will auto-sync
```

### Rollback to Previous Version

#### Using Helm Source:

```bash
# List history
argocd app history glance

# Rollback to specific revision
argocd app rollback glance <revision-number>
```

#### Using Kustomize Source:

```bash
# Revert to previous git commit
git revert HEAD
git push

# Or update targetRevision in Application
kubectl edit application glance -n argocd
```

### Delete the Application

```bash
# Using ArgoCD CLI (removes from ArgoCD only)
argocd app delete glance

# Using ArgoCD CLI (removes from ArgoCD and cluster)
argocd app delete glance --cascade

# Using kubectl
kubectl delete application glance -n argocd
```

### Pause Auto-Sync

```bash
# Disable auto-sync
argocd app set glance --sync-policy none

# Re-enable auto-sync
argocd app set glance --sync-policy automated
```

## Troubleshooting

### Application Won't Sync

**Check sync status:**
```bash
argocd app get glance
```

**Common issues:**

1. **Repository not accessible:**
   ```bash
   # Verify repository URL
   argocd app get glance -o yaml | grep repoURL
   
   # Test repository access
   git clone <repoURL>
   ```

2. **Invalid manifest:**
   ```bash
   # View sync errors
   argocd app get glance
   
   # Check application events
   kubectl describe application glance -n argocd
   ```

3. **Namespace doesn't exist:**
   ```bash
   # Ensure CreateNamespace is enabled
   kubectl edit application glance -n argocd
   
   # Add under syncOptions:
   # - CreateNamespace=true
   ```

### Pods Not Starting

```bash
# Check pod status
kubectl get pods -n glance

# Describe pod for events
kubectl describe pod <pod-name> -n glance

# Check logs
kubectl logs <pod-name> -n glance

# Common issues:
# - Image pull errors: Check image name and tag
# - Configuration errors: Check ConfigMap
# - Resource limits: Check resource quotas
```

### Configuration Not Applied

```bash
# Check ConfigMap
kubectl get configmap glance-config -n glance -o yaml

# Restart pods to pick up changes
kubectl rollout restart deployment/glance -n glance

# Or delete pods to force recreation
kubectl delete pods -n glance -l app=glance
```

### Ingress Not Working

```bash
# Check ingress resource
kubectl get ingress -n glance
kubectl describe ingress glance -n glance

# Check ingress controller
kubectl get pods -n ingress-nginx

# Check TLS certificate
kubectl get certificate -n glance
kubectl describe certificate glance-tls -n glance

# Test DNS resolution
nslookup glance.yourdomain.com

# Test from within cluster
kubectl run test --rm -it --image=curlimages/curl -- curl http://glance.glance.svc.cluster.local:8080
```

### Out of Sync State

```bash
# View differences
argocd app diff glance

# Force sync
argocd app sync glance --force

# Ignore certain fields
kubectl edit application glance -n argocd

# Add ignoreDifferences:
# ignoreDifferences:
#   - group: apps
#     kind: Deployment
#     jsonPointers:
#       - /spec/replicas  # Ignore if using HPA
```

### Permission Issues

```bash
# Check service account
kubectl get sa -n glance

# Check RBAC
kubectl auth can-i list pods --as=system:serviceaccount:glance:glance -n glance

# Check security context
kubectl get pod <pod-name> -n glance -o jsonpath='{.spec.securityContext}'
```

## Advanced Topics

### Multiple Environments

Deploy separate dev, staging, and prod environments:

```bash
# Development
kubectl apply -f argocd/application-kustomize-dev.yaml

# Production
kubectl apply -f argocd/application-kustomize-prod.yaml
```

Each environment can have different:
- Namespaces
- Resource limits
- Replicas
- Configurations
- Image tags

### GitOps Workflow

1. **Fork the repository**
2. **Customize configurations**
3. **Commit changes**
4. **Point ArgoCD to your fork**
5. **Enable auto-sync**
6. **All changes via Git commits**

Example workflow:
```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/glance.git
cd glance

# Create feature branch
git checkout -b update-config

# Make changes
vim kustomize/base/configmap.yaml

# Commit
git add .
git commit -m "Add weather widget"
git push origin update-config

# ArgoCD detects and syncs changes automatically
```

### Using Private Repository

```bash
# Add repository credentials to ArgoCD
argocd repo add https://github.com/YOUR_USERNAME/private-glance.git \
  --username YOUR_USERNAME \
  --password YOUR_GITHUB_TOKEN

# Or use SSH key
argocd repo add git@github.com:YOUR_USERNAME/private-glance.git \
  --ssh-private-key-path ~/.ssh/id_rsa
```

### App of Apps Pattern

Deploy Glance as part of a larger application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_ORG/platform.git
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated: {}
```

Where `apps/` contains multiple application manifests including Glance.

## Best Practices

1. **Use GitOps:** Store all configuration in Git
2. **Pin versions:** Use specific tags instead of `HEAD` or `latest`
3. **Test in dev:** Always test in development before production
4. **Monitor health:** Watch ArgoCD health status and Kubernetes events
5. **Resource limits:** Always set appropriate resource limits
6. **Security:** Use security contexts, run as non-root
7. **Backup:** Backup ConfigMaps and persistent volumes
8. **Documentation:** Document custom configurations

## Next Steps

- Read the [Glance Configuration Guide](../docs/configuration.md)
- Explore [Widget Options](../docs/glance.yml)
- Set up [Monitoring and Alerts](../docs/monitoring.md)
- Configure [Persistent Storage](../kustomize/README.md#add-persistent-storage)
- Enable [Authentication](../internal/glance/auth.go)

## Support

- **GitHub Issues:** https://github.com/glanceapp/glance/issues
- **Documentation:** https://github.com/glanceapp/glance/tree/main/docs
- **ArgoCD Docs:** https://argo-cd.readthedocs.io/

## Summary

You now have Glance running on your Kubernetes cluster via ArgoCD! 🎉

Quick commands to remember:
```bash
# Check status
argocd app get glance

# Sync manually
argocd app sync glance

# View in UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access Glance
kubectl port-forward -n glance svc/glance 8080:8080
```

Happy dashboard building! 📊
