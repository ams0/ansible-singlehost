# XWiki

Enterprise wiki platform with structured data, scripting, and extensibility. More powerful (and heavier) than simple wikis like WikiJS -- supports custom applications, macros, and a full extension manager.

## Resources

- Namespace, Database HelmRelease, HelmRepository, Application HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `database.host` | `xwiki-xwiki-database-cluster-rw` | CNPG cluster read-write service |
| `database.password` | From CNPG secret via `customKeyRef` | Password injected from operator-managed secret |

Chart source: `xwiki` from HelmRepository.

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: xwiki`, interval 10m, timeout 5m, 3 retries.

## Database

**CNPG PostgreSQL 18 cluster: `xwiki-database`** (standalone, 1 instance)

XWiki stores wiki pages, attachments metadata, user data, permissions, and extension manager state in PostgreSQL. As a full content management system, it needs ACID transactions for its structured data model -- pages have versions, permissions, and cross-references that must remain consistent.

## Persistence

No separate PVC is defined in the kustomization. XWiki may use chart-default persistence or store content primarily in the database.

## Routing

- **URL:** https://wiki.vps.kubespaces.cloud (note: "wiki" subdomain, not "xwiki")
- HTTPRoute references the shared Istio gateway in `istio-system`
