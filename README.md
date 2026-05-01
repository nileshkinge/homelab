# Homelab Docker Stacks

GitOps-style source of truth for homelab Docker Compose stacks.

## Layout

```text
ansible/
  inventory.ini
  host_vars/
    debian-trixie-101/
      vault.yml
    debian-lxc-102/
      vault.yml
  playbooks/
    bootstrap.yml
    deploy.yml
  roles/docker/
config/
  hosts/
    debian-trixie-101.yml
    debian-lxc-102.yml
group_vars/all/
  vars.yml
stacks/
  <stack-name>/docker-compose.yaml
```

## Hosts

- `debian-trixie-101` at `192.168.0.210`, user `uandme77`, uses sudo
- `debian-lxc-102` at `192.168.0.201`, user `ansiblessh`, no sudo

The inventory only defines hostnames and groups. Connection details live in `ansible/host_vars/<inventory-hostname>/vault.yml`.

## Playbooks

`bootstrap.yml` prepares Docker hosts:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml
```

If a host uses sudo and prompts for a password, add `--ask-become-pass`:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml --ask-vault-pass --ask-become-pass
```

`deploy.yml` checks out this repo on each host and deploys the Compose stacks assigned to that host:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml
```

## Stack Placement

Each host has its own stack file under `config/hosts/`. The file name must match the Ansible inventory hostname:

```yaml
# config/hosts/debian-trixie-101.yml
stacks:
  portainer:
    path: stacks/portainer
```

The deploy playbook only reads `config/hosts/{{ inventory_hostname }}.yml` for each host. It calculates a hash for each Compose stack, skips unchanged stacks, and deploys when a stack changed or has no existing Compose containers.

To intentionally deploy nothing on a host, use an empty stack map:

```yaml
stacks: {}
```

To test one host only, limit the playbook run:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml --ask-vault-pass --limit debian-trixie-101
```

## GitOps Settings

Set the real repository URL in `group_vars/all/vars.yml`:

```yaml
homelab_repo_url: "https://github.com/YOUR_USER/homelab.git"
homelab_repo_version: main
```

Each host stores deployment state at `~/.local/state/homelab/state.json`.

## Secrets

Do not commit plaintext `.env` files, SSH keys, API tokens, or vault password files.

Encrypted Ansible Vault files may be committed when the vault password is stored separately.

## Ansible Vault

Ansible automatically loads variables from `host_vars/<inventory-hostname>/` next to the inventory file. For example, when it sees `debian-trixie-101` in `ansible/inventory.ini`, it loads:

```text
ansible/host_vars/debian-trixie-101/vault.yml
```

That file can hold connection variables:

```yaml
ansible_host: 192.168.0.210
ansible_user: uandme77
ansible_become: true
```

If sudo requires a password, either pass it interactively with `--ask-become-pass` or store it encrypted in the host vault file:

```yaml
ansible_become_password: "your-sudo-password"
```

Encrypt the host variable files before publishing this repo publicly:

```bash
ansible-vault encrypt ansible/host_vars/debian-trixie-101/vault.yml
ansible-vault encrypt ansible/host_vars/debian-lxc-102/vault.yml
```

Run playbooks with Vault:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml --ask-vault-pass
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml --ask-vault-pass
```

Edit encrypted files with:

```bash
ansible-vault edit ansible/host_vars/debian-trixie-101/vault.yml
```

Useful Vault commands:

```bash
# View an encrypted file without editing it
ansible-vault view ansible/host_vars/debian-trixie-101/vault.yml

# Edit an encrypted file and save it encrypted again
ansible-vault edit ansible/host_vars/debian-trixie-101/vault.yml

# Change the vault password for a file
ansible-vault rekey ansible/host_vars/debian-trixie-101/vault.yml

# Temporarily decrypt a file
ansible-vault decrypt ansible/host_vars/debian-trixie-101/vault.yml

# Encrypt a plaintext file
ansible-vault encrypt ansible/host_vars/debian-trixie-101/vault.yml
```
