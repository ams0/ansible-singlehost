# Backstage

Spotify's open-source developer portal framework. Provides a unified platform for service catalogs, API documentation, CI/CD visibility, and custom plugins — all behind a single pane of glass for developers.

## Resources

- `namespace.yaml` — Namespace definition
- `backstage-helmrepo.yaml` — HelmRepository (`backstage.github.io/charts`)
- `backstage-db-helmrelease.yaml` — CNPG PostgreSQL 18 cluster
- `backstage-helmrelease.yaml` — Flux HelmRelease (app)
- `backstage-httproute.yaml` — Gateway API HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `backstage.appConfig.app.baseUrl` | `https://backstage.vps.kubespaces.cloud` | Frontend URL |
| `backstage.appConfig.backend.baseUrl` | `https://backstage.vps.kubespaces.cloud` | Backend API URL |
| `backstage.appConfig.backend.database.client` | `pg` | PostgreSQL client |
| `backstage.appConfig.backend.database.connection.host` | `backstage-backstage-database-cluster-rw` | CNPG service |
| `ingress.enabled` | `false` | Routing via HTTPRoute |
| `postgresql.enabled` | `false` | Disables bundled Bitnami PostgreSQL in favor of CNPG |

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: backstage`, interval 10m, timeout 5m, 3 retries.

## Database

**CNPG PostgreSQL 18** (`backstage-database`) — standalone, 1 instance.

Backstage stores catalog entities, API specs, user/group data, scaffolder task history, search indices, and plugin state in PostgreSQL. The chart bundles a Bitnami PostgreSQL subchart by default, but we disable it in favor of the cluster-wide CNPG operator for consistency with other apps.

DB password is injected via `POSTGRES_PASSWORD` environment variable from the CNPG-managed secret.

## Routing

| Hostname | Route |
|----------|-------|
| `backstage.vps.kubespaces.cloud` | HTTPRoute to `backstage-backstage` on port 7007 |

References the shared Istio gateway in `istio-system`.
