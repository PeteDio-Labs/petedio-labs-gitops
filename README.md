# PeteDio Labs GitOps - Centralized Control Plane

**Status**: ✅ Production - Migration completed 2025-12-25

This directory contains the **GitOps control plane** for the consolidated [`petedio-labs-gitops`](https://github.com/PeteDio-Labs/petedio-labs-gitops) monorepo. It centralizes ArgoCD Applications, AppProjects, SealedSecrets, Observability infrastructure, and Image Updater configuration, while workload manifests live in sibling directories such as `blog-infrastructure/kubernetes/`, `mission-control-infrastructure/kubernetes/`, and `pete-bot-infrastructure/kubernetes/`.

## Repository Structure

```
petedio-labs-infrastructure/kubernetes/
├── infrastructure/kubernetes/
│   ├── argocd/
│   ├── projects/              # AppProject definitions
│   │   ├── blog-dev.yaml
│   │   ├── blog-stage.yaml
│   │   └── mission-control.yaml
│   │   └── applications/          # ArgoCD Application manifests
│   ├── clusters/
│   ├── image-updater/
│   ├── observability-stack/
│   ├── blog-observability/
│   ├── pete-bot-observability/
│   ├── web-search-observability/
│   ├── web-search-service/
│   └── knowledge/
├── blog-infrastructure/kubernetes/
├── mission-control-infrastructure/kubernetes/
└── pete-bot-infrastructure/kubernetes/
```

## Architecture

### Secret Management
- **Global Secrets**: Image Updater registry credentials in `image-updater/base/registry-sealed-secret.yaml` (argocd namespace)
- **Workload Secrets**: Per-environment in `clusters/{dev,stage}/sealed-secrets/`
- All secrets are SealedSecrets, automatically unsealed by the sealed-secrets controller

### Image Updater
- Monitors container images in Nexus registry (docker.toastedbytes.com)
- Writes image updates to `.argocd-source-blog-{env}.yaml` in `blog-infrastructure/kubernetes/kubernetes/overlays/`
- ArgoCD auto-syncs changes from the `petedio-labs-gitops` monorepo

### Observability
- **Base**: Shared Prometheus/Grafana configuration
- **Overlays**: Environment-specific patches (dev, stage)
- Ingresses: `grafana-dev.toastedbytes.com`, `grafana-stage.toastedbytes.com`

### Service Selector Pattern
**Important**: Kustomize labels should NOT include `includeSelectors: true`. Service patches are used to ensure selectors only match on `app` label (not environment label) to avoid endpoint mismatches.

## Repository Links

- **Monorepo**: https://github.com/PeteDio-Labs/petedio-labs-gitops
- **Control Plane Path**: `infrastructure/kubernetes/`
- **Blog Workloads Path**: `blog-infrastructure/kubernetes/`
