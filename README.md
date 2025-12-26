# Homelab Infra GitOps (Option A)

This repo area centralizes ArgoCD control-plane, Image Updater, Observability, and per-environment SealedSecrets while keeping blog workloads in `developed-apps/blog/gitops/kubernetes/...`.

## Layout

- `argocd/projects/` — AppProject definitions (`petedillo-dev`, `petedillo-stage`)
- `argocd/applications/` — Applications (`blog-dev`, `blog-stage`, `observability-*`, `argocd-image-updater`)
- `image-updater/` — Kustomize base + values for ArgoCD Image Updater
- `observability/base|overlays/{dev,stage}` — Prometheus/Grafana and env patches
- `clusters/{dev,stage}/sealed-secrets/` — environment-specific sealed secrets

## Migration Order

1. Dev: sync `argocd/projects`, `argocd/applications` (observability-dev, argocd-image-updater, blog-dev).
2. Validate dev: namespaces, PVCs, Services/Endpoints, Ingress/TLS, NetworkPolicies.
3. Stage: sync `observability-stage`, `blog-stage`; validate like dev.
4. Decommission old ArgoCD app definitions after new ownership is stable.

## Namespace & Ingress Checks (kubectl)

```zsh
# Verify namespaces exist (ArgoCD will create when enabled)
kubectl get ns blog-dev blog-stage observability argocd

# Create missing namespaces proactively if needed
kubectl create ns blog-dev || true
kubectl create ns blog-stage || true
kubectl create ns observability || true

# Check SealedSecrets controller
kubectl get deploy -n kube-system sealed-secrets-controller || \
  kubectl get deploy -A | grep sealed-secrets

# Verify TLS secrets and hosts (dev/stage)
kubectl get secret -n blog-stage cloudflare-origin-cert
kubectl get ingress -n observability grafana

# Storage bindings
kubectl get pvc -A | grep -E "blog-(dev|stage)|observability"
kubectl get pv

# NetworkPolicies
kubectl get netpol -n blog-dev
kubectl get netpol -n blog-stage
```

## ArgoCD Ownership Checks

- Ensure only one `Application` manages each namespace at a time (avoid overlap during cutover).
- Confirm `spec.destination.namespace` and Kustomize overlay paths match existing setup.
- Preserve resource names/selectors; do not rename Services, Deployments, PVCs, or TLS secret names.

## Notes

- Global `registry-sealed-secret.yaml` for Image Updater lives in `image-updater/base` targeting `namespace: argocd`.
- Workload Nexus pull secrets remain per-env in `clusters/<env>/sealed-secrets/nexus-registry.yaml`.
- `.argocd-source-blog-<env>.yaml` remains in blog overlays; Applications use Kustomize without directory recursion.
