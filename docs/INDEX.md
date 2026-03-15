# homelab-gitops Docs Index

Central documentation for the homelab-gitops control plane repository.

## Documents

| Document | Description |
|---|---|
| [OBSERVABILITY.md](OBSERVABILITY.md) | Observability architecture — single namespace, Grafana dashboards, ServiceMonitors, PrometheusRules, alert inventory |

## Repository Structure

```
homelab-gitops/
├── argocd/
│   ├── applications/        # ArgoCD Application manifests
│   └── projects/            # AppProject definitions
├── blog-observability/      # Blog tenant monitoring (ServiceMonitor, alerts, dashboard)
│   ├── base/
│   └── overlays/{dev,stage}/
├── clusters/
│   ├── dev/sealed-secrets/
│   └── stage/sealed-secrets/
├── discord-bot-observability/  # Discord bot tenant monitoring
│   ├── base/
│   └── overlays/prod/
├── docs/                    # This folder
├── image-updater/           # ArgoCD Image Updater config
├── mission-control/         # Mission Control ArgoCD app definitions
├── observability-stack/     # Platform-level observability (Grafana ingress, infra alerts)
│   └── base/
└── README.md
```

## Related Repos

| Repo | Local Path | Purpose |
|---|---|---|
| `blog-gitops` | `gitops/blog-gitops/` | Blog app workload manifests |
| `discord-bot-gitops` | `gitops/discord-bot-gitops/` | Discord bot workload manifests |
| `mission-control-gitops` | `gitops/mission-control-gitops/` | Mission Control workload + observability manifests |
