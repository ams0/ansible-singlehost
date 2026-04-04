# Wekan

Open-source kanban board -- a self-hosted alternative to Trello. Supports boards, lists, cards, checklists, labels, and swimlanes.

## Resources

- Namespace, HelmRepository, HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `image.tag` | `v8.42` | Pinned -- v8.45+ has Node.js v22 and filesystem issues |
| `ingress.enabled` | `false` | Routing handled by HTTPRoute instead |
| `rootUrl` | `https://wekan.vps.kubespaces.cloud` | Used for OAuth callbacks and email links |
| `mongodb.url` | From `secretEnv` | `mongodb://wekan-wekan-mongodb:27017/wekan` -- points to the chart's built-in MongoDB StatefulSet |
| `secretManaged` | `true` | MongoDB URL stored as a managed secret |
| `mail.url` | `smtp://localhost:25` | Relies on host-level mail relay |
| `mail.from` | `wekan@vps.kubespaces.cloud` | Sender address for notifications |
| `writable_path` | `/data` | Directory for uploaded files |
| `sharedDataFolder.enabled` | `true` | 5Gi local-path volume mounted at `/data` |

Chart source: `wekan` from HelmRepository (`wekan.github.io/charts`).

Standard Flux settings: deployed to `flux-system` namespace with `targetNamespace: wekan`, interval 10m, timeout 5m, 3 retries.

## Database

**MongoDB** (bundled as a subchart in the Wekan Helm chart)

Wekan is one of the few apps in this cluster that uses MongoDB instead of PostgreSQL. This is because Wekan is built on Meteor.js, which traditionally uses MongoDB as its data store. The MongoDB instance runs as a StatefulSet managed by the Helm chart.

## Persistence

The shared data folder (5Gi, local-path) at `/data` stores uploaded attachments and card content. MongoDB persistence is managed by the subchart.

## Routing

- **URL:** https://wekan.vps.kubespaces.cloud
- HTTPRoute references the shared Istio gateway in `istio-system`
