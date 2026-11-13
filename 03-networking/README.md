# Project 03: Networking and Connectivity Troubleshooting

## 1. Project Overview

### Objective

Build practical networking knowledge by configuring, inspecting and troubleshooting communication between Proxmox, Ubuntu virtual machines and external networks.

The project focuses on IP addressing, subnetting, gateways, routing, Linux bridges, TCP connections, DNS resolution, ARP and network troubleshooting.

The objective is to understand how systems communicate across a local network and how to identify connectivity failures systematically.

## 2. Network Architecture

The homelab initially uses Windows Internet Connection Sharing (ICS) to provide internet access to the Proxmox server and its virtual machines.

### Network Topology

```text
                 Internet
                    |
             Wi-Fi Router
                    |
             Windows Laptop
              Wi-Fi: 192.168.1.104
                    |
          Windows Internet Connection
                 Sharing
                    |
          Ethernet: 192.168.137.1
                    |
              Proxmox VE
             192.168.137.102
                    |
                 vmbr0
              Linux Bridge
                    |
          ---------------------
          |         |         |
        VM100     VM101     VM102
        .171      DHCP      .250
          |         |         |
       Ansible   Managed    Docker
       Control    Node       Host
        Node
```

## 3. IP Addressing and Subnetting

The homelab uses the private IPv4 network:

```text
192.168.137.0/24
```

This subnet provides 256 total addresses, with 254 traditionally available for hosts.

### Observed IP Configuration

| Component | IP Address | Purpose |
|---|---|---|
| Windows Ethernet | 192.168.137.1 | ICS gateway |
| Proxmox VE | 192.168.137.102 | Hypervisor management |
| VM100 | 192.168.137.171 | Ansible control node |
| VM101 | DHCP assigned | Managed Linux node |
| VM102 | 192.168.137.250 | Docker host |

VM addresses may change because DHCP is used for some machines.

### Network Inspection

Used the following commands to inspect network configuration:

```bash
ip -br addr
ip route
```

These commands help identify interface addresses and the system's routing table.

## 4. Default Gateway and Routing

The default gateway is the next-hop device used when a destination is outside the local subnet.

The homelab gateway is:

```text
192.168.137.1
```

Example routing command:

```bash
ip route
```

A typical route looks like:

```text
default via 192.168.137.1 dev eth0
192.168.137.0/24 dev eth0
```

### Important Networking Concept

VM100 and VM102 are on the same subnet. They can communicate directly over the local network without sending traffic through the default gateway.

Traffic destined for an external network must use the default route.

## 5. Proxmox Linux Bridge

Proxmox uses a Linux bridge called `vmbr0` to connect virtual machines to the physical network interface.

### Configuration

- Bridge: `vmbr0`
- Physical interface: `nic0`
- Proxmox management address: `192.168.137.102`
- Subnet: `192.168.137.0/24`

### Bridge Inspection

```bash
bridge link
```

This command displays network interfaces attached to Linux bridges.

Observed interfaces included:

- `nic0`: physical network interface.
- `tap100i0`: VM100 virtual network interface.
- `tap101i0`: VM101 virtual network interface.
- `tap102i0`: VM102 virtual network interface.

### Key Learning

A Linux bridge operates like a virtual Layer 2 switch. It forwards Ethernet frames between connected interfaces.

## 6. VM-to-VM Communication

Tested connectivity between virtual machines on the same subnet.

Example:

```bash
ping 192.168.137.250
```

Also used TCP connectivity tests to verify that services were reachable.

### TCP Connection Inspection

Used:

```bash
ss -tn
```

Observed established connections between VM100 and VM102, including:

```text
192.168.137.250:9090
192.168.137.171:54968
```

This demonstrated that VM100 was communicating with a service listening on VM102.

### Port Connectivity Testing

Used `nc` (netcat) to test TCP connections between machines.

Example:

```bash
nc -zv 192.168.137.250 9090
```

This helps distinguish basic IP reachability from application-port availability.

## 7. ARP and Neighbour Tables

Used:

```bash
ip neigh
```

The neighbour table displays IP-to-MAC address mappings used for communication on the local network.

