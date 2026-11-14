# Incident Report: Proxmox Management Interface Outage

## 1. Incident Overview

**Incident:** Proxmox Web Management Interface Inaccessible

**Platform:** Proxmox VE

**Host:** HP ProDesk G6

**Management IP:** `192.168.137.102`

**Management Port:** `8006`

**Impact:** Unable to access the Proxmox web management interface from the Windows administration laptop.

**Resolution:** Physical restart of the Proxmox server.

## 2. Symptoms

After restarting Kubernetes VM112, access to the Proxmox management interface was lost.

Observed symptoms:

- Proxmox management IP remained `192.168.137.102`.
- Windows could successfully ping the Proxmox host.
- Existing virtual machines remained operational.
- TCP connectivity to port `8006` failed from Windows.
- The Proxmox web proxy service (`pveproxy`) was running.
- The service was listening on port `8006`.

## 3. Troubleshooting Performed

### 3.1 Verified Virtual Machine Status

Used the Proxmox physical console to inspect VM status:

```bash
qm list
```

All three existing Kubernetes virtual machines were running.

### 3.2 Verified Proxmox Web Proxy

Checked the service:

```bash
systemctl status pveproxy --no-pager
```

The service was active and running.

### 3.3 Verified Listening Port

```bash
ss -lntp | grep 8006
```

Confirmed that the Proxmox web proxy was listening on port `8006`.

### 3.4 Tested Local HTTPS Access

Executed directly on the Proxmox host:

```bash
curl -k -s -o /dev/null -w "%{http_code}\n" https://127.0.0.1:8006/
```

Result:

```text
200
```

This confirmed that the web interface was responding locally.

### 3.5 Tested Remote TCP Connectivity

From Windows PowerShell:

```powershell
Test-NetConnection 192.168.137.102 -Port 8006
```

The TCP connection failed.

### 3.6 Inspected Network Traffic

Used tcpdump on Proxmox:

```bash
tcpdump -ni vmbr0 -vv 'tcp port 8006'
```

Observed repeated TCP SYN packets from Windows targeting port `8006`.

No corresponding SYN-ACK response was observed in the reported packet output.

This indicated that connection requests were reaching the Proxmox bridge, but the TCP connection was not completing.

## 4. Resolution

A physical restart of the HP ProDesk G6 server was performed from the Proxmox console:

```bash
sync
reboot
```

After the restart:

- Proxmox booted successfully.
- All three existing virtual machines were still running.
- Access to the management interface was restored.

## 5. Root Cause and Findings

The exact underlying cause was not conclusively established.

The investigation narrowed the problem to the remote TCP connectivity path rather than an unresponsive Proxmox web application.

The physical restart restored access, but further investigation would be required to identify the specific cause.

## 6. Technical Lessons Learned

1. A running service does not necessarily mean remote clients can access it.
2. `systemctl status` verifies service state, not end-to-end connectivity.
3. `ss -lntp` helps confirm whether a service is listening on the expected port.
4. `curl` against localhost can distinguish local application availability from remote connectivity problems.
5. `Test-NetConnection` is useful for testing remote TCP port availability.
6. `tcpdump` helps identify whether connection requests reach a network interface.
7. Physical console access is an important recovery method when remote management becomes unavailable.
8. A physical server restart does not necessarily mean virtual machines or their data have been lost.

## 7. Follow-up Improvements

- Investigate host firewall and packet-filtering rules if the issue recurs.
- Review Proxmox boot and networking logs.
- Improve management network reliability.
- Document recovery procedures for future incidents.

## 8. Conclusion

This incident provided practical experience troubleshooting a Proxmox management-plane outage.

The investigation involved service verification, TCP port testing, packet inspection, and physical server recovery.

All existing virtual machines remained operational after recovery.
