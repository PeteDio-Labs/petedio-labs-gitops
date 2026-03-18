# GitOps Observability Architecture

Updated: 2026-03-15

## Architecture

Single `observability` namespace. One Grafana instance serves all app dashboards via sidecar ConfigMap discovery. Prometheus scrapes app namespaces through cross-namespace ServiceMonitors.

```
observability namespace:
  ├── kube-prometheus-stack (Helm)
  │   ├── Prometheus (scrapes all ServiceMonitors)
  │   └── Grafana (sidecar discovers ConfigMaps with grafana_dashboard: "1")
  ├── infrastructure-alerts PrometheusRule        ← observability-stack
  ├── grafana Ingress                             ← observability-stack
  ├── blog-api-monitor ServiceMonitor             ← blog-observability
  ├── blog-app-alerts PrometheusRule              ← blog-observability
  ├── blog-grafana-dashboards ConfigMap           ← blog-observability
  ├── discord-bot-monitor ServiceMonitor          ← discord-bot-observability
  ├── discord-bot-alerts PrometheusRule           ← discord-bot-observability
  ├── discord-bot-grafana-dashboards ConfigMap    ← discord-bot-observability
  ├── mission-control-backend-monitor ServiceMonitor  ← mission-control-observability
  ├── mission-control-alerts PrometheusRule            ← mission-control-observability
  └── mission-control-grafana-dashboards ConfigMap     ← mission-control-observability
```

## Repo Mapping

| ArgoCD Application | GitHub Repo | Source Path | Destination |
|---|---|---|---|
| `observability-stack` | `homelab-gitops` | `observability-stack/base` | `observability` |
| `blog-observability-dev` | `homelab-gitops` | `blog-observability/overlays/dev` | `observability` |
| `blog-observability-stage` | `homelab-gitops` | `blog-observability/overlays/stage` | `observability` |
| `discord-bot-observability` | `homelab-gitops` | `discord-bot-observability/overlays/prod` | `observability` |
| `mission-control-observability` | `mission-control-gitops` | `kubernetes/observability/base` | `observability` |

## Namespace Inventory

| Namespace | Role | Observability Resources |
|---|---|---|
| `observability` | Shared monitoring plane | All ServiceMonitors, PrometheusRules, Grafana dashboards, Prometheus, Grafana |
| `blog-dev` | Blog dev workload | Scraped by blog-api-monitor (dev overlay) |
| `blog-stage` | Blog stage workload | Scraped by blog-api-monitor (stage overlay) |
| `discord-bot` | Discord bot workload | Scraped by discord-bot-monitor |
| `mission-control` | Mission Control workload | Scraped by mission-control-backend-monitor |
| `argocd` | GitOps control plane | Hosts Application/AppProject CRs |

## Grafana Access

- **External (dev):** `https://grafana-dev.toastedbytes.com`
- **External (stage):** `https://grafana-stage.toastedbytes.com`
- **Internal:** `http://kube-prom-stack-grafana.observability.svc.cluster.local`

## Mission Control Integration

- **Prometheus URL:** `http://kube-prom-stack-kube-prome-prometheus.observability.svc.cluster.local:9090`
- **Grafana URL:** `https://grafana-dev.toastedbytes.com`
- **Grafana Internal URL:** `http://kube-prom-stack-grafana.observability.svc.cluster.local`
- Configured in: `mission-control-gitops/kubernetes/base/backend/configmap.yaml`

## Dashboard Inventory

| Dashboard | UID | Metrics Source |
|---|---|---|
| Discord Bot Metrics | `discord-bot-metrics` | App-level (prom-client) |
| Mission Control Backend | `mission-control-metrics` | App-level (prom-client) |
| Blog API | `blog-api-metrics` | Container-level (cAdvisor/kube-state-metrics) |

## Alert Rules

### Platform (infrastructure-alerts)
- `NodeDown` — node exporter down >5m (critical)
- `HighDiskUsage` — disk <15% available (warning)
- `PrometheusTargetDown` — API server unreachable >5m (critical)

### Blog (blog-app-alerts)
- `BlogHighMemoryUsage` — memory >90% limit (warning)
- `BlogPodCrashLooping` — restart rate >0.1/15m (warning)
- `BlogHighDatabaseConnections` — pg connections >90 (warning)
- `BlogAPIHighErrorRate` — 5xx >5% in 5m (warning)

### Discord Bot (discord-bot-alerts)
- `DiscordBotDown` — up==0 for 5m (critical)
- `DiscordBotPodNotReady` — pod not ready 5m (critical)
- `DiscordBotHighLatency` — p95 >2s (warning)
- `DiscordBotHighMemoryUsage` — memory >85% limit (warning)
- `DiscordBotCrashLooping` — restart rate >0.1/15m (critical)

### Mission Control (mission-control-alerts)
- `MissionControlBackendPodNotReady` — pod not ready 5m (warning)
- `MissionControlBackendRestarting` — >3 restarts in 15m (warning)

## Notes

- Blog API does not yet expose Prometheus `/metrics` (no micrometer dependency). Blog dashboard uses container-level metrics from cAdvisor/kube-state-metrics. Adding `micrometer-registry-prometheus` to the Spring Boot app will enable app-level metrics.
- All dashboard ConfigMaps use `grafana_dashboard: "1"` label for Grafana sidecar auto-discovery.
- AppProjects `blog-dev` and `blog-stage` allow both `blog-gitops` and `homelab-gitops` as source repos.
