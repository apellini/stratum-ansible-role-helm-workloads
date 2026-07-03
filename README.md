# stratum-ansible-role-helm-workloads

Ansible role that installs the Helm CLI at a pinned version and deploys STRATUM workload
charts from a data-driven list. Part of the STRATUM dev-environment bootstrap (Phase 2, D-INFRA-19).

Repo type: **public factory role** | Pattern: `apellini/stratum-ansible-role-*`
Released via **git tag** (`v0.1.0`, ...) -- never by branch.

---

## Requirements

- Target OS: Ubuntu 22.04 (Jammy) or 24.04 (Noble)
- Ansible controller: `ansible-core >= 2.16`
- Collections: `kubernetes.core` (install: `ansible-galaxy collection install kubernetes.core`)
- Target host prerequisites: k3s installed; Linkerd installed (run k3s and linkerd roles first)

---

## Role Variables

All variables are **required**. Declare them in the wrapper `group_vars/k3s.yml`.
The chart set lives there, not in this role (stateless/input-only pattern).

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `helm_version` | `str` | yes | Helm release tag. | `v3.15.2` |
| `helm_repositories` | `list[dict]` | yes | Helm repos to register. Each needs `name` and `url`. | see below |
| `helm_releases` | `list[dict]` | yes | Helm releases to deploy. Each needs `name`, `chart`, `chart_version`, `namespace`, optional `values`. | see below |

```yaml
helm_version: "v3.15.2"

helm_repositories:
  - name: qdrant
    url: https://qdrant.github.io/qdrant-helm

helm_releases:
  - name: qdrant
    chart: qdrant/qdrant
    chart_version: "0.9.0"
    namespace: stratum-data
    values:
      replicaCount: 1
```

---

## Tasks (in order)

1. Download the Helm archive from `get.helm.sh` (cached by version).
2. Extract the `helm` binary to `/usr/local/bin/` (idempotent -- skipped if already exists).
3. Register each repository in `helm_repositories` via `kubernetes.core.helm_repository`.
4. Deploy each release in `helm_releases` via `kubernetes.core.helm` with `wait: true`.

---

## Example Playbook

```yaml
- name: Deploy STRATUM Helm workloads on the k3s node
  hosts: k3s
  become: true
  roles:
    - role: stratum_factory.helm_workloads
```

---

## License

MIT
