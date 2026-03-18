# Crucix Helm Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/helm-crucix)](https://artifacthub.io/packages/helm/helm-crucix/crucix)
[![Chart Version](https://img.shields.io/badge/chart-0.2.0-blue)](https://github.com/lmacka/helm-crucix/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](https://github.com/lmacka/helm-crucix/blob/main/LICENSE)

Helm chart for [Crucix](https://github.com/calesthio/Crucix) — a self-hosted OSINT intelligence aggregation dashboard. 27 public sources, real-time Jarvis-style HUD, optional LLM-powered analysis, Telegram/Discord alerting.

## Features

- 27 OSINT sources (GDELT, NASA FIRMS, OpenSky, ACLED, FRED, Yahoo Finance, and more)
- 3D WebGL globe + flat map with live markers
- Optional LLM layer (Anthropic, OpenAI, Gemini, OpenRouter, Codex, MiniMax)
- Two-way Telegram and Discord bots
- Persistent sweep memory via PVC
- All API keys optional — graceful degradation

## Install

```bash
helm repo add crucix https://lmacka.github.io/helm-crucix/
helm repo update
helm install crucix crucix/crucix -n crucix --create-namespace
```

Dashboard available on port 80 (proxied to 3117). First sweep takes ~60 seconds.

## Uninstall

```bash
helm uninstall crucix -n crucix
```

## Configuration

All OSINT API keys are optional — sources degrade gracefully without them. Pass keys via `extraEnv` or a Kubernetes Secret with `extraEnvFrom`.

### Quick start with inline keys

```bash
helm install crucix crucix/crucix -n crucix --create-namespace \
  --set extraEnv[0].name=FRED_API_KEY \
  --set extraEnv[0].value=your-key-here
```

### Using a values file

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

### Using a Kubernetes Secret (recommended)

```bash
kubectl -n crucix create secret generic crucix-secret \
  --from-literal=FRED_API_KEY=your-key \
  --from-literal=FIRMS_MAP_KEY=your-key \
  --from-literal=EIA_API_KEY=your-key
```

Then reference it in values:

```yaml
extraEnvFrom:
  - secretRef:
      name: crucix-secret
```

## API Keys

All free unless noted. None are required — Crucix works with zero keys.

### OSINT Sources

| Variable | Service | Signup |
|----------|---------|--------|
| `FRED_API_KEY` | Federal Reserve Economic Data (yield curve, CPI, VIX, M2) | [fred.stlouisfed.org](https://fred.stlouisfed.org/docs/api/api_key.html) |
| `FIRMS_MAP_KEY` | NASA FIRMS satellite fire/thermal detection | [firms.modaps.eosdis.nasa.gov](https://firms.modaps.eosdis.nasa.gov/api/area/) |
| `EIA_API_KEY` | US Energy Information Admin (crude oil, nat gas) | [api.eia.gov](https://api.eia.gov/register) |
| `AISSTREAM_API_KEY` | Maritime AIS vessel tracking | [aisstream.io](https://aisstream.io/) |
| `ACLED_EMAIL` | Armed Conflict Location & Event Data | [acleddata.com/register](https://acleddata.com/register/) |
| `ACLED_PASSWORD` | *(paired with ACLED_EMAIL)* | *(same registration)* |
| `ADSB_API_KEY` | ADS-B Exchange unfiltered flight tracking (~$10/mo) | [rapidapi.com](https://rapidapi.com/adsbexchange/api/adsbexchange-com1) |
| `REDDIT_CLIENT_ID` | Reddit social sentiment | [reddit.com/prefs/apps](https://www.reddit.com/prefs/apps/) |
| `REDDIT_CLIENT_SECRET` | *(paired with REDDIT_CLIENT_ID, create "script" type)* | *(same page)* |

### LLM Provider (optional — enables AI trade ideas and smarter alerts)

| Variable | Description |
|----------|-------------|
| `LLM_PROVIDER` | `anthropic`, `openai`, `gemini`, `codex`, `openrouter`, or `minimax` |
| `LLM_API_KEY` | Provider API key (not needed for `codex`) |
| `LLM_MODEL` | Override default model (e.g. `claude-sonnet-4-6`, `gpt-5.4`) |

| Provider | Signup | Default Model |
|----------|--------|---------------|
| Anthropic | [console.anthropic.com](https://console.anthropic.com/) | claude-sonnet-4-6 |
| OpenAI | [platform.openai.com](https://platform.openai.com/) | gpt-5.4 |
| Google Gemini | [aistudio.google.com](https://aistudio.google.com/) | gemini-3.1-pro |
| OpenRouter | [openrouter.ai](https://openrouter.ai/) | openrouter/auto |
| MiniMax | [minimaxi.com](https://www.minimaxi.com/) | MiniMax-M2.5 |

### Notifications (optional)

| Variable | Service | How to Get |
|----------|---------|------------|
| `TELEGRAM_BOT_TOKEN` | Telegram bot alerts + commands | Create via [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Telegram chat target | Get via [@userinfobot](https://t.me/userinfobot) |
| `DISCORD_BOT_TOKEN` | Discord bot alerts + slash commands | [Discord Developer Portal](https://discord.com/developers/applications) |
| `DISCORD_CHANNEL_ID` | Discord channel target | Right-click channel (Developer Mode) |
| `DISCORD_GUILD_ID` | Instant slash command registration | Right-click server (Developer Mode) |

### No-Key Sources (18+)

GDELT, OpenSky, Safecast, WHO, OFAC, OpenSanctions, EPA RadNet, NOAA/NWS, USPTO, Bluesky, Telegram OSINT channels, KiwiSDR, CelesTrak, Yahoo Finance, US Treasury, BLS, UN Comtrade, USAspending.

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
      version: 0.2.0
      sourceRef:
        kind: HelmRepository
        name: crucix
  values:
    persistence:
      enabled: true
      storageClass: longhorn
    extraEnvFrom:
      - secretRef:
          name: crucix-secret
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `replicaCount` | int | `1` | Number of replicas |
| `image.repository` | string | `ghcr.io/calesthio/crucix` | Container image |
| `image.tag` | string | `latest` | Image tag |
| `image.pullPolicy` | string | `Always` | Pull policy |
| `imagePullSecrets` | list | `[]` | Image pull secrets |
| `nameOverride` | string | `""` | Override chart name |
| `fullnameOverride` | string | `""` | Override full release name |
| `serviceAccount.create` | bool | `true` | Create service account |
| `serviceAccount.automount` | bool | `false` | Automount SA token |
| `serviceAccount.annotations` | object | `{}` | SA annotations |
| `serviceAccount.name` | string | `""` | SA name override |
| `podAnnotations` | object | `{}` | Pod annotations |
| `podLabels` | object | `{}` | Additional pod labels |
| `podSecurityContext.runAsNonRoot` | bool | `true` | Run as non-root |
| `podSecurityContext.runAsUser` | int | `1000` | Run as UID |
| `podSecurityContext.fsGroup` | int | `1000` | FS group |
| `securityContext.allowPrivilegeEscalation` | bool | `false` | Prevent privilege escalation |
| `service.type` | string | `ClusterIP` | Service type |
| `service.port` | int | `80` | Service port |
| `service.targetPort` | int | `3117` | Container port |
| `ingress.enabled` | bool | `false` | Enable ingress |
| `ingress.className` | string | `""` | Ingress class |
| `ingress.annotations` | object | `{}` | Ingress annotations |
| `ingress.hosts` | list | `[]` | Ingress hosts |
| `ingress.tls` | list | `[]` | Ingress TLS config |
| `resources.requests.cpu` | string | `100m` | CPU request |
| `resources.requests.memory` | string | `256Mi` | Memory request |
| `resources.limits.memory` | string | `1Gi` | Memory limit |
| `persistence.enabled` | bool | `true` | Enable PVC for sweep memory |
| `persistence.storageClass` | string | `""` | Storage class |
| `persistence.accessMode` | string | `ReadWriteOnce` | PVC access mode |
| `persistence.size` | string | `1Gi` | PVC size |
| `persistence.mountPath` | string | `/app/runs` | Mount path for persistent data |
| `extraEnv` | list | `[]` | Additional environment variables |
| `extraEnvFrom` | list | `[]` | Environment from secrets/configmaps |
| `livenessProbe.httpGet.path` | string | `/api/health` | Liveness endpoint |
| `livenessProbe.initialDelaySeconds` | int | `15` | Liveness initial delay |
| `livenessProbe.periodSeconds` | int | `60` | Liveness check interval |
| `readinessProbe.httpGet.path` | string | `/api/health` | Readiness endpoint |
| `readinessProbe.initialDelaySeconds` | int | `5` | Readiness initial delay |
| `readinessProbe.periodSeconds` | int | `10` | Readiness check interval |
| `nodeSelector` | object | `{}` | Node selector |
| `tolerations` | list | `[]` | Tolerations |
| `affinity` | object | `{}` | Affinity rules |

## Links

- [Crucix upstream](https://github.com/calesthio/Crucix)
- [Docker image](https://ghcr.io/calesthio/crucix)
- [Chart source](https://github.com/lmacka/helm-crucix)
- [Issues](https://github.com/lmacka/helm-crucix/issues)
