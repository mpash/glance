# Customizing Glance Configuration

This guide explains how to customize your Glance dashboard configuration in a Kustomize deployment.

## Overview

Glance configuration is stored in a Kubernetes ConfigMap. To customize it for your deployment, you create a custom ConfigMap in your overlay that patches the base configuration.

## Quick Start

### 1. Create Custom ConfigMap in Your Overlay

Create `configmap.yaml` in your overlay directory:

```bash
# For mitchpash overlay
vim kustomize/overlays/mitchpash/configmap.yaml
```

Add your custom configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: glance-config
  namespace: glance
data:
  glance.yml: |
    server:
      port: 8080
      host: 0.0.0.0

    pages:
      - name: My Dashboard
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
                  - url: https://your-feed.com/rss
                    title: Your Feed
```

### 2. Add ConfigMap Patch to Kustomization

Edit your overlay's `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: glance

resources:
  - ../../base
  - ingress.yaml
  - hpa.yaml
  - pvc.yaml

patches:
  - path: deployment-patch.yaml
    target:
      kind: Deployment
      name: glance
  - path: configmap.yaml  # Add this
    target:
      kind: ConfigMap
      name: glance-config

images:
  - name: glanceapp/glance
    newTag: v0.7.0
```

### 3. Apply and Restart

```bash
# Apply the configuration
kubectl apply -k kustomize/overlays/mitchpash/

# Restart deployment to pick up changes
kubectl rollout restart deployment/glance -n glance

# Watch the rollout
kubectl rollout status deployment/glance -n glance
```

### 4. Verify Changes

```bash
# Check the configmap
kubectl get configmap glance-config -n glance -o yaml

# Check pod logs
kubectl logs -n glance deployment/glance

# Access your dashboard
# https://glance.mitchpash.com
```

## Configuration Options

### Available Widgets

Glance supports many widget types. Here are the most common:

#### Calendar
```yaml
- type: calendar
```

#### Weather
```yaml
- type: weather
  location: City, Country
  units: imperial  # or metric
```

#### RSS Feeds
```yaml
- type: rss
  limit: 15
  collapse-after: 5
  cache: 3h
  feeds:
    - url: https://example.com/feed.xml
      title: Example Feed
```

#### Markets
```yaml
- type: markets
  markets:
    - symbol: AAPL
      name: Apple
    - symbol: BTC-USD
      name: Bitcoin
```

#### Clock
```yaml
- type: clock
  hour-format: 12h  # or 24h
  timezones:
    - timezone: America/New_York
      label: Eastern
    - timezone: Europe/London
      label: London
```

#### Bookmarks
```yaml
- type: bookmarks
  groups:
    - title: Development
      links:
        - title: GitHub
          url: https://github.com
        - title: GitLab
          url: https://gitlab.com
```

#### Monitor
```yaml
- type: monitor
  title: Services
  sites:
    - title: Website
      url: https://example.com
      icon: /assets/icons/website.png
```

#### Server Stats
```yaml
- type: server-stats
  cpu-show-average: true
  disk-devices:
    - /dev/sda1
  network-devices:
    - eth0
```

### Layout Sizes

Columns can be sized:
- `small` - Narrow sidebar
- `full` - Full width
- `medium` - Medium width (not commonly used)

Example layout:
```yaml
pages:
  - name: Home
    columns:
      - size: small      # Left sidebar
        widgets:
          - type: calendar
          - type: weather
      
      - size: full       # Main content
        widgets:
          - type: rss
      
      - size: small      # Right sidebar
        widgets:
          - type: markets
```

## Common Customizations

### Change Dashboard Title

```yaml
pages:
  - name: "Your Custom Name"
    columns:
      # ...
```

### Add Multiple Pages

```yaml
pages:
  - name: Home
    columns:
      # ... home widgets
  
  - name: Monitoring
    columns:
      # ... monitoring widgets
```

### Customize Weather Location

```yaml
- type: weather
  location: San Francisco, United States
  units: metric  # celsius
```

### Add Custom RSS Feeds

```yaml
- type: rss
  limit: 20
  collapse-after: 10
  cache: 1h
  feeds:
    - url: https://hnrss.org/frontpage
      title: Hacker News
    - url: https://www.reddit.com/r/kubernetes/.rss
      title: r/kubernetes
