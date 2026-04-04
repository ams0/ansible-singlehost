# Harbor

Cloud-native container registry with vulnerability scanning, RBAC, and replication.

## Resources

- `namespace.yaml` - Namespace definition
- `harbor-pvc.yaml` - 10Gi PVC (local-path) for registry storage
- `harbor-helmrepo.yaml` - Helm chart repository source
- `harbor-helmrelease.yaml` - Flux HelmRelease
- `harbor-httproute.yaml` - Gateway API HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `expose.type` | `clusterIP` | No NodePort or LoadBalancer needed |
| `expose.tls.enabled` | `false` | TLS is terminated at the Istio gateway level |
| `externalURL` | `https://registry.vps.kubespaces.cloud` | Public-facing URL used in Docker login and image paths |
| `database` | internal (bundled) | Uses custom image `ams0/harbor-db:dev` |

Harbor's chart bundles its own PostgreSQL component, so it does not use a separate CNPG database cluster like other apps in this repo.

## Storage

**PVC: 10Gi local-path** (`harbor-pvc`)

Stores the container image registry data: Docker layers, image manifests, and chart artifacts. Without this volume, all pushed container images would be lost on pod restart.

## Routing

| Hostname | Route |
|----------|-------|
| `registry.vps.kubespaces.cloud` | HTTPRoute to Harbor core service |

References the shared Istio gateway in `istio-system`.
