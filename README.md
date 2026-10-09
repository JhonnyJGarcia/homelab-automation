# homelab-automation

Ansible playbooks I use to provision and manage the machines in my home lab.

## Playbooks

### `setup_urbackup.yml`

Playbook that prepares a **Raspberry Pi** as a backup host for an UrBackup server: refreshes the apt cache, installs **Docker** and docker-compose, creates the mount point for the external backup disk and enables the Docker service. Idempotent — safe to run again.

```bash
cp hosts.example.ini hosts.ini   # fill in host and user
ansible-playbook -i hosts.ini setup_urbackup.yml --ask-become-pass
```

The real inventory (`hosts.ini`) is gitignored; authentication is via SSH keys, so no credentials live in the repo.
