# Actual Budget

Personal finance and budgeting tool, a self-hosted alternative to YNAB and Mint. Provides envelope-style budgeting, transaction tracking, bank syncing, and financial reporting.

## Deployment

- **Chart**: `actualbudget` from HelmRepository
- **Image tag**: 26.3.0
- **Namespace**: `actual` (HelmRelease lives in `flux-system` with `targetNamespace: actual`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Ingress**: disabled -- routing is handled by Gateway API HTTPRoute instead of in-chart ingress

## Storage

- **PVC**: `actual-pvc`, 5Gi, `local-path` storageClass

Actual uses an embedded SQLite database, so no external PostgreSQL or other database is needed. The PVC stores budget data, transaction history, and the SQLite database files. Losing this volume means losing all financial data.

## Routing

- **URL**: https://actual.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `actual-pvc.yaml` | 5Gi persistent volume claim |
| `actual-helmrepo.yaml` | Helm chart source |
| `actual-helmrelease.yaml` | Application deployment |
| `actual-httproute.yaml` | Gateway API routing |
