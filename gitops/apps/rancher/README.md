# Rancher

Kubernetes cluster management UI by SUSE/Rancher Labs. Provides multi-cluster management, monitoring dashboards, and an application catalog.

## Resources

- `namespace.yaml` - Namespace definition (cattle-system)
- `rancher-helmrepo.yaml` - Helm chart repository source
- `rancher-helmrelease.yaml` - Flux HelmRelease
- `rancher-httproute.yaml` - Gateway API HTTPRoute
- `rancher-destinationrule.yaml` - Istio DestinationRule (traffic policy)

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `targetNamespace` | `cattle-system` | Rancher convention, not a generic app namespace |
| `hostname` | `rancher.vps.kubespaces.cloud` | Public-facing URL |
| `replicas` | `1` | Single replica (single-node cluster) |
| `service.type` | `ClusterIP` | Internal service |
| `ingress.enabled` | `false` | Uses Gateway API HTTPRoute instead |
| `tls` | `external` | TLS terminated at the Istio gateway, not by Rancher |

## Storage

No database or PVC needed. Rancher stores all of its state in Kubernetes etcd via Custom Resource Definitions (CRDs) and Secrets. This is by design -- Rancher treats the Kubernetes API as its database.

## Istio DestinationRule

A DestinationRule is included to configure Istio traffic policy for Rancher. This is likely needed to handle Rancher's WebSocket connections (used by the real-time UI) or to configure backend TLS settings between the Istio sidecar and the Rancher pod.

## Routing

| Hostname | Route |
|----------|-------|
| `rancher.vps.kubespaces.cloud` | HTTPRoute to Rancher service |

References the shared Istio gateway in `istio-system`.
