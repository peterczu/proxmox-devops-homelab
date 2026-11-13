# Project 02: Linux Administration

## 1. Project Overview

### Objective

Develop practical Linux system administration skills using Ubuntu Server virtual machines running on Proxmox VE.

This project focuses on managing Linux systems through the command line, administering services, understanding system resources, managing network connectivity and preparing Linux servers for automation and application hosting.

## 2. Lab Environment

The Linux administration environment consists of Ubuntu Server virtual machines hosted on Proxmox VE.

| VM | Hostname | Role |
|---|---|---|
| VM100 | ubuntu-devops-01 | Ansible control node |
| VM101 | ubuntu-devops-02 | Managed Linux node |
| VM102 | ubuntu-devops-03 | Docker host |

The machines communicate over the homelab network.

## 3. Linux Administration Skills

### 3.1 Command-Line Administration

Practised navigating and managing the Linux filesystem using commands such as:

```bash
pwd
ls -la
cd
mkdir
touch
cp
mv
rm
cat
head
tail
grep
find
```

These commands support everyday server administration, file inspection and troubleshooting.

### 3.2 Remote Administration with SSH

Used SSH to connect remotely to Ubuntu Server virtual machines.

Example:

```bash
ssh devops@192.168.137.171
```

Practised:
- Establishing remote server connections.
- Working with SSH keys.
- Managing multiple Linux machines remotely.
- Running administrative commands without a graphical interface.

### 3.3 Package Management

Used Ubuntu's APT package management system to install and maintain software.

Common commands:

```bash
sudo apt update
sudo apt upgrade
sudo apt install curl git
sudo apt remove package-name
```

Also verified installed tools and their versions.

### 3.4 System Services

Used systemd to inspect and manage Linux services.

Example commands:

```bash
systemctl status ssh
systemctl is-active qemu-guest-agent
systemctl list-units --type=service
```

Learned how Linux services operate in the background and how to inspect their current state.

### 3.5 Process and Resource Inspection

Used Linux utilities to inspect running processes and system resources.

Examples:

```bash
ps aux
top
free -h
df -h
lsblk
```

These tools help identify resource constraints and investigate server performance issues.

### 3.6 Network Administration

Practised inspecting network interfaces, IP addresses, routing tables and connectivity.

Commands:

```bash
ip -br addr
ip route
ip neigh
ping
ss -tulnp
curl
```

Applied these tools when troubleshooting connectivity between virtual machines and external networks.

## 4. Linux in Infrastructure Operations

Ubuntu Server provides the operating system foundation for the other projects in this homelab.

Linux administration supports:

- Managing the Ansible control node and managed hosts.
- Hosting Docker containers and application services.
- Inspecting application logs and service health.
- Troubleshooting networking and connectivity.
- Preparing infrastructure for Kubernetes.
- Supporting hybrid cloud infrastructure.

## 5. Practical Learning Outcomes

Through this project, I have developed practical experience with:

- Linux command-line operations.
- Remote server administration through SSH.
- Package management with APT.
- Inspecting and managing system services.
- Inspecting processes and system resources.
- Network interface and routing diagnostics.
- Working with Linux servers in a virtualized environment.

## 6. Future Improvements

Planned areas for further Linux administration practice:

- Linux users, groups and permissions.
- File ownership and access control.
- Disk partitioning and filesystem management.
- LVM and storage administration.
- System logs and journalctl.
- Cron jobs and scheduled tasks.
- Firewall configuration with UFW.
- SSH hardening and server security.
- Bash scripting and automation.
- RHCSA-aligned administration exercises.

## 7. Technologies and Tools

- Ubuntu Server
- Proxmox VE
- OpenSSH
- APT
- systemd
- Bash
- Linux networking utilities
- Git and GitHub

## 8. Project Status

**Status:** In progress

This project forms the Linux administration foundation of the Proxmox DevOps homelab. Additional exercises and practical evidence will be added as learning progresses.
