# Vault

HashiCorp Vault — secure secrets management and encryption.

## Resources

- Namespace, PVC, HelmRepository, HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `server.replicaCount` | `1` | Single-node standalone mode |
| `server.ha.enabled` | `false` | High availability disabled |
| `server.dataStorage.size` | `10Gi` | Persistent volume for secrets |
| `server.ui.enabled` | `true` | Web UI accessible |
| `ingress.enabled` | `false` | Routing handled by HTTPRoute instead |

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: vault`, interval 10m, timeout 5m, 3 retries.

## Persistence

**PVC: `vault-pvc` -- 10Gi, local-path storageClass**

All encrypted secrets and backend data live on this volume. Without it, Vault state is lost on restart.

## Database

None. Vault uses its integrated storage backend (filesystem via PVC).

## Routing

- **URL:** https://vault.vps.kubespaces.cloud
- **UI:** Root path `/`
- HTTPRoute references the shared Istio gateway in `istio-system`

## Initial Setup

After deployment, unseal Vault using the root token or unseal keys. See Vault documentation for initialization.

## Integration

RustFS and other apps can authenticate to Vault via Kubernetes auth method or static secrets.
