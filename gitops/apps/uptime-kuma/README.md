# Uptime Kuma

Self-hosted monitoring tool for tracking the availability of HTTP endpoints, TCP ports, DNS records, and more. A self-hosted alternative to Pingdom or UptimeRobot with a clean, responsive UI.

## Resources

- Namespace, HelmRepository, PVC, HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `volume.existingClaim` | `uptime-kuma-pvc` | Points to the pre-created PVC for data persistence |

Minimal configuration -- relies on chart defaults for everything else.

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: uptime-kuma`, interval 10m, timeout 5m, 3 retries.

## Persistence

**PVC: `uptime-kuma-pvc` -- 5Gi, local-path storageClass**

Stores the embedded SQLite database containing all monitor definitions, status history, notification configurations, and incident data. Without persistence, every restart would lose all monitoring setup and historical uptime data.

## Database

None. Uptime Kuma uses an embedded SQLite database stored on the PVC. No external database is required.

## Routing

- **URL:** https://uptime.vps.kubespaces.cloud
- HTTPRoute references the shared Istio gateway in `istio-system`
