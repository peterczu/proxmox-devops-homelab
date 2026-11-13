# Project 01: Proxmox Virtualization Infrastructure

## 1. Project Overview

### Objective

Build a local virtualization environment using Proxmox VE to provide an affordable, flexible infrastructure platform for learning Linux administration, networking, automation, containerization, Kubernetes and hybrid cloud technologies.

The homelab provides an environment where multiple virtual machines can communicate, host applications and simulate infrastructure commonly encountered in enterprise and cloud environments.

### Goals

- Install and configure Proxmox VE on physical hardware.
- Create and manage multiple Ubuntu Server virtual machines.
- Establish network connectivity between the hypervisor and virtual machines.
- Configure virtual networking using Linux bridges.
- Allocate hardware resources to virtual machines.
- Build a foundation for future DevOps and SRE projects.

## 2. Physical Infrastructure

### Host Hardware

| Component | Specification |
|---|---|
| Manufacturer | HP |
| Model | HP ProDesk G6 |
| Processor | 12 logical CPUs |
| Memory | 32 GB RAM |
| Storage | NVMe SSD |
| Hypervisor | Proxmox VE |

The physical server acts as the central virtualization host.

Proxmox VE manages the physical hardware and provides the platform for creating, running and administering virtual machines.

*Note: Exact CPU model, NVMe capacity and Proxmox VE version can be added later.*

## 3. Virtualization Platform

### Proxmox VE

Proxmox VE was installed directly on the physical server as a bare-metal hypervisor.

The Proxmox management interface is accessible through:

`https://192.168.137.102:8006`

The hypervisor provides:

- Virtual machine creation and management.
- CPU and memory allocation.
- Virtual networking.
- VM console access.
- VM startup and shutdown management.
- Infrastructure administration through the web interface and command line.

## 4. Virtual Machine Architecture

Three Ubuntu Server virtual machines were created to establish a small multi-node infrastructure environment.

| VM ID | Hostname | Role |
|---|---|---|
| 100 | ubuntu-devops-01 | Ansible control node |
| 101 | ubuntu-devops-02 | Managed Linux node |
| 102 | ubuntu-devops-03 | Docker application host |

### VM responsibilities

**VM 100 — Ansible Control Node**

Acts as the central automation machine.

Responsibilities:
- Managing Ansible inventory.
- Establishing SSH connections to managed nodes.
- Executing remote administration commands.
- Automating infrastructure and Docker operations.

**VM 101 — Managed Linux Node**

Provides an additional Linux environment for administration and automation exercises.

Responsibilities:
- Receiving remote Ansible commands.
- Linux administration practice.
- Testing connectivity and remote management.

**VM 102 — Docker Application Host**

Hosts the containerized application environment.

Responsibilities:
- Running Docker Engine.
- Running Docker Compose applications.
- Hosting Nginx, FastAPI and PostgreSQL containers.
- Providing an environment for container administration and monitoring.

## 5. Network Architecture

### Network Overview

The homelab uses a private IPv4 network:

`192.168.137.0/24`

Internet connectivity is currently provided through Windows Internet Connection Sharing (ICS).

The Windows laptop connects to the internet over Wi-Fi and shares that connection through its Ethernet adapter, which connects to the Proxmox server.

### Network Configuration

| Component | IP address | Purpose |
|---|---|---|
| Windows Ethernet adapter | 192.168.137.1 | Internet sharing gateway |
| Proxmox host | 192.168.137.102 | Hypervisor management |
| VM 100 | 192.168.137.171 | Ansible control node |
| VM 102 | 192.168.137.250 | Docker application host |
| VM 101 | DHCP assigned | Managed Linux node |

*VM addresses are observed addresses and may change because DHCP is currently used.*

### Proxmox Linux Bridge

The main Proxmox virtual bridge is:

`vmbr0`

The physical network interface `nic0` is attached to this bridge.

Virtual machine network interfaces are also connected to `vmbr0`, allowing the VMs to communicate with the physical network.

The bridge configuration allows virtual machines to participate in the same local network as the Proxmox host.

### Network Topology

