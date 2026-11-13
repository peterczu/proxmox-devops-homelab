# Project 04: Ansible Automation

## 1. Project Overview

### Objective

Build practical infrastructure automation skills using Ansible to manage and administer multiple Ubuntu Server virtual machines in a Proxmox homelab.

The project focuses on agentless automation, SSH connectivity, inventory management, ad-hoc commands, privilege escalation and remote application management.

The objective is to understand how infrastructure engineers automate repetitive administrative tasks across multiple servers.

## 2. Lab Architecture

The Ansible environment consists of one control node and managed Linux servers.

| VM | Hostname | Role |
|---|---|---|
| VM100 | ubuntu-devops-01 | Ansible control node |
| VM101 | ubuntu-devops-02 | Managed Linux node |
| VM102 | ubuntu-devops-03 | Docker host and managed node |

### Architecture

```text
             Ansible Control Node
              ubuntu-devops-01
                VM100
                   |
                 SSH
                   |
          ---------------------
          |                   |
     Managed Node        Docker Host
     ubuntu-devops-02    ubuntu-devops-03
         VM101                VM102
          |                    |
     Linux Server        Docker Services
```

Ansible connects to managed nodes over SSH and executes tasks remotely.

## 3. Ansible Control Node

VM100 is used as the Ansible control node.

The Ansible project directory is:

```text
/home/devops/ansible-lab
```

The control node contains:
- Ansible installation.
- Inventory configuration.
- SSH access to managed nodes.
- Automation commands and project files.

### Verify Ansible Installation

```bash
ansible --version
```

## 4. Inventory Management

Ansible inventory defines the servers that Ansible manages.

The inventory file is:

```text
inventory.ini
```

Example inventory structure:

```ini
[webservers]
192.168.137.250
```

The `webservers` group identifies the managed servers targeted by automation commands.

Inventory allows administrators to manage individual machines or groups of machines without repeatedly specifying connection details.

### Inventory Verification

```bash
ansible-inventory -i inventory.ini --list
```

## 5. SSH-Based Remote Management

Ansible uses SSH to communicate with Linux managed nodes.

Unlike agent-based management systems, Ansible does not require a permanent Ansible agent installed on each managed Linux server.

The control node initiates SSH connections and executes the requested modules remotely.

### Test Connectivity

```bash
ansible webservers -i inventory.ini -m ping
```

The Ansible `ping` module verifies that Ansible can connect to the managed host and execute Python-based module operations.

A successful result typically returns:

```text
SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 6. Ad-Hoc Commands

Ad-hoc commands allow administrators to execute individual tasks without creating a complete playbook.

Example:

```bash
ansible webservers -i inventory.ini -m command -a "hostname"
```

This executes the hostname command remotely on all hosts in the `webservers` group.

### Check Disk Usage

```bash
ansible webservers -i inventory.ini -m command -a "df -h"
```

### Check System Uptime

```bash
ansible webservers -i inventory.ini -m command -a "uptime"
```

Ad-hoc commands are useful for quick operational checks and repetitive administrative tasks.

## 7. Privilege Escalation

Some Linux administration tasks require elevated privileges.

Ansible supports privilege escalation using the `-b` option, which enables become functionality.

Example:

```bash
ansible webservers -i inventory.ini -b -m command -a "systemctl status docker"
```

The `-b` option allows Ansible to execute the task with elevated privileges, typically through sudo.

This avoids manually logging into every server to perform administrative operations.

## 8. Command Module vs Shell Module

An important practical lesson was understanding the difference between Ansible's `command` and `shell` modules.

### Command Module

The command module executes commands directly without invoking a shell.

Example:

```bash
ansible webservers -i inventory.ini -m command -a "hostname"
```

It does not interpret shell operators such as:

- Pipes (`|`)
- Redirection (`>`)
- Command chaining (`&&`)
- Shell variable expansion

### Shell Module

The shell module executes commands through a shell and supports shell features.

Example:

```bash
ansible webservers -i inventory.ini -m shell -a "df -h | grep /dev"
```

### Key Learning

Use the `command` module when shell functionality is unnecessary.

Use the `shell` module when pipes, redirection, command chaining or other shell features are required.

The command module is generally preferable when it can accomplish the task.

## 9. Remote Docker Compose Management

Ansible was used to manage Docker Compose services running on VM102.

The Docker Compose project directory on the managed host is:

```text
/opt/docker-compose-lab
```

Example command executed from VM100:

```bash
ansible webservers -i inventory.ini -b -m command -a "docker compose -f /opt/docker-compose-lab/compose.yaml up -d"
```

This remotely executes Docker Compose on the managed server.

### Operational Benefits

- Manage services without manually logging into the Docker host.
- Execute commands remotely.
- Reduce repetitive administrative work.
- Use a central control node to manage infrastructure.

## 10. Ansible Task Results

Ansible reports the result of each operation.

Common result indicators include:

| Result | Meaning |
|---|---|
| SUCCESS | Task completed successfully |
| CHANGED | Ansible reports a change was made |
| FAILED | Task execution failed |
| UNREACHABLE | Ansible could not connect to the host |

Example output:

```text
ubuntu-devops-03 | CHANGED | rc=0 >>
```

This confirms that the task executed successfully and Ansible reported a change.

A successful result does not always mean the application itself is healthy, so application-level verification remains important.

## 11. Practical Learning Outcomes

Through this project, I have gained practical experience with:

- Ansible control-node architecture.
- Managing Linux hosts through SSH.
- Inventory configuration.
- Ad-hoc automation.
- Remote command execution.
- Privilege escalation.
- Understanding Ansible modules.
- Managing Docker Compose remotely.
- Reading and interpreting Ansible execution results.

## 12. Future Improvements

Planned areas for further Ansible practice:

- YAML fundamentals.
- Ansible playbooks.
- Tasks and handlers.
- Variables and variable precedence.
- Templates using Jinja2.
- File and directory management.
- Package installation automation.
- Service management.
- Roles and reusable automation.
- Ansible Vault for secrets.
- Dynamic inventories.
- Infrastructure provisioning with Terraform and Ansible.

## 13. Technologies and Tools

- Ansible
- Ubuntu Server
- OpenSSH
- YAML
- INI inventory
- Python
- Docker
- Docker Compose
- Proxmox VE
- Git and GitHub

## 14. Project Status

**Status:** In progress

The initial Ansible control-node and managed-node environment is operational. SSH-based automation, ad-hoc commands, privilege escalation and remote Docker Compose management have been practised.

Further development will focus on reusable playbooks, roles, templates and configuration management.
