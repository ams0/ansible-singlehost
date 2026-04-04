# Open WebUI

Web interface for interacting with AI models, providing a ChatGPT-like UI for self-hosted or API-connected LLMs.

## Resources

- `namespace.yaml` - Namespace definition
- `openwebui-helmrepo.yaml` - Helm chart repository source
- `openwebui-helmrelease.yaml` - Flux HelmRelease
- `openwebui-httproute.yaml` - Gateway API HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `ollama.enabled` | `false` | No local LLM inference - connects to external API providers instead |
| `ingress.enabled` | `false` | Uses Gateway API HTTPRoute, not Ingress |

This is a minimal configuration that relies heavily on chart defaults. Open WebUI uses its built-in SQLite database for storing conversations, user accounts, and settings. No external database or explicit PVC is configured.

## Storage

No explicit PVC configured. The chart may provide default persistence for the SQLite database and uploaded files.

## Routing

| Hostname | Route |
|----------|-------|
| `chat.vps.kubespaces.cloud` | HTTPRoute to Open WebUI service |

References the shared Istio gateway in `istio-system`.
