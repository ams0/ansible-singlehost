# Omni (Sidero Labs)

Talos Linux cluster management platform. Provides a web UI and API to provision, manage, and monitor Talos-based Kubernetes clusters.

## Resources

- `namespace.yaml` - Namespace definition
- `omni-ocirepo.yaml` - OCI chart repository source
- `omni-helmrelease.yaml` - Flux HelmRelease (includes inline PVC and HTTPRoutes)

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `account` | `cloudlab` (id: `1b589eb7-...`) | Sidero Labs account |
| Auth | OIDC via Authentik | Provider URL: `authentik.vps.kubespaces.cloud` |
| OIDC client secret | from `omni-oidc` Kubernetes secret | Loaded at runtime |
| `initial-users` | `alessandro.vozza@linux.com` | First admin user |
| API | `https://omni.vps.kubespaces.cloud` | Management API endpoint |
| K8s Proxy | `https://omni-k8s.vps.kubespaces.cloud` | Kubernetes API proxy |
| SideroLink API | `https://omni-siderolink.vps.kubespaces.cloud` | Machine registration |
| SideroLink join | strict token mode | WireGuard endpoint TBD |
| Workload Proxy | disabled | Not currently in use |
| etcd encryption | GPG key from `omni-etcd-encryption-key` secret | Encryption at rest |

## Storage

**PVC: 10Gi local-path** (defined inline in HelmRelease, not a separate file)

Stores etcd data, the SQLite database (`/data/secondary-storage/sqlite.db`), and machine metadata. Omni uses SQLite rather than an external database, so this volume contains all persistent state.

No external database is needed.

## Routing

Omni defines **three separate HTTPRoutes** inline in the HelmRelease values (no separate HTTPRoute file):

| Hostname | Purpose |
|----------|---------|
| `omni.vps.kubespaces.cloud` | Web UI and management API |
| `omni-k8s.vps.kubespaces.cloud` | Kubernetes API proxy for managed clusters |
| `omni-siderolink.vps.kubespaces.cloud` | SideroLink machine registration API |

All routes reference the shared Istio gateway in `istio-system`.
