# Cluster Analysis Report: KOH-HX-VDI-Cluster

**Cluster Type:** DEFAULT_CLUSTER  
**vCenter:** koh-vmw-vcenter01.nseroot.com

## ESXi Hosts, SCVM IPs, SCVM Data IPs, vMotion IPs
| ESXi Host IP      | SCVM IP         | SCVM Data IP | Node Role | vMotion IP   |
|-------------------|-----------------|--------------|-----------|-------------|
| 172.17.240.223    | None            | None         | COMPUTE   | 10.10.11.31 |
| 172.17.240.131    | 172.17.240.140  | 10.10.10.34  | STORAGE   | 10.10.11.32 |
| 172.17.240.133    | 172.17.240.142  | 10.10.10.38  | STORAGE   | 10.10.11.34 |
| 172.17.240.132    | 172.17.240.141  | 10.10.10.36  | STORAGE   | 10.10.11.33 |
| 172.17.240.135    | 172.17.240.144  | 10.10.10.42  | STORAGE   | 10.10.11.36 |
| 172.17.240.130    | 172.17.240.139  | 10.10.10.32  | STORAGE   | 10.10.11.35 |
| 172.17.240.225    | None            | None         | COMPUTE   | 10.10.11.41 |
| 172.17.240.134    | 172.17.240.143  | 10.10.10.40  | STORAGE   | 10.10.11.37 |

## NTP and DNS
- **NTP Daemon:** Running on all nodes
- **NTP Sync Check:** FAIL (Daemon running, but cluster not time-synced per Hypercheck)
- **Primary NTP Source Check:** FAIL (NTP source not confirmed/reachable)
- **DNS Check:** FAIL (DNS resolution/connectivity failed)
- **Configured NTP/DNS IPs:** Not explicitly listed in the report; only NTP server names (e.g., *ntp-appliance-1/2) are shown.

## Storage Capacity
- **Space State:** healthy
- **Enospc State:** ENOSPACE_CLEAR (no space issues)
- **Claimed Drives:** 18 per node
- **Blacklisted Drives:** 0
- **Ignored Drives:** 1
- **System Disks Usage:** PASS (all <80%)
- **Cache Disks:** PASS
- **Cluster Usage:** Confirmed below 76% (no warnings or errors in report)

## Other Observations
- **Cluster Health:** healthy
- **All nodes online, no critical errors**
- **Services:** Most core services running; some optional services (e.g., iSCSI, HyperV, Replication) not running (expected if not used)
- **vMotion:** Enabled and tested between all nodes
- **ESXi Version:** VMware ESXi 7.0.3 U3 (24784741)
- **SCVM Version:** 5.0.2e-42642

---

## Key Observations
- <span style="color:red; font-weight:bold">NTP daemon is running but NTP sync check is FAIL and the Primary NTP Source check is also FAIL (source not resolved).</span>
- **DNS checks are FAIL (resolution/connectivity issues).**
- Storage is healthy; no space or disk health issues (Claimed=18, Blacklisted=0, Ignored=1).
- All nodes and core services operational; optional services (iSCSI, HyperV, Replication) intentionally not running.
- vMotion enabled and functional across compute and storage nodes.
- Cluster usage below 76% (no capacity warning thresholds breached).

### Discrepancies Corrected in this Report
- Removed duplicate legacy table header line.
- Corrected SCVM Data IPs (eth1) – prior values mistakenly used ESXi vmk1 or fabricated IPs (10.10.10.33/35/37 not present). Eth1 IP set validated from log: 10.10.10.32,34,36,38,40,42.
- Clarified NTP Sync status (was incorrectly documented as PASS; Hypercheck Summary shows FAIL).
- Added explicit distinction between SCVM Data IPs (eth1) and ESXi vmk1 storage VMkernel interfaces.
