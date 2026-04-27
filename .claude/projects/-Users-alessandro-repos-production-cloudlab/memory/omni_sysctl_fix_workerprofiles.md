---
name: Omni fs.file-max Sysctl Fix via k0s WorkerProfiles
description: Fixed "SysctlForbidden" errors in Omni pods by configuring kubelet allowedUnsafeSysctls via k0s workerProfiles
type: feedback
---

## Rule

k0s kubelet sysctls must be allowlisted via `spec.workerProfiles[].values.allowedUnsafeSysctls` in ClusterConfig, not via direct kubelet field at spec level.

## Why

Initial attempt to add `spec.kubelet.allowedUnsafeSysctls` directly to ClusterConfig failed with schema validation error: "unknown field 'kubelet'". k0s uses workerProfiles abstraction for kubelet configuration instead of direct field access.

## How to apply

In `k0sctl.yaml.j2` ClusterConfig spec, add:
```yaml
workerProfiles:
  - name: default
    values:
      allowedUnsafeSysctls:
        - {sysctl_name}
```

This allows the kubelet on all worker nodes to permit the specified unsafe sysctl(s) in pod security contexts.

## Verification

- Old Omni pod replicas (pre-fix): SysctlForbidden "fs.file-max" not allowlisted
- New pod replica (post-fix): 0 sysctl errors in events, pod Running 1/1 Ready, service endpoints active
