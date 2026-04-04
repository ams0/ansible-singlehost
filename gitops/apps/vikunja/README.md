# Vikunja

Open-source task and project management tool -- a self-hosted alternative to Todoist, Trello, or Microsoft To Do. Supports lists, kanban boards, Gantt charts, labels, reminders, and team collaboration.

## Resources

- Namespace, OCIRepository, Database HelmRelease, Application HelmRelease, HTTPRoute

## HelmRelease Values

| Key | Value | Notes |
|-----|-------|-------|
| `database.type` | `postgres` | Uses PostgreSQL instead of the default SQLite |
| `database.host` | `vikunja-vikunja-database-cluster-rw` | CNPG cluster read-write service |
| `database.credentials` | From CNPG secret | Username and password from the operator-managed secret |
| `service.publicUrl` | `https://todo.vps.kubespaces.cloud` | Uses "todo" subdomain, not "vikunja" |
| `ingress.enabled` | `false` | Routing handled by HTTPRoute instead |
| `mailer.host` | `smtp.gmail.com` | For task reminders, notifications, and invite emails |
| `mailer.password` | From secret | SMTP credentials stored in a Kubernetes secret |
| `defaultSettings.emailRemindersEnabled` | `true` | Users receive email reminders for due tasks |
| `defaultSettings.weekStart` | Monday | Calendar week starts on Monday |
| `defaultSettings.overdueTasksRemindersTime` | `7:00` | Overdue reminders sent at 7 AM |
| `defaultSettings.discoverable_by_name` | `true` | Users can be found by name for collaboration |

The chart is sourced from an **OCIRepository** (`vikunja-ocirepo`) rather than a traditional HelmRepository.

Standard Flux settings: interval 10m, timeout 5m, 3 retries.

## Database

**CNPG PostgreSQL 18 cluster: `vikunja-database`** (standalone, 1 instance)

Vikunja stores tasks, projects, labels, reminders, comments, attachments metadata, and user data in PostgreSQL. Task management with relations, filters, and multi-user collaboration requires a relational database with proper transaction support.

## Persistence

No PVC needed. All data lives in the PostgreSQL database.

## Routing

- **URL:** https://todo.vps.kubespaces.cloud (note: "todo" subdomain, not "vikunja")
- HTTPRoute references the shared Istio gateway in `istio-system`
