# Matrix

Matrix is an open standard for decentralized, real-time communication. It supports end-to-end encrypted messaging, VoIP, and bridging to other platforms.

## Status: Placeholder / Work-in-Progress

This deployment is **not fully configured**. Only skeleton resources exist.

## Resources

- `namespace.yaml` - Namespace definition
- `matrix-ocirepository.yaml` - OCI chart repository source

## Known Issues

The `kustomization.yaml` in this directory appears to be a copy of Keycloak's kustomization and contains Keycloak-specific references. This is likely a setup error from when the directory was initially scaffolded.

No HelmRelease, database, PVC, or HTTPRoute has been created yet.
