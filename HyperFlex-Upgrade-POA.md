# BKC HyperFlex Upgrade Runbook

This document follows the Cisco-prescribed upgrade sequence for Fabric Interconnect 6400

## Current vs Target Software Versions

| **Component** | **Current Version** | **Target Version** | **Notes** |
|---------------|--------------------|--------------------|-----------|
| **HyperFlex Data Platform (HXDP)** | 5.0.2e | 5.5(2b) | Major version upgrade |
| **ESXi** | 7.0.3 24723872 | 8.0U3-24674464 | Major version upgrade |
| **vCenter** | 7.0.3 24730281 | 8.0 U3 | Major version upgrade |
| **UCS Manager (UCSM)** | 4.2(3h) | 4.2(3o) | Infrastructure firmware upgrade (safe path stays on 4.2 train) |
| **UCS C-Series Server Firmware** | Unknown | 4.2(3o) | Requires validation of current version |


## Upgrade Flow Chart

```mermaid
flowchart TD
  A[Start Upgrade Process] --> B[Pre-Window Checklist & Inventory<br/>48-72 hrs before]
  B --> C{All Prereqs Complete?}
  C -->|No| B
  C -->|Yes| D[Final Pre-Window Checks<br/>T-60m]

  D --> E[Step 1: HXDP 5.0.2e→5.5(2b)<br/>60-120m]
  E --> F{HXDP Success?}
  F -->|No| G[Stop & TAC]
  F -->|Yes| H[Step 2: vCenter 7.0U3→8.0U3<br/>60-90m]

  H --> I{vCenter Success?}
  I -->|No| J[Rollback vCenter & TAC]
  I -->|Yes| K[Step 3: ESXi 7.0U3→8.0U3 (Rolling)<br/>45-90m/host]

  K --> L{All Hosts Success?}
  L -->|No| M[Rollback Host & TAC]
  L -->|Yes| N[Step 4: FI 4.2(3h)→4.2(3o)<br/>30-60m]

  N --> O{FI Success?}
  O -->|No| P[Rollback FI & TAC]
  O -->|Yes| Q[Step 5: Server FW to 4.2(3o)<br/>20-40m/node]

  Q --> R{All Nodes Success?}
  R -->|No| S[Stop HUU & TAC]
  R -->|Yes| T[Post-Validation Checks]

  T --> U{All Validation Passed?}
  U -->|No| V[Investigate / TAC]
  U -->|Yes| W[Re-enable Services]
  W --> X[Upgrade Complete]

  G --> Z[End - Failed]
  J --> Z
  M --> Z
  P --> Z
  S --> Z
  V --> AA{Critical Issues?}
  AA -->|Yes| Z
  AA -->|No| W
  X --> BB[End - Success]
```


## Software Binaries Inventory

https://software.cisco.com/download/home/286305544/type/286305994/release/5.5(2b)
storfs-packages-5.5.2b-43453.tgz
HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip
ucs-6400-k9-bundle-infra.4.2.3o.A.bin
ucs-k9-bundle-b-series.4.2.3o.B.bin
ucs-k9-bundle-c-series.4.2.3o.C.bin

Note: `ucs-6400-k9-bundle-infra.4.3.5d.A.bin` intentionally omitted in safe sequence (remaining on 4.2 train).




## Preflight

**Pre-window checklist (complete 48–72 hrs before) - See table below for detailed breakdown**

Key preparation items:
- Complete all items in the "Pre-Upgrade" phase of the table below
- **Open Proactive TAC Case**: Create Cisco TAC case with upgrade plan, cluster details, and maintenance window
- **Stakeholder Notification**: Notify all stakeholders, ensure TAC case active and ready

## Comprehensive Upgrade Plan - Table Format

