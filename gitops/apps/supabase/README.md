# Supabase

Open-source Firebase alternative providing a PostgreSQL database, authentication, REST/GraphQL APIs, real-time subscriptions, and file storage -- all behind a unified dashboard.

## Resources

- Namespace, HelmRepository, HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `auth.externalUrl` | `https://supabase.vps.kubespaces.cloud` | Public URL for the Auth (GoTrue) API |
| `studio.publicUrl` | `https://supabase.vps.kubespaces.cloud` | Public URL for the management dashboard |
| `kong.enabled` | `true` | Internal API gateway that routes between Supabase microservices (auth, rest, realtime, storage) |
| `dashboard.credentials` | Secret `supabase-dashboard-creds` | Contains username, password, and OpenAI API key for the SQL AI assistant |
| `ingress.enabled` | `false` | Routing handled by HTTPRoute instead |

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: supabase`, interval 10m, timeout 5m, 3 retries.

## Database

No separate database HelmRelease. The Supabase Helm chart bundles its own PostgreSQL instance internally -- this is core to Supabase's architecture since it exposes PostgreSQL directly to users via PostgREST and the client libraries.

## Persistence

No separate PVC defined. The chart manages its own persistence for both the bundled PostgreSQL and the storage service.

## Routing

- **URL:** https://supabase.vps.kubespaces.cloud
- HTTPRoute references the shared Istio gateway in `istio-system`
- Kong handles internal routing between Supabase's microservices (Studio, Auth, REST, Realtime, Storage)
