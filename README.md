# searxng-helm

A Helm chart for [SearXNG](https://github.com/searxng/searxng), a privacy-respecting metasearch engine.

## Installation

```bash
helm repo add searxng-helm https://bkero.github.io/searxng-helm
helm repo update
helm install searxng searxng-helm/searxng -n searxng --create-namespace \
  --set env.BASE_URL=https://search.example.com \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=search.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

## Configuration

All configuration is via `values.yaml`. Key parameters:

| Parameter | Description | Default |
|---|---|---|
| `image.tag` | Image tag (defaults to `appVersion`) | `""` |
| `replicaCount` | Number of replicas | `1` |
| `strategy.type` | Deployment strategy (`Recreate` or `RollingUpdate`) | `Recreate` |
| `resources` | CPU/memory requests and limits | See `values.yaml` |
| `serviceAccount.create` | Create a ServiceAccount | `true` |
| `serviceAccount.annotations` | Annotations for the ServiceAccount | `{}` |
| `podAnnotations` | Extra annotations added to the pod | `{}` |
| `podSecurityContext` | Pod-level security context | `runAsNonRoot: true, runAsUser: 977` |
| `containerSecurityContext` | Container-level security context | `allowPrivilegeEscalation: false` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class | `""` |
| `ingress.hosts` | Ingress hostnames and paths | See `values.yaml` |
| `ingress.tls` | TLS configuration | `[]` |
| `autoscaling.enabled` | Enable HorizontalPodAutoscaler | `false` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |
| `env` | Environment variables passed to the container | See `values.yaml` |
| `searxng.secretKey` | Secret key for SearXNG (auto-generated if empty) | `""` |
| `searxng.existingSecret` | Use an existing Secret for the secret key | `""` |
| `searxng.existingSecretKey` | Key in the existing Secret | `"secret-key"` |
| `searxng.settings` | Contents of `settings.yml` | See `values.yaml` |

### settings.yml

The `searxng.settings` value is rendered directly into `/etc/searxng/settings.yml` via a ConfigMap. Any valid SearXNG settings key is supported. Example:

```yaml
searxng:
  settings:
    use_default_settings:
      engines:
        keep_only:
          - google
          - duckduckgo
    server:
      bind_address: "0.0.0.0:8080"
    redis:
      url: "redis://default:password@redis-master.redis.svc.cluster.local"
```

See the [SearXNG settings documentation](https://docs.searxng.org/admin/settings/index.html) for all available options.

### Secret key

A secret key is required by SearXNG for session security. This chart manages it as a Kubernetes Secret.

**Auto-generated (default):** If `searxng.secretKey` is empty and no `existingSecret` is set, a random 32-character key is generated on first install and preserved across upgrades via `helm lookup`.

**Explicit value:**
```yaml
searxng:
  secretKey: "your-random-secret-here"
```

**Bring your own Secret:**
```yaml
searxng:
  existingSecret: "my-searxng-secret"
  existingSecretKey: "secret-key"
```

### Ingress with cert-manager

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: search.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: searxng-tls
      hosts:
        - search.example.com
env:
  BASE_URL: "https://search.example.com"
```

### Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80
```

Note: when autoscaling is enabled, `replicaCount` is ignored. Use `Recreate` strategy only with `replicaCount: 1`; switch to `RollingUpdate` when scaling beyond one replica.

## Upgrading

Bump `version` in `Chart.yaml` and push to `main`. The GitHub Actions workflow will lint the chart, package it, create a GitHub Release, and update the Helm repository index on the `gh-pages` branch.
