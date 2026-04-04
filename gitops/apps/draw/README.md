# Excalidraw (draw)

Collaborative virtual whiteboard and diagramming tool. Provides a simple, intuitive interface for creating diagrams, wireframes, and sketches directly in the browser.

## Deployment

- **Chart**: `excalidraw` from HelmRepository (deployed under the name "draw")
- **Namespace**: `draw` (HelmRelease lives in `flux-system` with `targetNamespace: draw`)
- **Reconciliation**: every 10m, timeout 5m, 3 retries on install/upgrade

## HelmRelease Values

- **Timezone**: `Europe/Berlin`

Minimal configuration -- Excalidraw is largely a client-side application with very few server-side settings.

## Storage

No database or PVC is needed. Excalidraw is a stateless, client-side application. Drawings live in the browser's local storage or are shared via collaboration links.

## Routing

- **URL**: https://draw.vps.kubespaces.cloud
- **HTTPRoute** references the shared Istio gateway in `istio-system`

## Resources

| File | Purpose |
|------|---------|
| `namespace.yaml` | Namespace definition |
| `draw-helmrepo.yaml` | Helm chart source |
| `draw-helmrelease.yaml` | Application deployment |
| `draw-httproute.yaml` | Gateway API routing |
