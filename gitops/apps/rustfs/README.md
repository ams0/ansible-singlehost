# RustFS

S3-compatible object storage server written in Rust. A lightweight alternative to MinIO for self-hosted S3-compatible storage.

## Resources

- Namespace, PVC, HelmRepository, HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `mode` | `standalone` | Single-node mode; distributed mode disabled |
| `auth.existingSecret` | `rustfs-auth` | Contains access key and secret key |
| `region` | `eu-oracle-1` | Custom region identifier |
| `kms.enabled` | `true` | Server-side encryption for stored objects |
| `kms.backend` | `local` | Encryption keys managed locally |
| `ingress.enabled` | `false` | Routing handled by HTTPRoute instead |

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: rustfs`, interval 10m, timeout 5m, 3 retries.

## Persistence

**PVC: `rustfs-pvc` -- 5Gi, local-path storageClass**

As an object storage server, all uploaded files (buckets and objects) live on this volume. Without it, every restart would lose all stored data.

## Database

None. RustFS uses its own metadata format on disk alongside the object data.

## Routing

- **URL:** https://rustfs.vps.kubespaces.cloud
- **Console:** `/rustfs/console/`
- HTTPRoute references the shared Istio gateway in `istio-system`