### Key Learning

Before sending an Ethernet frame to another local IPv4 host, a machine needs the destination MAC address.

ARP helps resolve the IPv4 address to the corresponding MAC address.

Neighbour states such as `REACHABLE`, `STALE`, `DELAY` and `FAILED` provide useful troubleshooting information.

A neighbour entry alone does not prove that an application or the remote host is fully reachable.

## 8. DNS and External Connectivity

Used DNS and HTTP tests to investigate internet connectivity.

### DNS Testing

```bash
nslookup google.com 8.8.8.8
```

### HTTP Connectivity Testing

```bash
curl -I --connect-timeout 10 https://www.google.com
```

These tests helped distinguish DNS resolution problems from broader external network connectivity failures.

### Important Learning

A machine can have:
- A configured IP address.
- An active network interface.
- A default gateway.
- A valid local network connection.

Yet still be unable to reach the internet.

Network configuration must be tested at multiple layers.

## 9. Real-World Troubleshooting: Windows ICS Failure

### Problem

After restarting the physical Proxmox server, the virtual machines lost external internet connectivity.

Symptoms included:

- Proxmox could not access external websites.
- DNS queries timed out.
- GitHub operations failed because hostnames could not be resolved.
- Virtual machines had IP addresses but could not reliably reach external networks.

### Investigation

Inspected:

- Proxmox network interfaces.
- Default routing configuration.
- Linux bridge membership.
- ARP neighbour information.
- Windows network adapter configuration.
- Windows Internet Connection Sharing service.

Confirmed that the Windows `SharedAccess` service was running.

Restarting the service alone did not resolve the issue.

### Root Cause and Resolution

The Windows ICS sharing configuration needed to be refreshed.

Performed the following steps:

1. Opened Windows Network Connections using `ncpa.cpl`.
2. Opened the Wi-Fi adapter properties.
3. Navigated to the Sharing tab.
4. Disabled Internet Connection Sharing.
5. Waited approximately 10–15 seconds.
6. Re-enabled sharing.
7. Selected the Ethernet adapter as the private network connection.
8. Applied the configuration.

### Verification

Tested external connectivity from Proxmox:

```bash
curl -I --connect-timeout 10 https://www.google.com
```

The request returned:

```text
HTTP/2 200
```

Internet connectivity was restored, and Git operations worked again.

### Lessons Learned

- An active interface does not guarantee end-to-end connectivity.
- A working local network does not guarantee internet access.
- DNS failures can be symptoms of broader connectivity problems.
- Windows ICS can require its sharing configuration to be refreshed.
- Troubleshooting should proceed systematically, checking interfaces, routes, local connectivity, gateway reachability, DNS and external access.

## 10. Useful Networking Commands

| Command | Purpose |
|---|---|
| `ip -br addr` | Display interface addresses |
| `ip route` | Display routing table |
| `ip neigh` | Inspect neighbour/ARP entries |
| `bridge link` | Inspect Linux bridge interfaces |
| `ping` | Test ICMP connectivity |
| `ss -tn` | Inspect TCP connections |
| `ss -tulnp` | Inspect listening ports |
| `nc -zv` | Test TCP port connectivity |
| `nslookup` | Test DNS resolution |
| `curl -I` | Test HTTP response headers |

## 11. Future Improvements

Planned networking exercises:

- Static IP configuration.
- DHCP reservations.
- VLAN configuration.
- Linux firewall administration.
- Proxmox firewall.
- Network segmentation.
- Routing between VLANs.
- DNS server configuration.
- Dedicated router deployment.
- Kubernetes networking and CNI.

## 12. Technologies and Tools

- Proxmox VE
- Ubuntu Server
- Linux bridges
- IPv4
- TCP/IP
- ARP
- DNS
- Windows Internet Connection Sharing
- `iproute2`
- `bridge`
- `ss`
- `ping`
- `nc`
- `curl`
- `nslookup`

## 13. Project Status

**Status:** In progress

Practical networking configuration and troubleshooting have been completed in the homelab. Further work will expand into VLANs, network segmentation, firewall management and Kubernetes networking.
