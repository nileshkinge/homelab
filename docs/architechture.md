# 🏗️ Architecture

---

## 📊 System Overview

```mermaid
flowchart TB

    subgraph CONTROL["Control Node"]
        A[Ansible CLI]
        B[Playbooks]
        C[Inventory]
        D[Config + Stacks]
    end

    subgraph HOSTS["Managed Hosts"]
        H1[Docker Engine]
        H2[Docker Compose]
        H3[Containers]
    end

    A --> B
    B --> C
    B --> D

    B -- SSH --> HOSTS

    H1 --> H2 --> H3
```

---

## 🔁 Deployment Flow

```mermaid
sequenceDiagram
    participant User
    participant Ansible
    participant Host

    User->>Ansible: Run deploy.yml
    Ansible->>Ansible: Load config + stacks

    loop For each stack
        Ansible->>Host: docker compose up -d
    end

    Host-->>Ansible: Status
```

---

## 🧠 Key Concepts

### Control Node

Runs Ansible and manages deployments.

---

### Hosts

Machines where containers run.

---

### Stack Mapping

Defined per host via config files.

---

### Idempotency

Only changes are applied.

---

### Secrets

Handled separately via Vault.

---

## 🚀 Future Direction

* GitOps automation
* Drift detection
* Monitoring integration

---
