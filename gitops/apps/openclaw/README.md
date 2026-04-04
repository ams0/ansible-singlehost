# OpenClaw

AI agent platform that runs autonomous agents with skills, browser automation, and tool use.

## Resources

- `namespace.yaml` - Namespace definition
- `openclaw-helmrepo.yaml` - Helm chart repository source
- `openclaw-helmrelease.yaml` - Flux HelmRelease
- `openclaw-httproute.yaml` - Gateway API HTTPRoute

## Architecture

Uses the app-template chart pattern with an init container and main container:

1. **Init container** (`init-skills`) - Installs skills from the ClawHub marketplace at startup. Skills installed: `agent-browser`, `tavily-search`, `find-skills`, `proactive-agent`, `humanizer`, `api-gateway`, `skill-vetter`, `auto-updater`, `free-ride`, `automation-workflows`, `weather`, `gog`.
2. **Main container** - Runs the OpenClaw agent server on gateway port 18789.

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `timeout` | `10m` | Longer than standard - complex deployment |
| Primary AI model | `anthropic/claude-opus-4-6` | Used for agent reasoning |
| Gateway port | `18789` | Local mode |
| Browser automation | enabled | CDP on `localhost:9222` |
| Session scope | per-sender | 60-minute idle reset |
| Agent timeout | 600 seconds | Max execution time per agent call |
| Logging | info level, compact style | Tool call redaction enabled |
| Resource limits | 200m-2 CPU / 1Gi-4Gi memory | Burst-capable for AI workloads |
| Node.js max old space | 3072 MB | Prevents OOM on large agent sessions |
| Security | non-root (UID 1000) | Read-only root filesystem |
| Secrets | from `openclaw-env-secret` | API keys and credentials |
| Allowed origins | `claw.vps.kubespaces.cloud`, `app://localhost` | CORS configuration |

## Configuration

Agent behavior is defined in an `openclaw.json` ConfigMap embedded in the HelmRelease values. This configures the AI model, session handling, browser automation, and installed skills.

## Storage

No separate database or PVC. OpenClaw stores sessions on disk using chart-managed persistence.

## Routing

| Hostname | Route |
|----------|-------|
| `claw.vps.kubespaces.cloud` | HTTPRoute to OpenClaw gateway |

References the shared Istio gateway in `istio-system`.
