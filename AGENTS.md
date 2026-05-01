# Repository Guidelines

## Project Structure & Module Organization

CloudLab manages an OCI host with Terraform, Ansible, and Flux GitOps. Root files include `site.yml`, `ansible.cfg`, `inventory.ini.example`, and `deploy.sh`.

- `terraform/`: OCI VM, network, variables, outputs, and `terraform.tfvars.example`.
- `roles/`: Ansible roles for host services such as `common`, `packages`, `k0s`, `flux`, `borg`, `tailscale`, and `datadog`.
- `group_vars/oracle_hosts/`: shared Ansible variables; keep secrets in encrypted `vault.yml`.
- `gitops/`: Flux-managed Kubernetes manifests. `gitops/kustomization.yaml` is the root entry point; apps live under `gitops/apps/{app}/`.
- `charts/`: local Helm charts and values.
- `ingress/`: host-level Traefik and Gateway API routing configuration.

## Build, Test, and Development Commands

- `terraform -chdir=terraform init`: initialize Terraform providers.
- `terraform -chdir=terraform plan`: preview OCI infrastructure changes.
- `terraform -chdir=terraform apply`: update OCI resources.
- `ansible oracle_hosts -m ping`: verify Ansible connectivity.
- `ansible-playbook site.yml --check --diff`: dry-run host configuration changes.
- `ansible-playbook site.yml --tags k0s,flux`: run selected role tags.
- `ansible-playbook site.yml`: apply full host configuration.
- `flux reconcile kustomization flux-system --with-source`: force Flux reconciliation.
- `kubectl get pods -A` and `flux get helmreleases -A`: inspect cluster state.

## Coding Style & Naming Conventions

Use YAML with two-space indentation. Keep Ansible role tasks in `roles/{role}/tasks/main.yml`, defaults in `defaults/main.yml`, handlers in `handlers/main.yml`, and templates as `.j2`. Prefer role-scoped variables, such as `borg_repository`.

For GitOps apps, use `gitops/apps/{app-name}/`, a singular `namespace.yaml`, `{app}-helmrelease.yaml`, optional `{app}-httproute.yaml`, and `README.md`. HelmReleases should live in `flux-system` and set `targetNamespace` to the app namespace. Disable chart ingress and use HTTPRoute instead.

## Testing Guidelines

There is no unit test suite. Validate changes with dry runs and native tooling before applying. Run `terraform fmt -check` and `terraform validate` in `terraform/` for Terraform edits. For Kubernetes manifests, use `kubectl apply --dry-run=server -f <file>` when CRDs are available. For Helm charts, run `helm lint charts/<chart>`.

## Commit & Pull Request Guidelines

Recent commits use short imperative subjects and conventional prefixes, especially `fix(scope): ...` or `fix: ...`. Prefer examples like `fix(omni): remove invalid sysctl` or `feat(flux): add receiver route`.

Pull requests should describe the affected layer (`terraform`, `roles`, `gitops`, `charts`, or `ingress`), list validation commands run, link related issues, and include screenshots or `kubectl`/`flux` output for user-facing app changes.

## Security & Configuration Tips

Do not commit real secrets, kubeconfigs, Terraform state, or private inventory values. Use `ansible-vault edit group_vars/oracle_hosts/vault.yml` for secrets and keep `terraform.tfvars` local. Start from `inventory.ini.example` and `terraform/terraform.tfvars.example`.