| **Phase** | **Step** | **Description** | **Considerations** | **Validation/Checks** | **Estimated Duration** |
|-----------|----------|-----------------|-------------------|----------------------|------------------------|
| **Pre-Upgrade** | Review Release Notes | Check Cisco HX Data Platform Release Notes for new features, supported hardware, interoperability, and caveats | Ensure target version meets environment needs. Identify supported UCS, ESXi, and vCenter versions. | Confirm compatibility using Cisco HyperFlex Software Requirements and Recommendations. | 30 min |
| **Pre-Upgrade** | Verify System Requirements | Ensure VMware vCenter is version 7.0 U2, 7.0 U3, or 8.0 or later. Verify ESXi compatibility with target HXDP version. | vCenter must support TLS 1.2. Interoperability Matrices to verify compatibility. | Use VMware Product Interoperability Matrices to verify compatibility. | 15 min |
| **Pre-Upgrade** | Check Cluster Health | Verify HyperFlex cluster is online and healthy using HX Connect or CLI (hxcli cluster info). | Cluster must be in lenient mode. Ensure no faults on UCS Fabric Interconnects or servers. | Check high availability status of Fabric Interconnects and server faults in Cisco UCS Manager. | 30 min |
| **Pre-Upgrade** | Run Hypercheck Tool | Execute HyperCheck Health & Pre-Upgrade tool to ensure system stability and resiliency. | Identifies potential issues before upgrade. | Review Hypercheck results for misconfigurations or warnings. | 45 min |
| **Pre-Upgrade** | Verify Storage Capacity | Ensure cluster storage utilization is below 76% (capacity + overhead). | Upgrade validation fails if storage exceeds 76%. | Check storage capacity in HX Storage Cluster Overview. | 15 min |
| **Pre-Upgrade** | Backup Configuration | Create an All Configuration backup file in Cisco UCS Manager. | Protects against configuration loss. | Verify backup is complete and accessible. | 30 min |
| **Pre-Upgrade** | Verify vMotion and DRS | Ensure vMotion is configured on all nodes and DRS is enabled (set to fully automated if licensed). | Manual VM migration required if DRS is disabled. | Confirm vMotion interfaces and MTU settings. | 30 min |
| **Pre-Upgrade** | Check ESX Agent Manager (EAM) | Verify EAM health is normal. | EAM issues can disrupt upgrades. | Check EAM status in vCenter. | 15 min |
| **Pre-Upgrade** | Delete Unused UCS Packages | Remove old firmware packages from UCS Fabric Interconnect bootflash. | Prevents bootflash partition from filling up. | Verify available space in UCS Manager under Local Storage Information. | 30 min |
| **Pre-Upgrade** | Verify Spanning Tree | Ensure upstream switches have STP PortFast enabled on Fabric Interconnect ports. | Prevents delays in link state changes during upgrades. | Check switch configuration for PortFast on uplink ports. | 20 min |
| **Pre-Upgrade** | Validate User Credentials | Verify passwords for vCenter admin, ESXi root, and Storage Controller VM (SCVM) admin/root. | Incorrect credentials can halt the upgrade. | Test login credentials for all components. | 20 min |
| **Pre-Upgrade** | Test Upgrade Eligibility | Run eligibility test via HX Connect for UCS firmware, HXDP, and ESXi. | Requires UCS Manager FQDN/IP, vCenter credentials, and upgrade bundles. | Review test results in HX Connect Upgrade page. | 45 min |
| **Pre-Upgrade** | Download Software | Obtain UCS firmware, HXDP upgrade bundle, and ESXi offline bundle from Cisco Software Download site. | Ensure versions match target requirements. | Verify downloaded files are correct and uncorrupted. | 60 min |
| **Pre-Upgrade** | Install se-core Package (if applicable) | For HXDP 5.5 or later, download and install se-core package (.deb.gz) on all nodes. | Required for encryption support; install after cluster creation. | Use hxcli encryption overview to confirm installation (State: NOT_CONFIGURED). | 30 min |
| **Pre-Upgrade** | Open Proactive TAC Case | Create Cisco TAC case with upgrade plan, cluster details, and maintenance window | Ensures TAC support availability during upgrade window | Verify TAC case is active and response team assigned | 30 min |
| **Pre-Upgrade** | Stakeholder Notification | Notify all stakeholders of upgrade plan and maintenance window | Ensures business alignment and communication | Confirm all stakeholders acknowledged upgrade plan | 15 min |
| **Execution** | Final Pre-Window Checks (T-60m) | Verify all pre-upgrade items completed, TAC case ready, team present | Critical go/no-go validation before starting upgrade | All items must be GREEN to proceed | 60 min |
| **Execution** | Step 1: HXDP Upgrade to 5.5(2b) | Rolling CVM upgrade first to establish platform baseline | Eliminates unsupported interim with newer vCenter/ESXi | hxcli cluster show/info healthy each CVM | 60-120 min |
| **Execution** | Step 2: vCenter Upgrade to 8.0 U3 | Perform VCSA upgrade (Stage 1 & 2) | Supported combo with HXDP 5.5(2b) | vCenter services & vCLS VMs healthy | 60-90 min |
| **Execution** | Step 3: ESXi Host Upgrade to 8.0 U3 | Rolling upgrade via HX Connect | One host at a time to preserve availability | Host summary shows ESXi 8.0 U3 | 45-90 min per host |
| **Execution** | Step 4: FI Infrastructure 4.2(3h)→4.2(3o) | Subordinate then primary | Preserve HA; validate data paths | show cluster extended-state; show fault | 30-60 min |
| **Execution** | Step 5: Server Firmware (HUU) to 4.2(3o) | Rolling per node | Can swap with Step 4 if preferred | show server inventory; hxcli healthy | 20-40 min per node |
| **Post-Upgrade** | Post-Validation Checks | Comprehensive validation of all upgraded components | Must pass all checks before declaring success | Run hxcli cluster show, verify all component versions | 30 min |
| **Post-Upgrade** | Verify Cluster Health | Check cluster status ('Ruthenium state: online, healthy') and all CVMs show HXDP 5.5(2b) | Ensures no residual issues from upgrade | Run hxcli cluster info and verify health metrics | 30 min |
| **Post-Upgrade** | Validate VM Functionality | Ensure VMs are running and can be migrated via vMotion. Test DRS/HA functionality | Confirms system stability post-upgrade | Test VM migration and verify no affinity rule conflicts | 45 min |
| **Post-Upgrade** | Check Replication and Drives | Verify data replication compliance, datastores intact, and replication intact | Ensures data integrity and cluster resiliency | Confirm replication factor and drive status in HX Connect | 30 min |
| **Post-Upgrade** | Re-enable Services | Re-enable SIOC and other services if disabled during upgrade | Services may have been disabled for upgrade stability | Verify all services are operational and configured correctly | 30 min |
| **Post-Upgrade** | Monitor Logs | Review upgrade logs for errors or warnings in HX Connect, ESXi, and UCS Manager | Identifies any issues requiring TAC intervention | Check logs in HX Connect or CLI for error messages | 30 min |


