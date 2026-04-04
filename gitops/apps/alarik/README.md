# Alarik

Notification and alerting platform with a separate console UI for managing alerts, templates, and notification history.

## Deployment

- **Chart**: sourced from OCIRepository (`alarik-ocirepo`)
- **Namespace**: `alarik` (HelmRelease lives in `flux-system` with `targetNamespace: alarik`)
- **Reconciliation**: every 10m, timeout **10m** (longer than the standard 5m), 3 retries on install/upgrade

## HelmRelease Values

- **API base URL**: `https://alarik.vps.kubespaces.cloud`
- **Console base URL**: `https://alarik-console.vps.kubespaces.cloud`
- **Account creation**: disabled (`allowAccountCreation: false`)
- **Auth**: references an existing Kubernetes secret `alarik-credentials`

## Storage

- **PVC**: `alarik-pvc`, 10Gi, `local-path` storageClass

Stores notification history, templates, and configuration data. No external database is needed.

## Routing

Two HTTPRoutes are configured for the separate API and console components:

| Route | URL |
|-------|-----|
| API | https://alarik.vps.kubespaces.cloud |
| Console | https://alarik-console.vps.kuberspaces.cloud |

Both reference the shared Istio gateway in `istio-system`.

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `alarik-ocirepo.yaml` | OCI chart source |
| `alarik-pvc.yaml` | 10Gi persistent volume claim |
| `alarik-helmrelease.yaml` | Application deployment |
| `alarik-httproute.yaml` | Gateway API routing (API) |
| `alarik-console-httproute.yaml` | Gateway API routing (console UI) |
