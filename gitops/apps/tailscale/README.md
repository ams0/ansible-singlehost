# Tailscale Operator

Runs the Tailscale mesh VPN operator in the cluster, enabling secure access to cluster services over the Tailscale network without exposing them to the public internet.

## Resources

- Namespace, HelmRepository, HelmRelease, DNS config (`tailscale-dnsconfig.yaml`), CoreDNS patch (`coredns-patch.yaml`)

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `oauth.clientId` | From secret `tailscale-oauth` | Injected via Flux `valuesFrom`, not stored in Git |
| `oauth.clientSecret` | From secret `tailscale-oauth` | Injected via Flux `valuesFrom`, not stored in Git |

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: tailscale`, interval 10m, timeout 5m, 3 retries.

OAuth credentials are loaded using Flux's `valuesFrom` mechanism, which injects secret values at reconciliation time. This keeps sensitive credentials out of the Git repository entirely.

## DNS Configuration

Two additional manifests configure DNS resolution for Tailscale's MagicDNS names from within the cluster:

- **tailscale-dnsconfig.yaml** -- Configures Tailscale DNS settings
- **coredns-patch.yaml** -- Patches CoreDNS to forward Tailscale domain queries to the Tailscale DNS resolver

This allows pods in the cluster to resolve `*.ts.net` hostnames natively.

## Persistence

None. State is managed by Tailscale's coordination server (control plane). The operator is stateless locally.

## Routing

None. This is a network-level operator, not a web application. It provides connectivity rather than serving HTTP traffic.
