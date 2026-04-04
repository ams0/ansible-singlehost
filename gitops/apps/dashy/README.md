# Dashy

Highly customizable dashboard and start page for the CloudLab homelab. Acts as the central portal linking to all other services running in the cluster.

## Deployment

- **Chart**: `dashy` from HelmRepository
- **Image**: `lissy93/dashy:latest`
- **Namespace**: `dashy` (HelmRelease lives in `flux-system` with `targetNamespace: dashy`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Configuration**: entirely inline via `static.configMapContent` -- the full dashboard config (sections, widgets, links) is defined directly in the HelmRelease values
- **Theme**: `one-dark` with a custom gradient background
- **Dashboard sections include**:
  - Time & Weather
  - GitHub & Dev
  - Tech News
  - Node Metrics (embedded Grafana iframe)
  - Productivity apps
  - AI & Agents
  - Development tools
  - Infrastructure
  - Observability
  - Auth & Security
  - Communication
  - And more

## Storage

No database or PVC is needed. Dashy is a static single-page application that reads its configuration from a Kubernetes ConfigMap generated from the HelmRelease values. All configuration is declarative and lives in Git.

## Routing

- **URL**: https://dash.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `dashy-helmrepo.yaml` | Helm chart source |
| `dashy-helmrelease.yaml` | Application deployment + full dashboard config |
| `dashy-httproute.yaml` | Gateway API routing |
