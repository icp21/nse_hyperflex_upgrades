# NSE HyperFlex Upgrade Runbook

This document follows the Cisco-prescribed upgrade sequence for Fabric Interconnect 6400 + VIC-1455 (copper SFP) environments:
1) UCS Infrastructure (Fabric Interconnect) firmware → 2) UCS server firmware (HUU) → 3) HXDP upgrade → 4) vCenter upgrade → 5) ESXi host upgrades (rolling).

Important FI/Server Firmware Notes:

**Critical Restrictions:**

HXAF240-M5 Clusters using Samsung SSDs with 3.8TB and 7.6TB capacities:
- **HXDP 5.5(2x)**: Install or upgrade to UCS versions 4.2(3l) or later for HXAF240-M5 Clusters using Samsung SSD drives with PID HX-SD76T61X-EV or HX-SD38T61X-EV (UCS-SD76T61X-EV or UCS-SD38T61X-EV)
- **ESXi 7.0U3 Compatibility**: Compatible with target versions, allowing flexibility in vCenter and ESXi upgrade timing


## Inventory

https://software.cisco.com/download/home/286305544/type/286305994/release/5.5(2b)

**Required Software Bundles:**
- **HXDP Bundle**: storfs-packages-5.5.2b-43453.tgz
- **ESXi Bundle**: HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip
- **UCS FI Infrastructure**: ucs-6400-k9-bundle-infra.4.2.3o.A.bin
- **UCS B-Series HUU**: ucs-k9-bundle-b-series.4.2.3o.B.bin
- **UCS C-Series HUU**: ucs-k9-bundle-c-series.4.2.3o.C.bin



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
| **Execution** | Perform Upgrade | Use HX Connect UI for upgrades from HXDP 3.5 (1a) or later (auto-bootstrap). For earlier versions, use manual bootstrap via CLI. | Perform during low-traffic hours or maintenance window. Offline upgrades require VMs to be shut down. | Monitor upgrade progress in HX Connect or CLI. Ensure no errors in logs. | Variable |
| **Execution** | Upgrade Components | Execute combined upgrade (HXDP + ESXi) or individual upgrades based on cluster needs. | Combined upgrades optimize reboots. Split upgrades: HXDP first, then ESXi. | Verify each component upgrade completes successfully. | Variable |
| **Post-Upgrade** | Verify Cluster Health | Check cluster status ('Ruthenium state: online, healthy'). | Ensures no residual issues from upgrade. | Run hxcli cluster info and verify health metrics. | 30 min |
| **Post-Upgrade** | Validate VM Functionality | Ensure VMs are running and can be migrated via vMotion. Ascertain that DRS and vMotion are functioning correctly. | Confirms system stability post-upgrade. | Test VM migration and verify no affinity rule conflicts. | 45 min |
| **Post-Upgrade** | Check Replication and Drives | Verify data replication compliance and drive failures. | Ensures data integrity and cluster resiliency. | Confirm replication factor and drive status in HX Connect. | 30 min |
| **Post-Upgrade** | Monitor Logs | Review upgrade logs for errors or warnings. | Identifies any issues requiring TAC intervention. | Check logs in HX Connect or CLI for error messages. | 30 min |

## Execution Sequence

**Upgrade sequence (MUST follow for FI 6400 + VIC-1455):**
1. **UCS Infrastructure (Fabric Interconnect) firmware upgrade (A/B)** — preserve HA, upgrade subordinate FI first
2. **UCS server firmware (HUU) upgrade (rolling)** — via HX Connect HUU flow  
3. **HX Data Platform upgrade → HXDP 5.5(2b)** (cluster rolling)
4. **vCenter upgrade to v8.0.x** (must be ≥ ESXi target)
5. **ESXi host upgrades to ESXi 8.0 U3** — using HX Connect / Cisco offline bundle (rolling)
6. **Post-validation and re-enable services** if needed



## Detailed Runbook

Detailed step-by-step runbook

Step 0 — Final pre-window checks

