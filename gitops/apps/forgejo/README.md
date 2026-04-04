# Forgejo

Community-driven Git forge, a fork of Gitea. Self-hosted alternative to GitHub and GitLab, providing Git hosting, code review, issue tracking, and CI/CD capabilities.

## Deployment

- **Chart**: sourced from OCIRepository (`forgejo-ocirepo`)
- **Namespace**: `forgejo` (HelmRelease lives in `flux-system` with `targetNamespace: forgejo`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Database host**: `forgejo-forgejo-database-cluster-rw`, credentials from CNPG-generated secret
- **Registration**: disabled (`DISABLE_REGISTRATION: true`)
- **Mailer**: `smtp-relay.gmail.com:587` with password from a Kubernetes secret
- **Domain / Root URL**: `code.vps.kubespaces.cloud`

## Database

- **CNPG PostgreSQL 18** cluster (`forgejo-database`), standalone mode, 1 instance

A Git forge needs a relational database for users, repository metadata, issues, pull requests, webhooks, OAuth tokens, and more. Git data (the actual repositories) lives separately on the PVC.

## Storage

- **PVC**: `forgejo-pvc`, 30Gi, `local-path` storageClass

Stores Git repositories (bare repos), LFS objects, avatars, and attachments. At 30Gi, this is the largest app PVC in the cluster because Git history accumulates over time and repository data grows steadily.

## Routing

- **URL**: https://code.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `forgejo-ocirepo.yaml` | OCI chart source |
| `forgejo-database-helmrelease.yaml` | CNPG PostgreSQL cluster |
| `forgejo-pvc.yaml` | 30Gi persistent volume claim |
| `forgejo-helmrelease.yaml` | Application deployment |
| `forgejo-httproute.yaml` | Gateway API routing |
