# 🏠 Setup Guide

This guide walks through setting up the homelab automation system from scratch.

---

## 🎯 Overview

This system uses:

* Ansible (automation)
* Docker + Docker Compose (services)
* SSH (secure remote execution)

---

## 🧰 Prerequisites

On your control machine:

```bash
sudo apt update && sudo apt install -y ansible git openssh-client
```

---

## 🔐 SSH Setup

Generate a key:

```bash
ssh-keygen -t ed25519
```

Copy to hosts:

```bash
ssh-copy-id <user>@<host-ip>
```

Verify:

```bash
ssh <user>@<host-ip>
```

---

## ⚙️ Configure Inventory

Edit:

```
ansible/inventory.ini
```

Example:

```ini
[docker_hosts]
host1 ansible_host=<ip> ansible_user=<user>
host2 ansible_host=<ip> ansible_user=<user>
```

---

## 🐳 Bootstrap Hosts

Install Docker (only if missing):

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml --ask-become-pass
```

---

## 📦 Define Stack Placement

Create:

```
config/hosts/<host>.yml
```

Example:

```yaml
stacks:
  my-app:
    path: stacks/my-app
```

---

## ▶️ Deploy Services

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml --ask-vault-pass
```

---

## 🔐 Secrets

Store sensitive values in:

```
ansible/host_vars/<host>/vault.yml
```

Encrypt:

```bash
ansible-vault encrypt ansible/host_vars/<host>/vault.yml
```

---

## 🧪 Test Connectivity

```bash
ansible all -i ansible/inventory.ini -m ping
```

---

## ⚠️ Troubleshooting

| Issue                 | Fix                            |
| --------------------- | ------------------------------ |
| SSH asks for password | Fix SSH keys                   |
| sudo not found        | disable become or install sudo |
| permission denied     | check user privileges          |

---

## ✅ Done

You now have a working multi-host Docker deployment system.

---
