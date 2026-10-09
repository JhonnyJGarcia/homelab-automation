# homelab-automation

Ansible playbooks I use to provision and manage the machines in my home lab.

## `raspberry.yml` — Raspberry Pi network node

Rebuilds my **Raspberry Pi 4** (Debian 13, arm64) from a clean OS to the services it runs 24/7. Two roles:

| Role | What it does |
|---|---|
| `docker` | Installs **Docker Engine** and the **Compose plugin** from Docker's official apt repository, removing the conflicting distro packages first |
| `compose_stacks` | Deploys each service as a **Docker Compose** stack under `~/docker/<name>/`, renders its config files and brings it up |

Stacks:

- **AdGuard Home** — network-wide DNS filtering and DHCP (host networking)
- **RustDesk server** (`hbbs` + `hbbr`) — self-hosted remote support
- **cAdvisor** — container metrics for **Prometheus**, bound to localhost only
- **prometheus-pve-exporter** — **Proxmox VE** metrics via a read-only API token

### Design choices

- **Templates, not copies:** compose files are Jinja2 templates; image versions and host-specific values live in `group_vars/raspberry/vars.yml`
- **Secrets stay out of Git:** the Proxmox API token lives in an **ansible-vault** encrypted `vault.yml` (gitignored); its config file is deployed with mode `0600` and `no_log`
- **Least exposure:** metrics exporters listen on `127.0.0.1` only — Prometheus runs on the same host
- **Idempotent:** safe to re-run; only changed stacks are restarted

### Usage

```bash
cp hosts.example.ini hosts.ini                                   # host and user
cp group_vars/raspberry/vault.yml.example group_vars/raspberry/vault.yml
ansible-vault encrypt group_vars/raspberry/vault.yml             # fill in, then encrypt
ansible-galaxy collection install community.docker

ansible-playbook raspberry.yml --ask-become-pass --ask-vault-pass --check --diff   # dry run
ansible-playbook raspberry.yml --ask-become-pass --ask-vault-pass
```

### Layout

```
raspberry.yml
group_vars/raspberry/
  vars.yml                 # versions, stack list, non-secret settings
  vault.yml.example        # template for the encrypted secrets file
roles/
  docker/tasks/main.yml
  compose_stacks/
    tasks/main.yml
    templates/<stack>/compose.yml.j2
```
