# Atlantis

Terraform pull request automation. Atlantis listens for GitHub webhooks on pull requests and runs `terraform plan`/`apply` via PR comments (`atlantis plan`, `atlantis apply`), keeping Terraform workflows in the GitHub UI rather than on operator workstations.

## Deployment

- **Chart**: `atlantis` from HelmRepository
- **Namespace**: `atlantis` (HelmRelease lives in `flux-system` with `targetNamespace: atlantis`)
- **Image**: `ghcr.io/runatlantis/atlantis` (tag pinned in HelmRelease, Renovate-managed)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Org allowlist**: `github.com/ams0/*` — only repos under this owner can trigger plans
- **Atlantis URL**: `https://atlantis.vps.kubespaces.cloud` (used as the webhook callback URL)
- **GitHub user**: `ams0`
- **Service**: ClusterIP on port 80
- **StatefulSet security context**: `fsGroup: 1000`, `runAsUser: 100`

## Secrets

Atlantis expects a Kubernetes Secret named **`atlantis-vcs`** in the `atlantis` namespace, containing:

- `token` — GitHub personal access token with `repo` scope
- `webhook-secret` — shared secret that signs incoming webhook payloads

Create manually (not committed):

```bash
kubectl create secret generic atlantis-vcs -n atlantis \
  --from-literal=token=<github-pat> \
  --from-literal=webhook-secret=<random-string>
```

## GitHub Webhook Setup

In the target repositories, add a webhook:

- **Payload URL**: `https://atlantis.vps.kubespaces.cloud/events`
- **Content type**: `application/json`
- **Secret**: same value as `atlantis-vcs/webhook-secret`
- **Events**: Pull requests, Pull request reviews, Issue comments, Pushes

## Storage

- **PVC**: chart-managed `atlantis-atlantis-data` (`local-path`, via `volumeClaim.enabled: true`) — persists Terraform working directories and plan outputs across pod restarts and Helm upgrades
- A standalone `atlantis-pvc` was removed: the chart never bound it, so it stayed perpetually `Pending` and failed the root Kustomization health check

## Routing

- **URL**: https://atlantis.vps.kubespaces.cloud
- **Backend service**: `atlantis-atlantis` (Helm produces `{release}-{chart}`) on port 80
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `atlantis-helmrepo.yaml` | Helm chart source |
| `atlantis-helmrelease.yaml` | Application deployment (chart-managed PVC) |
| `atlantis-httproute.yaml` | Gateway API routing |
