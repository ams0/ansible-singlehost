# Garage

Lightweight S3-compatible distributed object storage written in Rust. Provides an S3 API for applications that need object/blob storage without depending on a cloud provider.

## Deployment

- **Chart**: sourced from a **GitRepository** (`garage-gitrepo`), path `./script/helm/garage` -- this is unusual because most apps use HelmRepository or OCIRepository
- **Image**: `dxflrs/arm64_garage` with a specific commit SHA tag (ARM64-specific build for the Oracle Cloud ARM instance)
- **Namespace**: `garage` (HelmRelease lives in `flux-system` with `targetNamespace: garage`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

**Note**: The GitRepository source is commented out in the kustomization, suggesting this app may be in a transitional state or temporarily disabled.

## HelmRelease Values

- **S3 API root domain**: `s3.vps.kubespaces.cloud`

## Storage

Two separate persistent volumes:

| Volume | Size | Purpose |
|--------|------|---------|
| Meta | 100Mi | Cluster metadata and routing table |
| Data | 5Gi | Actual S3 objects |

Both use the `local-path` storageClass. No external database is needed -- Garage uses its own internal metadata store.

## Routing

- **URL**: https://s3.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `garage-helmrelease.yaml` | Application deployment |
| `garage-httproute.yaml` | Gateway API routing |
| ~~`garage-gitrepo.yaml`~~ | GitRepository source (commented out) |
