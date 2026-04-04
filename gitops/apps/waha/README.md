# WAHA (WhatsApp HTTP API)

Provides a REST API to interact with WhatsApp. Used for automation and integration with tools like n8n to send/receive WhatsApp messages programmatically.

## Resources

- Namespace, PVC, Deployment, Service, HTTPRoute

## Deployment (Raw Manifests)

WAHA does **not** use a HelmRelease. It is deployed using raw Kubernetes manifests (Deployment + Service) instead of a Helm chart.

| Key | Value | Notes |
|-----|-------|-------|
| `image` | `devlikeapro/waha:arm` | ARM64-specific image tag, with Renovate comment for auto-updates |
| `containerPort` | `3000` | HTTP API port |
| `ephemeral-storage.requests` | `256Mi` | For temp files during media processing |
| `ephemeral-storage.limits` | `512Mi` | Cap on temporary storage usage |

## Persistence

**PVC: `waha-pvc` -- 5Gi, local-path storageClass**

A single PVC is mounted at two subpaths:

| Mount Path | SubPath | Purpose |
|------------|---------|---------|
| `/app/.sessions` | `sessions` | WhatsApp session data. Losing this means re-scanning the QR code and re-authenticating with WhatsApp |
| `/app/.media` | `media` | Downloaded media files from WhatsApp messages |

## Database

None. Session state and media are file-based, stored on the PVC.

## Routing

- **URL:** https://waha.vps.kubespaces.cloud
- HTTPRoute references the shared Istio gateway in `istio-system`
