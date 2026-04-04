# Minecraft

Java Edition game server managed via Helm, using the popular `itzg/minecraft-server` community Docker image that handles server setup automatically.

## Resources

- `namespace.yaml` - Namespace definition
- `minecraft-pvc.yaml` - 20Gi PVC (local-path) for world data
- `minecraft-helmrepo.yaml` - Helm chart repository source
- `minecraft-helmrelease.yaml` - Flux HelmRelease
- `minecraft-httproute.yaml` - Gateway API HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `image.repository` | `itzg/minecraft-server` | Community image with auto-setup |
| `image.tag` | `latest` | Tracks latest stable release |
| `minecraftServer.eula` | `true` | Required to run the server |
| `minecraftServer.motd` | "Welcome to my FluxCD managed Minecraft server!" | Message of the Day |
| `minecraftServer.difficulty` | `normal` | Standard survival difficulty |
| `minecraftServer.gameMode` | `survival` | Default game mode |
| `minecraftServer.maxPlayers` | `10` | Player cap |
| `minecraftServer.pvp` | `true` | Player-vs-player combat enabled |
| `minecraftServer.whitelist` | (none) | Open to all players |
| `minecraftServer.levelName` | `world` | World directory name |
| `rcon.enabled` | `true` | Remote console on port 25575 |
| `rcon.password` | `changeme` | RCON authentication password |
| `serviceType` | `ClusterIP` | Internal service type |
| `timeout` | `10m` | Longer than standard 5m - world generation takes time on first boot |

## Storage

**PVC: 20Gi local-path** (`minecraft-pvc`)

Stores world data, player inventories, configuration files, and plugins. Minecraft worlds grow continuously as players explore new chunks. Losing this volume means losing the entire world, all builds, and player progress.

No database is needed. Minecraft stores everything in region files and NBT data on disk.

## Routing

| Hostname | Route |
|----------|-------|
| `minecraft.vps.kubespaces.cloud` | HTTPRoute (web map or status page) |

References the shared Istio gateway in `istio-system`. Game traffic (TCP port 25565) is handled separately from the HTTP route.
