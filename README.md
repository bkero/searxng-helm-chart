# searxng-helm

A Helm chart for [SearXNG](https://github.com/searxng/searxng), a privacy-respecting metasearch engine.

## Installation

```bash
helm repo add searxng-helm https://bkero.github.io/searxng-helm-chart
helm repo update
helm install searxng searxng-helm/searxng -n searxng --create-namespace
```

## Configuration

All configuration is done via `values.yaml`. Key values:

| Parameter | Description | Default |
|---|---|---|
| `image.tag` | Image tag (defaults to `appVersion`) | `""` |
| `replicaCount` | Number of replicas | `1` |
| `strategy.type` | Deployment strategy | `Recreate` |
| `resources` | CPU/memory requests and limits | See `values.yaml` |
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.className` | Ingress class | `traefik` |
| `ingress.hosts` | Ingress hostnames | `search.privacyplz.org` |
| `env` | Environment variables passed to the container | See `values.yaml` |
| `searxng.settings` | Contents of `settings.yml`, rendered as YAML | See `values.yaml` |

### settings.yml

The `searxng.settings` value is rendered directly into `/etc/searxng/settings.yml` inside the container via a ConfigMap. Any valid SearXNG settings key is supported. Example:

```yaml
searxng:
  settings:
    use_default_settings:
      engines:
        remove:
          - wikidata
    server:
      secret_key: "changeme"
      bind_address: "0.0.0.0:8080"
    redis:
      url: "redis://default:password@redis-master.redis.svc.cluster.local"
```

See the [SearXNG settings documentation](https://docs.searxng.org/admin/settings/index.html) for all available options.

### Secret key

Set `searxng.settings.server.secret_key` in your values, or pass it via the `SEARXNG_SECRET` environment variable (takes precedence):

```yaml
env:
  SEARXNG_SECRET: "your-secret-key"
```

## Upgrading

Bump `version` in `Chart.yaml` and push to `main`. The GitHub Actions workflow will package the chart, create a GitHub Release, and update the Helm repository index on the `gh-pages` branch.
