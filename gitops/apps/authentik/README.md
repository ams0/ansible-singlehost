# Authentik

Open-source identity provider (IdP) and SSO platform. A self-hosted alternative to Auth0 and Okta, providing authentication flows, user management, LDAP, SAML, and OAuth2/OIDC.

## Deployment

- **Chart**: `authentik` from HelmRepository
- **Namespace**: `authentik` (HelmRelease lives in `flux-system` with `targetNamespace: authentik`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Gateway**: disabled -- routing is handled by Gateway API HTTPRoute
- **Database host**: `authentik-authentik-database-cluster-rw`
- **Secrets**: both server and worker pods receive:
  - Database password from the CNPG-generated secret
  - Application secret key from `authentik-secret`

## Database

- **CNPG PostgreSQL 18** cluster (`authentik-database`), standalone mode, 1 instance

Authentik requires PostgreSQL to store users, groups, authentication flows, policies, tokens, and audit logs. Identity data is critical infrastructure -- this is not optional.

## Storage

No PVC is needed. Authentik stores all persistent data in PostgreSQL. Media and static files are ephemeral.

## Routing

- **URL**: https://authentik.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `authentik-helmrepo.yaml` | Helm chart source |
| `authentik-database-helmrelease.yaml` | CNPG PostgreSQL cluster |
| `authentik-helmrelease.yaml` | Application deployment (server + worker) |
| `authentik-httproute.yaml` | Gateway API routing |
