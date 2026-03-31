# Velero - Kubernetes Backup

Velero provides application-aware backups of Kubernetes resources and persistent volume data. It backs up both K8s manifests (deployments, services, configmaps, etc.) and PVC file contents via Kopia.

## Architecture

```
Velero (Kopia) → S3 API → RustFS on Mac Mini (feynman:9000) over Tailscale
```

- **Backup storage**: RustFS S3 bucket `velero` on `feynman.rhino-butterfly.ts.net:9000`
- **PVC backup engine**: Kopia (deduplicated, encrypted file-level backups)
- **Plugin**: `velero-plugin-for-aws` for S3-compatible storage
- **Node agent**: DaemonSet that accesses pod volumes for filesystem backup

## Schedules

| Schedule | Cron | TTL | Description |
|----------|------|-----|-------------|
| `daily-all-apps` | `0 2 * * *` (02:00 UTC) | 14 days | All app namespaces |
| `weekly-disaster-recovery` | `0 3 * * 0` (Sunday 03:00 UTC) | 90 days | All app namespaces |

### Backed up namespaces

n8n, forgejo, minecraft, keycloak, actual, alarik, uptime-kuma, outline, waha, rustfs

## Credentials

S3 credentials are stored in the `velero-s3-credentials` secret in the `velero` namespace, created by the Ansible `flux` role. Velero expects an INI-style file format:

```ini
[default]
aws_access_key_id=<key>
aws_secret_access_key=<secret>
```

Vault variables: `vault_velero_s3_access_key`, `vault_velero_s3_secret_key`

## Manual Backup

```bash
kubectl apply -f - <<EOF
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: manual-backup
  namespace: velero
spec:
  includedNamespaces:
    - actual
  defaultVolumesToFsBackup: true
  storageLocation: feynman
  ttl: 72h0m0s
EOF

# Watch progress
kubectl get backup.velero.io manual-backup -n velero -w
```

## Restore

```bash
# List available backups
kubectl get backup.velero.io -n velero

# Restore a backup
kubectl apply -f - <<EOF
apiVersion: velero.io/v1
kind: Restore
metadata:
  name: restore-actual
  namespace: velero
spec:
  backupName: manual-backup
  includedNamespaces:
    - actual
  restorePVs: true
EOF

# Watch progress
kubectl get restore.velero.io restore-actual -n velero -w
```

## k0s-Specific Configuration

k0s uses a non-standard kubelet path. The node-agent must be configured with:

```yaml
nodeAgent:
  podVolumePath: /var/lib/k0s/kubelet/pods
```

Without this, the node-agent crashes with `unexpected directory structure for host-pods volume`.

## Troubleshooting

### BackupStorageLocation shows Unavailable

Check the velero pod logs:

```bash
kubectl logs -n velero -l name=velero -c velero --tail=50
```

Common causes:
- S3 endpoint unreachable (check Tailscale connectivity)
- Invalid credentials (recreate the secret via `ansible-playbook site.yml --tags flux`)
- Bucket doesn't exist on the S3 endpoint

### PVC backup fails with "invalid storage provider"

This happens when `BackupRepository` CRDs are stale (e.g., after switching storage backends). Delete them and retry:

```bash
kubectl delete backuprepositories -n velero --all
```

Velero will recreate them against the current BackupStorageLocation on the next backup.

### FUSE mounts (SSHFS) don't work as hostPath volumes

FUSE filesystems are userspace mounts. Container runtimes (runc/containerd) cannot bind-mount them — you get `no such device` errors. Use kernel-level mounts (CIFS/NFS) or S3-compatible object storage instead.

## Relationship to Borg Backup

Borg and Velero serve complementary roles:

| | Borg | Velero |
|---|------|--------|
| **Scope** | Host filesystem | Kubernetes resources + PVC data |
| **Restores** | Files on disk | Full app (namespace, deployments, services, PVCs) |
| **Target** | Mac Mini via SSH | Mac Mini via S3 (RustFS) |
| **Schedule** | Daily 02:30 UTC | Daily 02:00 UTC + Weekly Sunday 03:00 UTC |
| **Path** | `/Volumes/Store/Backup/VPS/borg` | S3 bucket `velero` on RustFS |

Borg backs up `/opt/local-path-provisioner` (raw PV data on disk) as disaster recovery. Velero provides application-aware restore — it can recreate an entire namespace with all its resources and data in one operation.
