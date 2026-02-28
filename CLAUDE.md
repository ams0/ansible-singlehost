# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CloudLab manages a single Oracle Cloud (OCI) ARM64 host using **Terraform** (infra provisioning), **Ansible** (host configuration), and **Flux CD** (Kubernetes GitOps). The Kubernetes distribution is **k0s** with **Istio** service mesh and **Gateway API** for routing.

Domain: `*.vps.kubespaces.cloud`

## Commands

### Ansible
```bash
# Full deployment
ansible-playbook site.yml

# Run specific roles by tag
ansible-playbook site.yml --tags packages
ansible-playbook site.yml --tags k0s,flux
ansible-playbook site.yml --tags borg,backup

# Dry run
ansible-playbook site.yml --check --diff

# Test connectivity
ansible oracle_hosts -m ping
```

Available tags: `common`, `packages`, `fail2ban`/`security`, `cron`, `docker`, `traefik`/`ingress`, `tailscale`, `borg`/`backup`, `datadog`/`monitoring`, `k0s`/`kubernetes`, `flux`/`gitops`, `flux-webhook`, `claude-code`/`tools`

### Terraform
```bash
cd terraform && terraform init && terraform apply
```

### Kubernetes / Flux
```bash
# Force Flux reconciliation
flux reconcile kustomization flux-system --with-source

# Check HelmRelease status
flux get helmreleases -A

# Check all Flux resources
flux get all -A
```

## Architecture

### Deployment Flow
```
git push → GitHub Actions (roles/ingress changes) → Ansible configures host
git push → Flux CD (gitops/ changes) → Reconciles Kubernetes resources every 10m
```

GitHub Actions runs Ansible only when `roles/**` or `ingress/**` change. Flux watches `gitops/**` directly from the Git repo.

### Directory Layout

- `terraform/` — OCI VM provisioning (VCN, subnet, security list, compute)
- `roles/` — 12 Ansible roles orchestrated by `site.yml`
- `gitops/` — All Kubernetes manifests, Flux-managed:
  - `gitops/kustomization.yaml` — Root kustomization (entry point for Flux)
  - `gitops/apps/` — Application deployments (each app is a subdirectory)
  - `gitops/observability/` — Prometheus, Grafana, Thanos, Alertmanager
  - `gitops/istio/` — Service mesh configuration
  - `gitops/cnpg/` — CloudNativePG operator
  - `gitops/databases/` — Database cluster definitions
  - `gitops/gateways/` — Istio Gateway + config
- `ingress/` — Traefik reverse proxy configs (Docker-based, on host)
- `group_vars/oracle_hosts/` — Ansible variables (`main.yml` + encrypted `vault.yml`)

### Application Pattern

Each app in `gitops/apps/{app-name}/` follows this structure:
```
kustomization.yaml          # Lists all resources for this app
namespace.yaml              # Namespace definition
{app}-helmrelease.yaml      # Flux HelmRelease (helm.toolkit.fluxcd.io/v2)
{app}-ocirepository.yaml    # OR {app}-helmrepo.yaml (chart source)
{app}-pvc.yaml              # Persistent volume claim (if needed)
{app}-httproute.yaml        # Gateway API HTTPRoute
```

All HelmReleases live in `flux-system` namespace with `targetNamespace` pointing to the app namespace. Standard settings: `interval: 10m`, `timeout: 5m`, 3 retries on install/upgrade.

### HTTPRoute Convention
Routes reference the shared Istio gateway:
```yaml
parentRefs:
  - name: gateway
    namespace: istio-system
    sectionName: http
hostnames:
  - "{app}.vps.kubespaces.cloud"
```

### Renovate Image Tag Pattern
Image tags use inline comments for Renovate auto-updates:
```yaml
tag: "2.7.5" # renovate: datasource=docker depName=n8nio/n8n
```

### Secrets
- Ansible secrets: encrypted with `ansible-vault` in `group_vars/oracle_hosts/vault.yml`
- Kubernetes database secrets: managed by CloudNativePG operator (referenced via `secretKeyRef`)

## Current Applications

n8n, Actual (budgeting), Dashy, Forgejo, Garage (S3), Minecraft, Rancher — all in `gitops/apps/`

Observability stack: Prometheus + Thanos + Grafana + Alertmanager in `gitops/observability/`
