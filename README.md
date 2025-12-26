# Homelab Infra GitOps - Centralized Control Plane

**Status**: ✅ Production - Migration completed 2025-12-25

This repository serves as the **single source of truth** for the homelab GitOps control plane. It centralizes ArgoCD Applications, AppProjects, SealedSecrets, Observability infrastructure, and Image Updater configuration. Application workloads remain in their respective repositories (e.g., [blog-gitops](https://github.com/petedillo/blog-gitops)).

## Repository Structure

```
homelab-gitops/
├── argocd/
│   ├── projects/              # AppProject definitions
│   │   ├── petedillo-dev.yaml
│   │   └── petedillo-stage.yaml
│   └── applications/          # ArgoCD Application manifests
│       ├── blog-dev.yaml
│       ├── blog-stage.yaml
│       ├── observability-dev.yaml
│       ├── observability-stage.yaml
│       └── argocd-image-updater.yaml
├── clusters/
│   ├── dev/sealed-secrets/    # Dev environment secrets
│   │   ├── admin-credentials.yaml
│   │   ├── blog-app-credentials.yaml
│   │   ├── jwt-secret.yaml
│   │   ├── nexus-registry.yaml
│   │   └── postgres-credentials.yaml
│   └── stage/sealed-secrets/  # Stage environment secrets
│       ├── admin-credentials.yaml
│       ├── blog-app-credentials.yaml
│       ├── cloudflare-cert.yaml
│       ├── jwt-secret.yaml
│       ├── nexus-registry.yaml
│       └── postgres-credentials.yaml
├── image-updater/
│   ├── base/                  # Image Updater Kustomize base
│   │   ├── kustomization.yaml
│   │   └── registry-sealed-secret.yaml
│   └── values/
│       └── values.yaml
└── observability/
    ├── base/                  # Prometheus/Grafana base
    │   ├── grafana-dashboards.yaml
    │   ├── grafana-ingress.yaml
    │   ├── kustomization.yaml
    │   └── prometheus-rules.yaml
    └── overlays/
        ├── dev/
        │   └── kustomization.yaml
        └── stage/
            ├── grafana-ingress-patch.yaml
            ├── kustomization.yaml
            └── prometheus-rules-patch.yaml
```

## Architecture

### Secret Management
- **Global Secrets**: Image Updater registry credentials in `image-updater/base/registry-sealed-secret.yaml` (argocd namespace)
- **Workload Secrets**: Per-environment in `clusters/{dev,stage}/sealed-secrets/`
- All secrets are SealedSecrets, automatically unsealed by the sealed-secrets controller

### Image Updater
- Monitors container images in Nexus registry (docker.toastedbytes.com)
- Writes image updates to `.argocd-source-blog-{env}.yaml` in blog-gitops overlays
- ArgoCD auto-syncs changes from blog-gitops repository

### Observability
- **Base**: Shared Prometheus/Grafana configuration
- **Overlays**: Environment-specific patches (dev, stage)
- Ingresses: `grafana-dev.toastedbytes.com`, `grafana-stage.toastedbytes.com`

### Service Selector Pattern
**Important**: Kustomize labels should NOT include `includeSelectors: true`. Service patches are used to ensure selectors only match on `app` label (not environment label) to avoid endpoint mismatches.

## Repository Links

- **This Repository**: https://github.com/petedillo/homelab-gitops
- **Blog Workloads**: https://github.com/petedillo/blog-gitops
