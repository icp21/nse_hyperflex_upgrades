# Cluster Analysis Report: KOH-VMW-MGMT-Cluster

**Cluster Type:** DEFAULT_CLUSTER  
**vCenter:** koh-vmw-vcenter01.nseroot.com

## ESXi Hosts, SCVM IPs, SCVM Data IPs, vMotion IPs
| ESXi Host IP      | SCVM IP         | SCVM Data IP | Node Role | vMotion IP   |
|-------------------|-----------------|--------------|-----------|-------------|
| 172.17.240.137    | 172.17.240.146  | 10.10.10.24  | STORAGE   | 10.10.11.22 |
| 172.17.240.138    | 172.17.240.147  | 10.10.10.26  | STORAGE   | 10.10.11.23 |
| 172.17.240.136    | 172.17.240.145  | 10.10.10.22  | STORAGE   | 10.10.11.21 |

## NTP and DNS
- **NTP Daemon:** Running on all nodes
- **NTP Sync Check:** FAIL (Daemon running, but cluster not time-synced per Hypercheck Summary)
- **Primary NTP Source Check:** FAIL (NTP source not confirmed/reachable)
- **DNS Check:** FAIL (DNS resolution/connectivity failed)
- **Configured NTP/DNS IPs:** Not explicitly listed in the report; only NTP server names (e.g., *ntp-appliance-1) are shown.

## Storage Capacity
- **Space State:** healthy
- **Enospc State:** ENOSPACE_CLEAR (no space issues)
- **Claimed Drives:** 7 per node
- **Blacklisted Drives:** 0
- **Ignored Drives:** 1
- **System Disks Usage:** PASS (all <80%)
- **Cache Disks:** PASS
- **Cluster Usage:** Confirmed below 76% (no warnings or errors in report)

## Other Observations
- **Cluster Health:** healthy
- **All nodes online, no critical errors**
- **Services:** Most core services running; some optional services not running (expected if not used)
- **vMotion:** Enabled and tested between all nodes
- **ESXi Version:** VMware ESXi 7.0.3 U3 (24784741)
- **SCVM Version:** 5.0.2e-42642

---

## Key Observations
- <span style="color:red; font-weight:bold">NTP daemon running but NTP sync check FAIL and Primary NTP Source check FAIL.</span>
- **DNS checks FAIL (resolution/connectivity issues).**
- Storage healthy; no space or disk health issues (Claimed=7, Blacklisted=0, Ignored=1).
- All nodes and core services operational; optional services intentionally not running.
- vMotion enabled and functional across storage nodes.
- Cluster usage below 76%.

### Discrepancies / Clarifications
- Removed duplicate table header.
- Added SCVM Data IPs (eth1) column; mapped eth0 (SCVM IP) to corresponding eth1 (10.10.10.22/24/26) as per log Eth1 list order vs SCVM order.
- Corrected NTP Sync status from PASS to FAIL to align with Hypercheck Summary.
- Clarified difference between SCVM Data IPs and ESXi vmk1 storage VMkernel IPs (10.10.10.21/23/25 belong to ESXi vmk1, not SCVM eth1).
