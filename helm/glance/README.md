# Glance Helm Chart

A Helm chart for deploying [Glance](https://github.com/glanceapp/glance) - a self-hosted dashboard that puts all your feeds in one place.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installing the Chart

To install the chart with the release name `glance`:

```bash
helm install glance ./helm/glance
```

The command deploys Glance on the Kubernetes cluster with the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `glance` deployment:

```bash
helm uninstall glance
```

## Parameters

### Global parameters

| Name               | Description                                     | Value |
| ------------------ | ----------------------------------------------- | ----- |
| `replicaCount`     | Number of Glance replicas                       | `1`   |
| `nameOverride`     | String to partially override glance.fullname    | `""`  |
| `fullnameOverride` | String to fully override glance.fullname        | `""`  |

### Image parameters

| Name                | Description                                        | Value              |
| ------------------- | -------------------------------------------------- | ------------------ |
| `image.repository`  | Glance image repository                            | `glanceapp/glance` |
| `image.tag`         | Glance image tag (overrides the appVersion)        | `""`               |
| `image.pullPolicy`  | Glance image pull policy                           | `IfNotPresent`     |
| `imagePullSecrets`  | Specify docker-registry secret names as an array   | `[]`               |

### ServiceAccount parameters

| Name                         | Description                                                        | Value  |
| ---------------------------- | ------------------------------------------------------------------ | ------ |
| `serviceAccount.create`      | Enable creation of ServiceAccount for Glance pod                   | `true` |
| `serviceAccount.automount`   | Automatically mount a ServiceAccount's API credentials             | `true` |
| `serviceAccount.annotations` | Annotations for service account                                    | `{}`   |
| `serviceAccount.name`        | Name of the service account to use                                 | `""`   |

### Security parameters

| Name                                   | Description                                       | Value   |
| -------------------------------------- | ------------------------------------------------- | ------- |
| `podSecurityContext.runAsNonRoot`      | Set container's Security Context runAsNonRoot     | `true`  |
| `podSecurityContext.runAsUser`         | Set container's Security Context runAsUser        | `1000`  |
| `podSecurityContext.runAsGroup`        | Set container's Security Context runAsGroup       | `1000`  |
| `podSecurityContext.fsGroup`           | Set container's Security Context fsGroup          | `1000`  |
| `securityContext.readOnlyRootFilesystem` | Mount container's root filesystem as read-only | `true`  |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation               | `false` |

### Service parameters

| Name                     | Description                       | Value       |
| ------------------------ | --------------------------------- | ----------- |
| `service.type`           | Glance service type               | `ClusterIP` |
| `service.port`           | Glance service port               | `8080`      |
| `service.targetPort`     | Target port on the container      | `8080`      |
| `service.annotations`    | Additional service annotations    | `{}`        |

### Ingress parameters

| Name                  | Description                                              | Value         |
| --------------------- | -------------------------------------------------------- | ------------- |
| `ingress.enabled`     | Enable ingress record generation for Glance              | `false`       |
| `ingress.className`   | IngressClass that will be used                           | `""`          |
| `ingress.annotations` | Additional annotations for the Ingress resource          | `{}`          |
| `ingress.hosts`       | An array with hosts configuration                        | `[...]`       |
| `ingress.tls`         | TLS configuration for ingress                            | `[]`          |

### Resource parameters

| Name                        | Description                       | Value    |
| --------------------------- | --------------------------------- | -------- |
| `resources.limits.cpu`      | CPU resource limits               | `500m`   |
| `resources.limits.memory`   | Memory resource limits            | `256Mi`  |
| `resources.requests.cpu`    | CPU resource requests             | `100m`   |
| `resources.requests.memory` | Memory resource requests          | `128Mi`  |

### Probe parameters

| Name                                    | Description                                      | Value  |
| --------------------------------------- | ------------------------------------------------ | ------ |
| `livenessProbe.httpGet.path`            | Path for the liveness probe                      | `/`    |
| `livenessProbe.httpGet.port`            | Port for the liveness probe                      | `http` |
| `livenessProbe.initialDelaySeconds`     | Initial delay seconds for liveness probe         | `10`   |
| `livenessProbe.periodSeconds`           | Period seconds for liveness probe                | `10`   |
| `readinessProbe.httpGet.path`           | Path for the readiness probe                     | `/`    |
| `readinessProbe.httpGet.port`           | Port for the readiness probe                     | `http` |
| `readinessProbe.initialDelaySeconds`    | Initial delay seconds for readiness probe        | `5`    |
| `readinessProbe.periodSeconds`          | Period seconds for readiness probe               | `10`   |

### Autoscaling parameters

| Name                                         | Description                                              | Value   |
| -------------------------------------------- | -------------------------------------------------------- | ------- |
| `autoscaling.enabled`                        | Enable autoscaling for Glance                            | `false` |
| `autoscaling.minReplicas`                    | Minimum number of Glance replicas                        | `1`     |
| `autoscaling.maxReplicas`                    | Maximum number of Glance replicas                        | `10`    |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization percentage                        | `80`    |

### Other parameters

| Name            | Description                                    | Value  |
| --------------- | ---------------------------------------------- | ------ |
| `nodeSelector`  | Node labels for pod assignment                 | `{}`   |
| `tolerations`   | Tolerations for pod assignment                 | `[]`   |
| `affinity`      | Affinity for pod assignment                    | `{}`   |
| `env`           | Environment variables                          | `[]`   |
| `envFrom`       | Environment variables from ConfigMap or Secret | `[]`   |
| `volumes`       | Additional volumes                             | `[]`   |
| `volumeMounts`  | Additional volume mounts                       | `[]`   |

### Configuration parameters

| Name     | Description                                          | Value       |
| -------- | ---------------------------------------------------- | ----------- |
| `config` | Glance configuration (mounted as `/app/config/glance.yml`) | See values.yaml |

## Configuration

The Glance application is configured via the `config` section in `values.yaml`. This configuration is mounted as `/app/config/glance.yml` in the container.

### Example: Basic Installation

```bash
helm install glance ./helm/glance
```

### Example: With Custom Configuration

Create a `custom-values.yaml` file:

```yaml
config:
  pages:
    - name: Dashboard
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
```

Install with custom values:

```bash
helm install glance ./helm/glance -f custom-values.yaml
```

### Example: With Ingress Enabled

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: glance.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: glance-tls
      hosts:
        - glance.example.com
```

### Example: With Environment Variables

You can pass environment variables to the application for use in the configuration:

```yaml
env:
  - name: GITHUB_TOKEN
    valueFrom:
      secretKeyRef:
        name: glance-secrets
        key: github-token
  - name: WEATHER_LOCATION
    value: "San Francisco, United States"

config:
  pages:
    - name: Home
      columns:
        - size: small
          widgets:
            - type: weather
              location: ${WEATHER_LOCATION}
            - type: releases
              token: ${GITHUB_TOKEN}
              repositories:
                - glanceapp/glance
```

First, create the secret:

```bash
kubectl create secret generic glance-secrets --from-literal=github-token=ghp_your_token_here
```

Then install the chart:

```bash
helm install glance ./helm/glance -f values-with-env.yaml
```

### Example: With Resource Limits

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 512Mi
  requests:
    cpu: 200m
    memory: 256Mi
```

### Example: With Persistence (Optional)

```yaml
persistence:
  enabled: true
  size: 2Gi
  storageClassName: standard
```

## Upgrading

To upgrade the Glance deployment:

```bash
helm upgrade glance ./helm/glance
```

To upgrade with a specific image version:

```bash
helm upgrade glance ./helm/glance --set image.tag=v0.6.0
```

To upgrade with new configuration:

```bash
helm upgrade glance ./helm/glance -f custom-values.yaml
```

## Advanced Configuration

### Using External ConfigMap

If you want to manage the configuration separately, you can create your own ConfigMap and disable the default one:

1. Create your ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-glance-config
data:
  glance.yml: |
    # Your configuration here
```

2. Mount it by adding to your values:

```yaml
volumes:
  - name: config
    configMap:
      name: my-glance-config
```

### Multiple Pages

Glance supports multiple pages (tabs). Configure them in the `config.pages` array:

```yaml
config:
  pages:
    - name: Home
      columns:
        - size: small
          widgets:
            - type: calendar
    - name: News
      columns:
        - size: full
          widgets:
            - type: rss
              feeds:
                - url: https://news.ycombinator.com/rss
```

## Troubleshooting

### Viewing Logs

```bash
kubectl logs -l app.kubernetes.io/name=glance
```

### Configuration Issues

If the application fails to start, check the logs for configuration errors:

```bash
kubectl logs -l app.kubernetes.io/name=glance --tail=100
```

### Accessing the Application

If you're using ClusterIP service (default), use port-forwarding:

```bash
kubectl port-forward svc/glance 8080:8080
```

Then access at http://localhost:8080

### Testing the Chart

To test the chart installation without actually installing it:

```bash
helm install glance ./helm/glance --dry-run --debug
```

To validate the chart:

```bash
helm lint ./helm/glance
```

## Support

For issues, questions, or contributions:
- GitHub Issues: https://github.com/glanceapp/glance/issues
- Documentation: https://github.com/glanceapp/glance/blob/main/docs/configuration.md
- Discord: https://discord.com/invite/7KQ7Xa9kJd

## License

This Helm chart is licensed under the same license as Glance: AGPL-3.0