```text
                 INTERNET
                     |
               Wi-Fi Router
                     |
              Windows Laptop
               192.168.1.x
                     |
              Windows ICS
               192.168.137.1
                     |
                 Ethernet
                     |
              Proxmox Server
               192.168.137.102
                     |
                  vmbr0
                     |
          +----------+----------+
          |          |          |
        VM 100     VM 101     VM 102
        Ansible    Managed    Docker
        Control    Node       Host
```

### Network Concepts Practised

During infrastructure setup and troubleshooting, the following networking concepts were explored:

- IPv4 addressing and subnet masks.
- Default gateways and routing tables.
- Linux bridge networking.
- Virtual machine network connectivity.
- MAC addresses and ARP neighbour discovery.
- TCP connections and source/destination ports.
- Network troubleshooting using `ping`, `ip`, `ss`, `curl` and `nc`.
- Internet Connection Sharing and DNS resolution.

## 6. Implementation and Validation

The following activities were completed during the infrastructure setup:

1. Installed Proxmox VE on the HP ProDesk G6.
2. Configured the Proxmox management network.
3. Established the Linux bridge `vmbr0`.
4. Created three Ubuntu Server virtual machines.
5. Assigned each VM a specific role in the homelab.
6. Established SSH access to the virtual machines.
7. Tested connectivity between Windows, Proxmox and the VMs.
8. Verified virtual network interfaces and bridge membership.

### Useful Proxmox Commands

Check network interfaces:

```bash
ip -br addr
```

Display routing information:

```bash
ip route
```

Inspect bridge membership:

```bash
bridge link
```

Inspect neighbour entries:

```bash
ip neigh
```

List virtual machines:

```bash
qm list
```

Check VM status:

```bash
qm status 100
```

## 7. Troubleshooting Experience

### Issue: Internet connectivity lost from Proxmox and virtual machines

**Symptoms**

- GitHub operations failed with DNS resolution errors.
- Package downloads from Linux virtual machines failed.
- Proxmox could not resolve external domain names.
- Direct HTTPS connections to external IP addresses timed out.

**Investigation**

The following areas were checked:

- Proxmox network interfaces.
- Linux bridge status.
- Gateway configuration.
- ARP neighbour entries.
- Windows Ethernet and Wi-Fi configuration.
- Windows routing table.
- Windows Internet Connection Sharing service.

**Root cause and resolution**

The Windows Internet Connection Sharing configuration was suspected of being stuck.

Disabling and re-enabling Internet Connection Sharing on the Windows Wi-Fi adapter, with Ethernet selected as the shared connection, restored connectivity.

The Proxmox host subsequently established successful HTTPS connections and GitHub pushes worked again.

**Lesson learned**

A running network interface and a valid IP configuration do not guarantee internet connectivity.

Network troubleshooting requires distinguishing between local connectivity, routing, DNS resolution and external service connectivity.

## 8. Lessons Learned

This project established practical understanding of:

- Bare-metal virtualization.
- Hypervisor administration.
- Virtual machine resource allocation.
- Linux bridge networking.
- Private IPv4 networks.
- DHCP and default gateways.
- Communication between physical and virtual machines.
- Basic network troubleshooting.

It also established the foundation for managing infrastructure through automation rather than manually configuring every server.

## 9. Future Improvements

The following improvements are planned:

- Introduce a dedicated router for homelab networking.
- Implement VLAN segmentation.
- Configure DHCP reservations for virtual machines.
- Document static IP and network allocation.
- Build a multi-node Kubernetes cluster.
- Introduce centralized monitoring.
- Explore hybrid networking between the homelab and AWS.

## 10. Project Status

**Status: Foundation established — ongoing development.**

The Proxmox virtualization environment is operational, with three Ubuntu Server virtual machines available for subsequent Linux, networking, automation and containerization projects.

The infrastructure will continue evolving as additional DevOps capabilities are implemented.

---

**Technologies:** Proxmox VE, Ubuntu Server, Linux, KVM/QEMU, Linux Bridge, IPv4, SSH, Windows Internet Connection Sharing.

**Repository:** [proxmox-devops-homelab](https://github.com/peterczu/proxmox-devops-homelab)
