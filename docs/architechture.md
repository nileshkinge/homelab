# 🏗️ Homelab Architecture

This diagram shows how the control node (WSL) interacts with VM and LXC hosts using Ansible and deploys Docker-based services.

---

## 📊 System Architecture

```mermaid
flowchart TB

    %% Control Node
    subgraph CONTROL["🧑‍💻 Control Node (WSL Ubuntu)"]
        A[Ansible CLI]
        B[Playbooks<br/>bootstrap.yml / deploy.yml]
        C[Inventory + group_vars]
        D[stacks.yml]
    end

    %% VM Host
    subgraph VM["🖥️ VM: debian-trixie-101"]
        VM1[Docker Engine]
        VM2[Docker Compose]
        VM3[Containers:<br/>Portainer<br/>Immich<br/>Nginx Proxy Manager]
    end

    %% LXC Host
    subgraph LXC["📦 LXC: debian-lxc-102"]
        LXC1[Docker Engine]
        LXC2[Docker Compose]
        LXC3[Containers:<br/>Filebrowser<br/>Media Server]
    end

    %% Connections
    A --> B
    B --> C
    B --> D

    B -- SSH --> VM
    B -- SSH --> LXC

    VM1 --> VM2 --> VM3
    LXC1 --> LXC2 --> LXC3
```

---

## 🔁 Deployment Flow

```mermaid
sequenceDiagram
    participant User
    participant WSL as Control Node (Ansible)
    participant VM as VM Host
    participant LXC as LXC Host

    User->>WSL: Run ansible-playbook deploy.yml
    WSL->>WSL: Load inventory + stacks.yml

    loop For each stack
        WSL->>WSL: Match host == inventory_hostname
        alt Stack belongs to VM
            WSL->>VM: docker compose up -d
        else Stack belongs to LXC
            WSL->>LXC: docker compose up -d
        end
    end

    VM-->>WSL: Status OK
    LXC-->>WSL: Status OK
```

---

## 🧠 Key Concepts

### 🔹 Control Node

* Runs inside WSL (Ubuntu)
* Executes Ansible playbooks
* Holds configuration and stack definitions

---

### 🔹 Inventory-Based Targeting

* Hosts grouped as:

  * `docker_vm`
  * `docker_lxc`
* Controls sudo vs non-sudo behavior

---

### 🔹 Stack Routing (`stacks.yml`)

* Defines where each service runs
* Enables multi-host deployment

---

### 🔹 Idempotent Execution

* Docker installed only if missing
* Services deployed safely on repeat runs

---

### 🔹 SSH-Based Execution

* No agents required
* Secure, key-based authentication

---

## 🚀 Future Architecture (Planned)

```mermaid
flowchart LR

    GitHub[(GitHub Repo)]
    Actions[GitHub Actions / CI]
    WSL[Control Node]

    GitHub --> Actions
    Actions --> WSL

    WSL --> VM
    WSL --> LXC
```

* GitOps workflow
* Auto-deploy on commit
* Drift detection

---
