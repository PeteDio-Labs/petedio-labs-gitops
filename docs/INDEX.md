# `petedio-labs-gitops` Docs Index

Central documentation for the `infrastructure/kubernetes/` control-plane directory inside the `petedio-labs-gitops` monorepo.

## Documents

| Document | Description |
|---|---|
| [OBSERVABILITY.md](OBSERVABILITY.md) | Observability architecture — single namespace, Grafana dashboards, ServiceMonitors, PrometheusRules, alert inventory |

## Repository Structure

```
petedio-labs-infrastructure/kubernetes/
├── infrastructure/kubernetes/
│   ├── argocd/
│   ├── blog-observability/
│   ├── clusters/
│   ├── knowledge/
│   ├── image-updater/
│   ├── observability-stack/
│   ├── pete-bot-observability/
│   ├── searxng/
│   ├── web-search-observability/
│   └── web-search-service/
├── blog-infrastructure/kubernetes/
├── mission-control-infrastructure/kubernetes/
└── pete-bot-infrastructure/kubernetes/
```

## Monorepo Directories

| Directory | Local Path | Purpose |
|---|---|---|
| `gitops` | `infrastructure/kubernetes/infrastructure/kubernetes/` | Argo CD control plane, shared observability, platform apps |
| `blog-gitops` | `infrastructure/kubernetes/blog-infrastructure/kubernetes/` | Blog app workload manifests |
| `pete-bot-gitops` | `infrastructure/kubernetes/pete-bot-infrastructure/kubernetes/` | Pete Bot workload manifests |
| `mission-control-gitops` | `infrastructure/kubernetes/mission-control-infrastructure/kubernetes/` | Mission Control workload + observability manifests |