**Critical Go/No-Go Validation (T-60m):**
- **Verify all Pre-Upgrade table items completed successfully**
- **TAC Ready**: **Verify proactive TAC case is open** (case #: ____________)
- **Team Ready**: All team members present, TAC contact information available

**Decision Point**: All items must be GREEN to proceed. Any RED items require resolution before starting upgrade.

Step 1 — UCS Infrastructure (Fabric Interconnect) upgrade
Goal: upgrade FI A/B to qualified FI image while preserving HA.

**Phase 1A - Upload FI Image:**
- Upload FI image to UCSM: Equipment → Fabric Interconnects → Firmware Management → Downloaded Images
- Verify image integrity and version compatibility

**Phase 1B - Upgrade Subordinate FI First:**
1. Identify subordinate FI: Equipment → Fabric Interconnects → check HA Status
2. Navigate to: Equipment → Fabric Interconnects → [Subordinate-FI] → General → Firmware Management → Update Firmware
3. Select uploaded firmware bundle and click OK
4. FI will reboot automatically

**Phase 1C - Critical Validation Before Upgrading Primary FI:**

**MANDATORY: Following Dell UCS Best Practices - Verify Data Paths Before Primary FI Reboot**

```bash
# 1. Check cluster state and confirm HA READY:
UCSM-A# show cluster extended-state
# Verify: Both FIs show UP, mgmt services UP, HA READY

# 2. Verify Fibre Channel Data Paths (if applicable):
UCSM-A# connect nxos a
UCSM-A(nxos)# show npv flogi-table
UCSM-A(nxos)# exit

UCSM-A# connect nxos b  
UCSM-A(nxos)# show npv flogi-table
UCSM-A(nxos)# exit
# Record total flogis for both FIs - numbers should match

# 3. Verify Ethernet Data Paths:
UCSM-A# connect nxos a
UCSM-A(nxos)# show int br | grep -v down | wc -l
UCSM-A(nxos)# exit

UCSM-A# connect nxos b
UCSM-A(nxos)# show int br | grep -v down | wc -l  
UCSM-A(nxos)# exit
# Record active interface counts for each FI

# 4. Check for active faults (must be clear):
UCSM-A# show fault
# Clear any faults before proceeding

# CRITICAL: Only proceed to primary FI reboot if all validations pass
```

**Phase 1D - Upgrade Primary FI:**
1. Only proceed if ALL validations pass
2. Follow same procedure for primary FI
3. System will briefly be single-homed during primary FI reboot

**Phase 1E - Final FI Validation:**
```bash
# 1. Verify cluster state and HA status:
UCSM-A# show cluster extended-state
# Confirm: Both FIs UP, PRIMARY/SUBORDINATE roles, HA READY

# 2. Re-verify all data paths match pre-upgrade baselines:
UCSM-A# connect nxos a
UCSM-A(nxos)# show npv flogi-table
UCSM-A(nxos)# show int br | grep -v down | wc -l
UCSM-A(nxos)# exit

UCSM-A# connect nxos b
UCSM-A(nxos)# show npv flogi-table  
UCSM-A(nxos)# show int br | grep -v down | wc -l
UCSM-A(nxos)# exit

# 3. Verify server connectivity:
UCSM-A# show server status
UCSM-A# show fault

# 4. Confirm firmware versions updated:
UCSM-A# show fabric-interconnect detail
```

Step 2 — UCS server firmware (HUU) — rolling
Goal: apply HUU 'ucs-huu-4.2(3o).zip' to all blades (one node at a time).

**Prerequisites:**
- Confirm UCS Infrastructure firmware upgrade completed successfully
- Verify all FI paths operational

HX Connect → Repository → Upload ucs-huu-4.2(3o).zip (verify SHA256)

For each node:
  - Host enters maintenance mode per HX Connect
  - Node reboots and firmware applied
After each node:
  - Validate server firmware versions in UCS Manager
Validation:
  - hxcli cluster show
Rollback:
  - If a node fails to boot: stop HUU job, reapply previous firmware if staged, restore UCS config and open TAC.

Step 3 — HXDP upgrade to 5.5(2b)

**Prerequisites:**
- Confirm UCS server firmware upgrade completed successfully
- Verify cluster health: hxcli cluster show
Upload hxdp-5.5.2b-bundle.tar.gz to HX Connect.
HX Connect → Upgrades → HXDP upgrade → select management cluster → run prechecks and start.
Monitor CVM rolling upgrades and ensure cluster health.
**Note:** 8-node cluster will perform rolling upgrade of all 8 CVMs sequentially.

Validation:
- hxcli cluster show
Rollback:
- If HXDP fails, stop and open TAC with hypercheck output; do not proceed to host upgrades.


Step 5 — vCenter upgrade to v8.0.x
Final vCenter backup.
Perform VCSA upgrade (stage 1 & 2) per VMware KB.
Validate vCenter services and vCLS VMs.

Step 6 — ESXi hosts upgrade to ESXi 8.0 U3 (rolling)
Preferred: HX Connect host upgrade job with Cisco offline bundle.
Upload cisco-esxi-8.0U3-offline.zip to HX Connect.
HX Connect → Upgrades → Host Upgrade → select hosts → choose Cisco ESXi bundle → rolling.
Monitor host evacuation, patch, reboot and return to cluster.

Manual esxcli template (only if HX Connect unavailable):
# put host into maintenance
vim-cmd hostsvc/maintenance_mode_enter
# list available profiles in bundle
esxcli software sources profile list -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip
# apply chosen profile (replace <PROFILE_NAME>)
esxcli software profile update -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip -p HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7
reboot
vim-cmd hostsvc/maintenance_mode_exit

Validation:
- Host summary shows ESXi 8.0 U3


## Post-validation

**Post-upgrade validation & acceptance checks (see Post-Upgrade table items for detailed validation):**
- hxcli cluster show — cluster online/healthy
- All CVMs show HXDP 5.5(2b)
- vCenter is v8.0.x and ESXi hosts show ESXi 8.0 U3
- Datastores intact and replication intact
- Test vMotion between hosts and validate DRS/HA
- Review HX Connect logs, ESXi logs, UCS Manager faults

## Time Estimates Summary - 8-Node Cluster

**Note:** Conservative estimates with buffer included for 8-node cluster environment.

| **Phase** | **Component** | **Estimated Duration** | **8-Node Calculation** | **Notes** |
|-----------|--------------|------------------------|-------------------------|-----------|
| **Preparation** | Pre-window checks | 90 minutes | - | Includes final validations, team readiness, buffer |
| **Step 1** | UCS Infrastructure (FI) upgrade | 60-90 minutes | - | Complete A/B upgrade with validation + buffer |
| **Step 1B** | Subordinate FI reboot | 20-40 minutes | - | Per FI reboot and comeback + buffer |
| **Step 1D** | Primary FI reboot | 20-40 minutes | - | Per FI reboot and comeback + buffer |
| **Step 2** | UCS Server Firmware (HUU) | 4-6 hours | 8 nodes × 30-45 min | **Major time component** - rolling upgrade |
| **Step 3** | HXDP upgrade (8 CVMs) | 2-3 hours | - | 8-node cluster CVM rolling upgrade + buffer |
| **Step 5** | vCenter upgrade | 90-120 minutes | - | Both upgrade stages + buffer |
| **Step 6** | ESXi host upgrade | 8-12 hours | 8 hosts × 60-90 min | **Largest time component** - rolling upgrade |
| **Post-validation** | Final checks and testing | 60-90 minutes | - | Comprehensive validation + buffer |

**Total Estimated Window:** 16-24 hours (conservative estimate for 8-node cluster)

**Recommended Execution Strategy for Non-Production Cluster:**
- **Weekday maintenance window** (client preference for non-prod environment)
- **Extended weekday window**: Plan for 2-3 consecutive weekdays if needed
- **Staged approach**: Can split across multiple weekday maintenance windows
- **Critical path**: HUU (4-6h) + HXDP (2-3h) + ESXi upgrades (8-12h) = 14-21 hours core upgrade time
- **Business hours flexibility**: Non-prod allows for daytime execution and troubleshooting

**Critical Variables Affecting Time:**
- Number of UCS servers (affects HUU duration)
- Number of ESXi hosts (affects host upgrade duration)
- HyperFlex cluster size (affects HXDP upgrade duration)
- Network speed and bundle upload times
- Any unexpected issues requiring troubleshooting



