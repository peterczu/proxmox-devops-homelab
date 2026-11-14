# Kubernetes SSH Access and Key Management

## 1. Overview

SSH public-key authentication provides secure, passwordless access to the Kubernetes nodes.

This homelab uses two separate SSH identities:

- Windows administration key: used by the administrator's Windows workstation.
- Ansible control-node key: used by VM100 to connect to and manage Kubernetes nodes.

Both public keys are configured through Terraform and Proxmox Cloud-Init.

## 2. SSH Architecture

| Component | Role | SSH identity |
|---|---|---|
| Windows workstation | Infrastructure administration | id_ed25519 |
| VM100 (Ansible control node) | Configuration management | ansible-control |
| VM110 (Kubernetes control plane) | Kubernetes control plane | Accepts both keys |
| VM111 (Kubernetes worker 01) | Kubernetes worker | Accepts both keys |
| VM112 (Kubernetes worker 02) | Kubernetes worker | Accepts both keys |

## 3. Key Locations

### Windows administration workstation

The Terraform project contains:

- `id_ed25519` — private key
- `id_ed25519.pub` — public key

The private key must remain on the Windows workstation.

### Ansible control node

VM100 stores its SSH identity in:

- `/home/devops/.ssh/id_ed25519` — private key
- `/home/devops/.ssh/id_ed25519.pub` — public key

The Ansible private key is used to authenticate against managed Kubernetes nodes.

**Security:** Private keys must never be copied into Git repositories or shared. Only public keys should be distributed to servers.

## 4. Terraform and Cloud-Init Configuration

The Terraform project stores the Ansible public key in:

`ansible-control.pub`

The Kubernetes VM resource declares both public keys in its Cloud-Init user account configuration:

```hcl
user_account {
  username = "devops"

  keys = [
    trimspace(file("${path.module}/id_ed25519.pub")),
    trimspace(file("${path.module}/ansible-control.pub"))
  ]
}