## Detailed Runbook

Detailed step-by-step runbook following the prescribed upgrade sequence

### Step 0 — Final Pre-Window Checks (T-60m)

**Critical Go/No-Go Validation:**
- **Verify all Pre-Upgrade table items completed successfully**
- **TAC Ready**: **Verify proactive TAC case is open** (case #: ____________)
- **Team Ready**: All team members present, TAC contact information available
- **Software Ready**: All upgrade bundles downloaded and validated
- **Cluster Health**: Final cluster health check - must be online/healthy

**Decision Point**: All items must be GREEN to proceed. Any RED items require resolution before starting upgrade.

---

### Step 1 — HXDP Upgrade 5.0.2e → 5.5(2b) (60–120 minutes)

Goal: Establish supported data platform baseline first to avoid interim incompatibilities.

Prerequisites:
- Cluster healthy (hxcli cluster show/info)
- Storage utilization < 76%
- Eligibility & hypercheck clean
- Proactive TAC case active

Execution:
1. Upload HXDP bundle `storfs-packages-5.5.2b-43453.tgz` in HX Connect → Upgrades → Data Platform
2. Run pre-checks and remediate any warnings (STOP on failures)
3. Start rolling upgrade – CVMs upgrade sequentially (expect 10–15 min each + services settle)
4. After each CVM completion: verify cluster health (hxcli cluster show)

Validation:
```bash
hxcli cluster show
hxcli cluster info
hxcli cluster storage-summary
```
All CVMs must report 5.5(2b), no degraded components.

Rollback:
- If a CVM fails upgrade or health degrades: pause, collect logs (hxcli log collect), engage TAC. Do not advance sequence until resolved.

---

### Step 2 — vCenter Upgrade 7.0 U3 → 8.0 U3 (60–90 minutes)

Goal: Upgrade management plane now that HXDP is at 5.5(2b).

Prerequisites:
- HXDP 5.5(2b) stable
- Recent VCSA backup & (optional) snapshot
- DNS, NTP verified

Execution:
1. Launch installer ISO (mac/Win) → Stage 1 deploy new appliance
2. Stage 2 data migration & configuration
3. Monitor installer progress & VAMI (https://vc:5480)
4. Remove snapshot after success (if used)

Validation:
- Services: All green in VAMI
- vCLS VMs healthy
- Inventory & permissions intact

Rollback:
- Failure during Stage 2: power off new appliance, restore original from backup; engage TAC.

---

### Step 3 — ESXi Host Upgrade 7.0 U3 → 8.0 U3 (Rolling, 45–90 min/host)

Goal: Upgrade hypervisor layer under upgraded HXDP & vCenter.

Prerequisites:
- HXDP 5.5(2b) & vCenter 8.0 U3 healthy
- vMotion & DRS operational; no resync / rebuild backlog
- Sufficient cluster capacity N+1

Execution (HX Connect preferred):
1. Upload `HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip`
2. Initiate rolling host upgrade (one at a time) via HX Connect → Upgrades → Host Upgrade
3. For each host: evacuate VMs (vMotion), enter maintenance mode, apply bundle, reboot, exit maintenance
4. Confirm host rebalances before next

Fallback Manual (esxcli):
```bash
vim-cmd hostsvc/maintenance_mode_enter
esxcli software sources profile list -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip
esxcli software profile update -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip -p HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7
reboot
vim-cmd hostsvc/maintenance_mode_exit
```

Validation Per Host:
- Version shows ESXi 8.0 U3
- Host rejoined cluster; no new PSOD / hardware alarms
- HX cluster healthy (hxcli cluster show)

Rollback:
- Attempt alt boot option (previous ESXi) or reinstall previous image profile; engage TAC.

---

### Step 4 — Fabric Interconnect Infrastructure Firmware 4.2(3h) → 4.2(3o) (30–60 minutes)

Goal: Move FIs within 4.2 train after compute layer stable.

Prerequisites:
- ESXi fleet upgraded (or deviation approved via TAC)
- UCS cluster HA: subordinate identified; no critical faults

Execution:
1. Upload infra bundle `ucs-6400-k9-bundle-infra.4.2.3o.A.bin`
2. Upgrade subordinate FI first; monitor reboot (20–30m)
3. Validate:
```bash
show cluster extended-state
show fault
```
4. Upgrade primary FI; brief single-homed period; monitor services
5. Final validation:
```bash
show cluster extended-state
show fabric-interconnect detail
show server status
show fault
```

Rollback:
- If failure: attempt previous image activation (startup version), keep other FI stable; TAC.

---

### Step 5 — Server (B/C Series) Firmware (HUU) to 4.2(3o) (20–40 min/node)

Goal: Align server firmware with infra version 4.2(3o); maintains supported matrix.

Prerequisites:
- FI pair on 4.2(3o) stable
- HX cluster healthy; capacity normal; no rebuild backlog

Execution:
1. Upload `ucs-k9-bundle-b-series.4.2.3o.B.bin` & `ucs-k9-bundle-c-series.4.2.3o.C.bin`
2. HX Connect → Upgrades → Hardware → Host Firmware Upgrade → Rolling
3. Per node: maintenance mode -> firmware apply -> reboot -> exit maintenance
4. Verify firmware inventory before next node

Validation:
```bash
hxcli cluster show
show server inventory
```

Rollback:
- Halt job; do not proceed to next node; restore prior package if possible; TAC.

---

### Step 6 — Post-Validation & Service Restoration (60–90 minutes)

Comprehensive System Validation:
1. Cluster Health:
```bash
hxcli cluster show
hxcli cluster info
hxcli cluster storage-summary
```
2. Version Audit:
  - vCenter 8.0 U3
  - ESXi 8.0 U3 on all hosts
  - HXDP 5.5(2b) on all CVMs
  - FI firmware 4.2(3o) (both)
  - Server firmware 4.2(3o)
3. Functional Tests: vMotion, DRS/HA, datastore accessibility, VM performance spot-check
4. Data Integrity: replication factor compliance, no degraded disks
5. Services: re-enable SIOC / any temporarily disabled features
6. Log Review: HX Connect, VCSA tasks/events, UCS faults, ESXi host logs

Acceptance Criteria (all TRUE): versions correct, cluster healthy, no critical faults, workloads stable.

Completion:
- Update TAC & stakeholders
- Capture runbook deviations & lessons learned
- Schedule 24h post-monitor review

---

**Comprehensive System Validation:**

1. **Cluster Health Verification:**
```bash
# Verify cluster online and healthy
hxcli cluster show
hxcli cluster info

# Check replication status
hxcli cluster storage-summary
```

2. **Component Version Validation:**
  - **vCenter**: Confirm v8.0 U3
  - **ESXi**: All hosts show ESXi 8.0 U3
  - **HXDP**: All CVMs show 5.5(2b)
  - **UCS Firmware**: Confirm FI 4.2(3o) and servers 4.2(3o)

3. **Functional Testing:**
  - **Test vMotion between hosts**
  - **Validate DRS/HA functionality**
  - **Verify datastore accessibility**
  - **Confirm VM functionality**

4. **Data Integrity Checks:**
  - **Verify replication factor compliance**
  - **Check datastore integrity**
  - **Validate no data loss**

5. **Service Restoration:**
  - **Re-enable SIOC (if disabled)**
  - **Restore any disabled services**
  - **Clear maintenance notifications**

6. **Log Review:**
  - **HX Connect upgrade logs**
  - **ESXi host logs (/var/log/vmware.log)**
  - **UCS Manager event logs**
  - **vCenter task and event logs**

**Final Acceptance Criteria:**
- ✅ All components at target versions
- ✅ Cluster state: "Ruthenium state: online, healthy"
- ✅ No critical faults or alarms
- ✅ VM functionality confirmed
- ✅ vMotion and DRS operational
- ✅ Data replication compliant

**Completion Actions:**
- **Update TAC case with successful completion**
- **Notify stakeholders of upgrade completion**
- **Document any issues encountered**
- **Schedule post-upgrade monitoring period**

---

### Emergency Procedures

**If Critical Issues Arise:**
1. **Stop current upgrade phase immediately**
2. **Document exact error messages and symptoms**
3. **Contact TAC immediately with case details**
4. **Preserve system state for TAC analysis**
5. **Follow TAC guidance for recovery procedures**

**Escalation Contacts:**
- **Primary TAC Case**: ____________
- **Secondary TAC Number**: 1-800-553-2447
- **Internal Escalation**: [Team Lead Contact]


## Time Estimates Summary - 8-Node Cluster

**Note:** Conservative estimates with buffer included for 8-node cluster environment.

| **Phase** | **Component** | **Estimated Duration** | **8-Node Calculation** | **Notes** |
|-----------|--------------|------------------------|-------------------------|-----------|
| **Preparation** | Final pre-window checks (T-60m) | 60 minutes | - | Critical go/no-go validation before starting |
| **Step 1** | HXDP upgrade to 5.5(2b) | 60-120 minutes | 8 CVMs sequential | Rolling CVM upgrades + health gates |
| **Step 2** | vCenter upgrade to 8.0 U3 | 60-90 minutes | - | Stage 1 & 2 + validation buffer |
| **Step 3** | ESXi host upgrade to 8.0 U3 | 6-12 hours | 8 hosts × 45-90 min | Largest rolling component (one host at a time) |
| **Step 4** | FI Infrastructure (A/B) 4.2(3h)→4.2(3o) | 30-60 minutes | - | Subordinate then primary with HA checks |
| **Step 5** | Server Firmware (HUU) rolling 4.2(3o) | 4-6 hours | 8 nodes × 20-40 min | Sequential node firmware alignment |
| **Step 6** | Post-validation & service restoration | 60-90 minutes | - | Comprehensive functional + integrity testing |

**Total Estimated Window:** 14-21 hours (conservative estimate; overlap limited due to sequential safeguards)

**Critical Path Components:**
- **ESXi Host Upgrades**: 6-12 hours
- **Server Firmware (HUU)**: 4-6 hours
- **Combined Rolling (Hosts + HUU)**: 10-18 hours

**Additional Considerations:**
- Network speed affects bundle upload times
- Any troubleshooting adds time
- Can be split across multiple maintenance windows if needed
- Non-production environment allows for extended weekday windows


**Critical Variables Affecting Time:**
- Number of ESXi hosts (host rolling duration)
- Number of UCS servers (HUU rolling duration)
- HyperFlex CVM count (HXDP sequential CVM time)
- Network throughput (bundle upload / image staging)
- Rebuild / resync backlog (may pause between hosts)
- Unexpected remediation (faults, TAC investigations)


