# 🏠 Homelab Setup Guide (From Scratch)

This guide walks through setting up the homelab automation system from zero.

---

# 🎯 Overview

This setup uses:

* Ansible (automation)
* Docker + Docker Compose (services)
* WSL Ubuntu (control machine)
* SSH keys (secure access)

---

# 🖥️ Infrastructure

| Host Type | Example           | Notes     |
| --------- | ----------------- | --------- |
| VM        | debian-trixie-101 | Uses sudo |
| LXC       | debian-lxc-102    | No sudo   |

---

# 🧰 Step 1: Setup WSL (Control Machine)

Install dependencies:

```
sudo apt update && sudo apt upgrade -y
sudo apt install -y ansible openssh-client git
```

Verify:

```
ansible --version
```

---

# 🔐 Step 2: Generate SSH Key

```
ssh-keygen -t ed25519 -C "homelab"
```

---

# 🔑 Step 3: Copy SSH Key to Hosts

### VM

```
ssh-copy-id uandme77@192.168.0.210
```

### LXC (manual if needed)

```
cat ~/.ssh/id_ed25519.pub
```

Paste into:

```
~/.ssh/authorized_keys
```

Set permissions:

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

# ✅ Step 4: Verify SSH

```
ssh uandme77@192.168.0.210
ssh ansiblessh@192.168.0.201
```

---

# 📁 Step 5: Clone or Create Project

```
git clone <your-repo>
cd homelab
```

---

# ⚙️ Step 6: Configure Inventory

`ansible/inventory.ini`

```
[docker_vm]
debian-trixie-101 ansible_host=192.168.0.210 ansible_user=uandme77 ansible_become=true

[docker_lxc]
debian-lxc-102 ansible_host=192.168.0.201 ansible_user=ansiblessh ansible_become=false

[docker_hosts:children]
docker_vm
docker_lxc
```

---

# 🐳 Step 7: Bootstrap Hosts

```
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml -K
```

---

# 📦 Step 8: Define Services

`config/stacks.yml`

```
stacks:
  portainer:
    host: debian-trixie-101
    path: stacks/portainer

  filebrowser:
    host: debian-lxc-102
    path: stacks/filebrowser
```

---

# ▶️ Step 9: Deploy Services

```
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml -K
```

---

# 🔐 Step 10: Secrets Handling

Create:

```
group_vars/all/secrets.yml
```

Example:

```
ansible_become_password: yourpassword
```

---

## Option A: Ignore file

Add to `.gitignore`

---

## Option B: Encrypt

```
ansible-vault encrypt group_vars/all/secrets.yml
```

Edit:

```
ansible-vault edit group_vars/all/secrets.yml
```

---

# 🧪 Step 11: Test Connectivity

```
ansible all -i ansible/inventory.ini -m ping
```

---

# ⚠️ Troubleshooting

### sudo not found (LXC)

Set:

```
ansible_become=false
```

---

### Permission denied (usermod)

Use:

* root user OR
* install sudo

---

### SSH asking password

Fix SSH key setup

---

# 🧠 Design Notes

* VM uses sudo → full OS behavior
* LXC avoids sudo → lightweight
* stacks.yml controls service placement
* Compose used for simplicity

---

# 🚀 Next Steps

* Add GitOps (auto deploy)
* Add container state detection
* Add monitoring / alerts

---

# ✅ Done

You now have a working multi-host homelab automation system 🎉

---
