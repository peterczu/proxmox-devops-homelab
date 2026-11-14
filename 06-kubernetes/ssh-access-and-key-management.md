# Kubernetes SSH Access and Key Management

## 1. Overview

SSH public-key authentication provides secure, passwordless access to the Kubernetes nodes.

This homelab uses two separate SSH identities:

- **Windows administration key:** Used by the administrator's Windows workstation.
- **Ansible control-node key:** Used by VM100 to connect to and manage Kubernetes nodes.

Both public keys are configured through Terraform and Proxmox Cloud-Init.

## 2. SSH Architecture

| Component | Role | SSH identity |
|---|---|---|
| Windows workstation | Infrastructure administration | `id_ed25519` |
| VM100 (`ubuntu-devops-01`) | Ansible control node | `ansible-control` |
| VM110 (`k8s-control-01`) | Kubernetes control plane | Accepts both keys |
| VM111 (`k8s-worker-01`) | Kubernetes worker 01 | Accepts both keys |
| VM112 (`k8s-worker-02`) | Kubernetes worker 02 | Accepts both keys |

## 3. SSH Key Locations

### Windows administration workstation

The Terraform project contains:

- `id_ed25519` — private key
- `id_ed25519.pub` — public key

The private key remains on the Windows workstation.

### Ansible control node

VM100 stores its SSH identity in:

- `/home/devops/.ssh/id_ed25519` — private key
- `/home/devops/.ssh/id_ed25519.pub` — public key

The Ansible private key authenticates against the managed Kubernetes nodes.

**Security:** Private keys must never be committed to Git repositories or shared. Only public keys should be distributed to servers.

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
```

When Terraform provisions Kubernetes VMs, Proxmox Cloud-Init receives both public keys for the `devops` account.

This makes SSH access part of the infrastructure provisioning configuration rather than requiring manual key installation for every new node.

## 5. Ansible Inventory Configuration

The Kubernetes nodes are grouped in the Ansible inventory:

```ini
[k8s_control_plane]
k8s-control-01 ansible_host=192.168.137.183 ansible_user=devops ansible_ssh_private_key_file=/home/devops/.ssh/id_ed25519

[k8s_workers]
k8s-worker-01 ansible_host=192.168.137.115 ansible_user=devops ansible_ssh_private_key_file=/home/devops/.ssh/id_ed25519
k8s-worker-02 ansible_host=192.168.137.212 ansible_user=devops ansible_ssh_private_key_file=/home/devops/.ssh/id_ed25519

[kubernetes:children]
k8s_control_plane
k8s_workers
```

The `[kubernetes:children]` declaration makes both child groups members of the parent `kubernetes` group.

This allows Ansible commands targeting `kubernetes` to reach all three nodes.

## 6. SSH Authentication Troubleshooting

An initial Ansible connectivity issue occurred because the Kubernetes nodes had the Windows workstation's public key, but Ansible was attempting to authenticate using VM100's different private key.

The two key pairs had different fingerprints.

The issue was resolved by adding VM100's public key to the `authorized_keys` file of all three Kubernetes nodes.

This preserved the existing Windows administration access while allowing Ansible to authenticate successfully.

### Important distinction

- `authorized_keys` is stored on the server and contains public keys permitted to log in.
- `known_hosts` is stored on the client and records the identity fingerprints of servers it has connected to.

These files serve different purposes and are not interchangeable.

### SSH authentication flow

1. Ansible runs on VM100.
2. Ansible uses `/home/devops/.ssh/id_ed25519` as its private key.
3. The Kubernetes node checks whether the matching public key is authorized.
4. If authentication succeeds, Ansible can execute tasks on the remote node.

## 7. Connectivity Verification

From VM100, run:

```bash
ansible kubernetes -i inventory.ini -m ping
```

Expected result:

```text
k8s-worker-01 | SUCCESS => {
    "ping": "pong"
}

k8s-control-01 | SUCCESS => {
    "ping": "pong"
}

k8s-worker-02 | SUCCESS => {
    "ping": "pong"
}
```

A successful Ansible ping confirms SSH connectivity and Python interpreter availability on the managed nodes.

The actual test succeeded on all three Kubernetes nodes.

## 8. Infrastructure as Code Verification

After adding both public keys to the Terraform configuration, the following commands were executed from the Windows Terraform project directory:

```powershell
terraform fmt
terraform validate
terraform plan
terraform apply
terraform plan
```

The final plan reported:

```text
No changes.
```

This confirmed that Terraform had converged with the updated configuration.

The update was applied in place without destroying or recreating the existing Kubernetes VMs.

**Important:** Updating Terraform's Cloud-Init configuration does not automatically guarantee that an already initialized guest will rerun Cloud-Init or modify its existing `authorized_keys` file. The existing Ansible public keys had already been installed manually on the three running nodes.

## 9. Lessons Learned

This exercise demonstrated:

1. SSH authentication depends on using the correct private/public key pair.
2. Multiple public keys can authorize access to the same Linux account.
3. Ansible requires its own private key to match a public key authorized on managed nodes.
4. Terraform can manage SSH access as part of infrastructure provisioning.
5. Cloud-Init can inject SSH public keys when provisioning virtual machines.
6. Cloud-Init configuration changes do not necessarily rerun inside existing guests.
7. `authorized_keys` and `known_hosts` have different security purposes.
8. Terraform plans help identify infrastructure changes before they are applied.
9. Private keys must be protected and excluded from version control.

## 10. Final State

All three Kubernetes nodes are reachable from the Ansible control node using SSH public-key authentication.

Both Windows administration and Ansible management access are supported.

Terraform configuration now declares both public keys for Kubernetes VM provisioning.

The homelab has a repeatable SSH access configuration that can be reused when provisioning additional Kubernetes nodes.