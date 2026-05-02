# 🏠 Homelab Docker Stacks (GitOps with Ansible)

A **general-purpose GitOps framework** for deploying Docker Compose stacks across multiple hosts using Ansible.

---

## ✨ Features

* Idempotent Docker setup (install only if missing)
* Multi-host support (VMs, LXCs, bare metal)
* Host-specific stack placement
* Change-aware deployments
* GitOps-style workflow
* Secure secret handling via Ansible Vault

---

## 📁 Project Structure

```
homelab/
├── ansible/
│   ├── inventory.ini
│   ├── host_vars/
│   │   ├── <host>/
│   │   │   └── vault.yml
│   │
│   ├── playbooks/
│   │   ├── bootstrap.yml
│   │   └── deploy.yml
│   │
│   └── roles/
│       └── docker/
│
├── config/
│   └── hosts/
│       ├── <host>.yml
│
├── group_vars/
│   └── all/
│       └── vars.yml
│
├── stacks/
│   ├── <stack-name>/
│   │   └── docker-compose.yml
│
├── docs/
│   ├── setup.md
│   └── architecture.md
│
└── README.md
```

---

## 🧠 How It Works

* `inventory.ini` defines your hosts
* `config/hosts/<host>.yml` defines which stacks run on each host
* `stacks/` contains Docker Compose definitions
* Ansible:

  * connects via SSH
  * ensures Docker is installed
  * deploys only changed stacks

---

## 🚀 Usage

### Bootstrap hosts

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/bootstrap.yml --ask-vault-pass --ask-become-pass
```

---

### Deploy stacks

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml --ask-vault-pass
```

---

### Deploy a single host

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy.yml --limit <host> --ask-vault-pass
```

---

## 🔐 Secrets & Vault

Sensitive data is stored in:

```
ansible/host_vars/<host>/vault.yml
```

Used for:

* credentials
* API keys
* environment variables

Encrypt:

```bash
ansible-vault encrypt ansible/host_vars/<host>/vault.yml
```

---

## ⚠️ Design Principles

* Infrastructure is defined in config files
* Secrets are stored separately
* No environment-specific values in repo
* Idempotent deployments only

---

## 📚 Documentation

* Setup guide → `docs/setup.md`
* Architecture → `docs/architecture.md`

---

## 🚀 Goal

A reusable, scalable GitOps framework for managing containerized services across multiple hosts.

---
