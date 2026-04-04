# Coder

Cloud development environment platform. Provides remote dev workspaces (VS Code, JetBrains) running as Kubernetes pods, giving developers consistent, reproducible environments.

## Deployment

- **Chart**: `coder` from HelmRepository
- **Namespace**: `coder` (HelmRelease lives in `flux-system` with `targetNamespace: coder`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Service type**: ClusterIP
- **Access URL**: `https://coder.vps.kubespaces.cloud`
- **Resource requests**: 250m CPU, 512Mi memory (limits commented out)
- **Database connection**: `CODER_PG_CONNECTION_URL` sourced from the CNPG-generated secret's `uri` key

## Database

- **CNPG PostgreSQL 18** cluster (`coder-database`), standalone mode, 1 instance

Coder stores workspace definitions, user accounts, templates, provisioner state, and audit logs in PostgreSQL. The connection string is injected as an environment variable.

## Storage

No PVC is needed. Workspace state and metadata live in the database. Actual dev environments run as separate pods with their own storage.

## Routing

- **URL**: https://coder.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `coder-helmrepo.yaml` | Helm chart source |
| `coder-database-helmrelease.yaml` | CNPG PostgreSQL cluster |
| `coder-helmrelease.yaml` | Application deployment |
| `coder-httproute.yaml` | Gateway API routing |
