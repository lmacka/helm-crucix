# Crucix Helm Chart

Helm chart for [Crucix](https://github.com/calesthio/Crucix) — a self-hosted OSINT intelligence aggregation dashboard with a real-time HUD.

## Install

```bash
helm repo add crucix https://lmacka.github.io/helm-crucix/
helm repo update
helm install crucix crucix/crucix -n crucix --create-namespace
```

## Configuration

All OSINT API keys are optional — sources degrade gracefully without them. Pass keys via `extraEnv` or `extraEnvFrom` referencing a Secret.

```bash
helm install crucix crucix/crucix -n crucix --create-namespace \
  --set extraEnv[0].name=FRED_API_KEY \
  --set extraEnv[0].value=your-key \
  --set extraEnv[1].name=REFRESH_INTERVAL_MINUTES \
  --set extraEnv[1].value=30
```

Or with a values file:

```yaml
extraEnv:
  - name: TZ
    value: "Australia/Brisbane"
  - name: REFRESH_INTERVAL_MINUTES
    value: "15"

extraEnvFrom:
  - secretRef:
      name: crucix-secret

persistence:
  enabled: true
  storageClass: longhorn
  size: 1Gi
```

### Optional API Keys

All free registration. Pass via Secret and `extraEnvFrom`:

| Variable | Service |
|----------|---------|
| `FRED_API_KEY` | [Federal Reserve Economic Data](https://fred.stlouisfed.org/docs/api/api_key.html) |
| `FIRMS_MAP_KEY` | [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/api/area/) |
| `EIA_API_KEY` | [US Energy Information Admin](https://api.eia.gov/register) |
| `AISSTREAM_API_KEY` | [AIS Maritime Tracking](https://aisstream.io/) |
| `ACLED_EMAIL` / `ACLED_PASSWORD` | [Armed Conflict Data](https://acleddata.com/register/) |
| `LLM_PROVIDER` / `LLM_API_KEY` | AI-enhanced analysis (anthropic, openai, gemini) |

## Flux / GitOps

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: crucix
spec:
  interval: 60m
  url: https://lmacka.github.io/helm-crucix/
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: crucix
spec:
  interval: 1h
  chart:
    spec:
      chart: crucix
      version: 0.1.1
      sourceRef:
        kind: HelmRepository
        name: crucix
  values:
    persistence:
      enabled: true
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `image.repository` | string | `ghcr.io/calesthio/crucix` | Container image |
| `image.tag` | string | `latest` | Image tag |
| `image.pullPolicy` | string | `Always` | Pull policy |
| `service.type` | string | `ClusterIP` | Service type |
| `service.port` | int | `80` | Service port |
| `service.targetPort` | int | `3117` | Container port |
| `persistence.enabled` | bool | `true` | Enable PVC for alert memory |
| `persistence.storageClass` | string | `""` | Storage class |
| `persistence.accessMode` | string | `ReadWriteOnce` | PVC access mode |
| `persistence.size` | string | `1Gi` | PVC size |
| `persistence.mountPath` | string | `/app/runs` | Mount path |
| `extraEnv` | list | `[]` | Additional env vars |
| `extraEnvFrom` | list | `[]` | Additional env sources (secrets) |
| `ingress.enabled` | bool | `false` | Enable ingress |
| `resources.requests.cpu` | string | `100m` | CPU request |
| `resources.requests.memory` | string | `256Mi` | Memory request |
| `resources.limits.memory` | string | `1Gi` | Memory limit |
| `serviceAccount.create` | bool | `true` | Create service account |
| `serviceAccount.automount` | bool | `false` | Automount SA token |
| `replicaCount` | int | `1` | Replicas |
