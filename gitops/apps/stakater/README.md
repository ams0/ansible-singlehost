# Stakater Reloader

Kubernetes controller that watches for ConfigMap and Secret changes and triggers rolling restarts on Deployments/StatefulSets that reference them. This is a cluster utility, not a user-facing application.

## Resources

- Namespace, HelmRepository, HelmRelease

## HelmRelease Values

No custom values configured -- runs entirely with chart defaults. The controller watches all namespaces for annotation-based opt-in by default.

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: stakater`, interval 10m, timeout 5m, 3 retries.

## Why It Exists

Without Reloader, updating a Secret or ConfigMap would not automatically restart the pods that consume it. Kubernetes only delivers the updated volume/env, but many applications only read config at startup. In a GitOps workflow where secrets and configs change via Git commits, Reloader closes the gap by ensuring pods pick up changes immediately.

## Persistence

None. No PVC or database needed -- Reloader is stateless and reacts to Kubernetes API watch events in real time.

## Routing

None. This is an internal operator with no web UI or API endpoint.