```

### Add Stock Tickers

```yaml
- type: markets
  markets:
    - symbol: SPY
      name: S&P 500
    - symbol: ^VIX
      name: VIX
    - symbol: GC=F
      name: Gold
```

## Workflow for Updates

### Option 1: GitOps (Recommended)

```bash
# 1. Edit config locally
vim kustomize/overlays/mitchpash/configmap.yaml

# 2. Test locally
kubectl apply -k kustomize/overlays/mitchpash/ --dry-run=client

# 3. Commit and push
git add kustomize/overlays/mitchpash/configmap.yaml
git commit -m "Update Glance dashboard configuration"
git push

# 4. ArgoCD will auto-sync (if enabled)
# Or manually sync:
argocd app sync glance-mitchpash

# 5. Restart deployment
kubectl rollout restart deployment/glance -n glance
```

### Option 2: Quick Edit (Development)

```bash
# Edit configmap directly
kubectl edit configmap glance-config -n glance

# Restart to apply
kubectl rollout restart deployment/glance -n glance
```

**Note:** Direct edits will be overwritten on next GitOps sync!

## Troubleshooting

### Configuration Not Updating

```bash
# Check configmap was updated
kubectl get configmap glance-config -n glance -o yaml

# Force restart
kubectl rollout restart deployment/glance -n glance

# Check pod logs for errors
kubectl logs -n glance deployment/glance

# Check if config is mounted
kubectl exec -n glance deployment/glance -- cat /app/config/glance.yml
```

### YAML Syntax Errors

```bash
# Validate before applying
kubectl apply -k kustomize/overlays/mitchpash/ --dry-run=client

# Test kustomize build
kubectl kustomize kustomize/overlays/mitchpash/
```

### Pods Not Starting

```bash
# Check pod status
kubectl get pods -n glance

# Describe pod for events
kubectl describe pod -n glance <pod-name>

# Check logs
kubectl logs -n glance <pod-name>

# Common issues:
# - Invalid YAML indentation
# - Missing required fields
# - Invalid widget configuration
```

## Example Configurations

### Minimal Setup
```yaml
pages:
  - name: Home
    columns:
      - size: full
        widgets:
          - type: calendar
```

### Developer Dashboard
```yaml
pages:
  - name: Dev Dashboard
    columns:
      - size: small
        widgets:
          - type: calendar
          - type: bookmarks
            groups:
              - title: Code
                links:
                  - title: GitHub
                    url: https://github.com
      
      - size: full
        widgets:
          - type: rss
            feeds:
              - url: https://github.com/trending.atom
                title: GitHub Trending
              - url: https://news.ycombinator.com/rss
                title: Hacker News
```

### System Monitor Dashboard
```yaml
pages:
  - name: Monitoring
    columns:
      - size: full
        widgets:
          - type: monitor
            sites:
              - title: Production
                url: https://prod.example.com
              - title: Staging
                url: https://staging.example.com
          
          - type: server-stats
            cpu-show-average: true
```

## Best Practices

1. **Version Control:** Always keep configs in Git
2. **Test First:** Use `--dry-run=client` before applying
3. **Small Changes:** Make incremental updates
4. **Document:** Comment complex configurations
5. **Backup:** Keep old configs for rollback
6. **Validate:** Check pod logs after updates
7. **Cache:** Set appropriate cache times for RSS feeds

## Additional Resources

- [Glance Configuration Docs](../docs/configuration.md)
- [Widget Examples](../docs/glance.yml)
- [Kustomize Documentation](https://kustomize.io/)

## Current Configuration Location

For the mitchpash deployment:
- **ConfigMap:** `kustomize/overlays/mitchpash/configmap.yaml`
- **Applied to:** `glance` namespace
- **Mounted at:** `/app/config/glance.yml` in pods

## Quick Reference Commands

```bash
# Apply config changes
kubectl apply -k kustomize/overlays/mitchpash/

# Restart deployment
kubectl rollout restart deployment/glance -n glance

# View current config
kubectl get configmap glance-config -n glance -o yaml

# Edit config directly (not recommended for prod)
kubectl edit configmap glance-config -n glance

# Check logs
kubectl logs -n glance deployment/glance -f

# Get pod shell
kubectl exec -n glance deployment/glance -it -- sh
```

---

**Last Updated:** January 7, 2026  
**Glance Version:** v0.7.0
