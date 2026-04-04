# CodiMD (HackMD CE)

Collaborative real-time markdown editor. Allows multiple users to edit documents simultaneously with live preview, similar to Google Docs but for Markdown.

## Deployment

- **Chart**: `codimd` from HelmRepository
- **Namespace**: `codimd` (HelmRelease lives in `flux-system` with `targetNamespace: codimd`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Service type**: ClusterIP
- **PostgreSQL**: explicitly disabled (`postgresql.enabled: false`) -- running without a persistent database, likely using SQLite or in-memory storage

## Storage

No PVC is configured. Documents may not persist across pod restarts in this minimal configuration.

## Routing

- **URL**: https://codimd.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `codimd-helmrepo.yaml` | Helm chart source |
| `codimd-helmrelease.yaml` | Application deployment |
| `codimd-httproute.yaml` | Gateway API routing |
