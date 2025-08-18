# BKC HyperFlex Upgrade Runbook

This document follows the Cisco-prescribed upgrade sequence for Fabric Interconnect 6400

## Current vs Target Software Versions

| **Component** | **Current Version** | **Target Version** | **Notes** |
|---------------|--------------------|--------------------|-----------|
| **HyperFlex Data Platform (HXDP)** | 5.0.2e | 5.5(2b) | Major version upgrade |
| **ESXi** | 7.0.3 24723872 | 8.0U3-24674464 | Major version upgrade |
| **vCenter** | 7.0.3 24730281 | 8.0 U3 | Major version upgrade |
| **UCS Manager (UCSM)** | 4.2(3H) | 4.3.5d | Infrastructure firmware upgrade |
| **UCS C-Series Server Firmware** | Unknown | 4.2.3o | Requires validation of current version |


## Upgrade Flow Chart

```mermaid
flowchart TD
  A[Start Upgrade Process] --> B[Pre-Window Checklist<br/>48-72 hrs before]
  B --> C{All Prerequisites<br/>Complete?}
  C -->|No| B
  C -->|Yes| D[Final Pre-Window Checks<br/>T-60m]
  
  D --> E[Step 1: Upgrade vCenter<br/>to v8.0.x<br/>60-90 min]
  E --> F{vCenter Upgrade<br/>Successful?}
  F -->|No| G[Rollback vCenter<br/>Open TAC Case]
  F -->|Yes| H[Step 2: Upgrade UCS Infrastructure Firmware<br/>Fabric Interconnect A/B<br/>30-60 min]
  
  H --> I{FI Upgrade<br/>Successful?}
  I -->|No| J[Rollback FI<br/>Open TAC Case]
  I -->|Yes| K[Step 3: Upgrade UCS Firmware, HXDP and ESXi]
  
  K --> L[Sub-step 3a: UCS Server Firmware HUU<br/>Rolling upgrade<br/>20-40 min per node]
  L --> M{HUU Upgrade<br/>Successful?}
  M -->|No| N[Stop HUU job<br/>Restore UCS config<br/>Open TAC Case]
  M -->|Yes| O[Sub-step 3b: HXDP Upgrade<br/>to 5.5(2b)<br/>60-120 min]
  
  O --> P{HXDP Upgrade<br/>Successful?}
  P -->|No| Q[Stop HXDP upgrade<br/>Open TAC Case]
  P -->|Yes| R[Sub-step 3c: ESXi Host Upgrade<br/>to 8.0 U3<br/>45-90 min per host]
  
  R --> S{ESXi Upgrade<br/>Successful?}
  S -->|No| T[Rollback ESXi<br/>Open TAC Case]
  S -->|Yes| U[Post-Validation Checks]
  
  U --> V{All Validation<br/>Passed?}
  V -->|No| W[Investigate Issues<br/>Contact TAC if needed]
  V -->|Yes| X[Re-enable Services<br/>SIOC, etc.]
  X --> Y[Upgrade Complete]
  
  G --> Z[End - Failed]
  J --> Z
  N --> Z
  Q --> Z
  T --> Z
  W --> AA{Critical<br/>Issues?}
  AA -->|Yes| Z
  AA -->|No| X
  Y --> BB[End - Success]
```


## Software Binaries Inventory

https://software.cisco.com/download/home/286305544/type/286305994/release/5.5(2b)
storfs-packages-5.5.2b-43453.tgz
HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip
ucs-6400-k9-bundle-infra.4.3.5d.A.bin
ucs-k9-bundle-b-series.4.2.3o.B.bin
ucs-k9-bundle-c-series.4.2.3o.C.bin
ucs-6400-k9-bundle-infra.4.2.3o.A.bin




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
| **Execution** | Step 1: Upgrade vCenter to v8.0.x | Perform VCSA upgrade (stage 1 & 2) per VMware KB | vCenter must be upgraded before ESXi. Create final backup first | Validate vCenter services and vCLS VMs | 60-90 min |
| **Execution** | Step 2: Upgrade UCS Infrastructure (FI A/B) | Upload FI firmware, upgrade subordinate FI first, then primary FI | MUST preserve HA. Verify data paths before primary FI reboot | Verify cluster state, data paths, and firmware versions | 30-60 min |
| **Execution** | Step 3a: UCS Server Firmware (HUU) Rolling | Apply HUU firmware to all servers one at a time via HX Connect | Upload HUU bundle, rolling upgrade maintains cluster availability | Validate server firmware versions after each node | 20-40 min per node |
| **Execution** | Step 3b: HXDP Upgrade to 5.5(2b) | Upload HXDP bundle, perform cluster rolling upgrade of all CVMs | 8-node cluster performs sequential CVM upgrades | Monitor CVM upgrades, verify cluster health with hxcli cluster show | 60-120 min |
| **Execution** | Step 3c: ESXi Host Upgrade to 8.0 U3 | Rolling upgrade using HX Connect with Cisco offline bundle | Preferred method via HX Connect, manual esxcli as backup | Host summary shows ESXi 8.0 U3, verify maintenance mode cycles | 45-90 min per host |
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

### Step 1 — vCenter Upgrade to v8.0.x (60-90 minutes)

**Goal**: Upgrade vCenter to v8.0 U3 before ESXi upgrades per Cisco sequence

**Prerequisites:**
- Final vCenter backup completed
- All Pre-Upgrade validations passed

**Execution:**
1. **Create final vCenter backup**
2. **Download vCenter 8.0 U3 ISO from VMware**
3. **Perform VCSA upgrade (Stage 1 & 2) per VMware documentation**
  - Stage 1: Install new vCenter appliance
  - Stage 2: Data migration and configuration transfer
4. **Monitor upgrade progress and logs**

**Validation:**
- vCenter services operational
- vCenter Lifecycle Manager (vCLS) VMs healthy
- All hosts visible and manageable
- No critical alarms or errors

**Rollback Plan:**
- If upgrade fails: Restore from backup, open TAC case
- **STOP**: Do not proceed to infrastructure upgrades if vCenter upgrade fails

---

### Step 2 — UCS Infrastructure Firmware Upgrade (30-60 minutes)

**Goal**: Upgrade Fabric Interconnect A/B to qualified firmware while preserving HA

**Prerequisites:**
- vCenter upgrade completed successfully
- UCS Manager accessible and stable

**Phase 2A - Upload FI Image:**
- Upload FI firmware bundle to UCSM: Equipment → Fabric Interconnects → Firmware Management → Downloaded Images
- Verify image integrity: `ucs-6400-k9-bundle-infra.4.3.5d.A.bin`

**Phase 2B - Upgrade Subordinate FI First:**
1. Identify subordinate FI: Equipment → Fabric Interconnects → check HA Status
2. Navigate to: Equipment → Fabric Interconnects → [Subordinate-FI] → General → Firmware Management → Update Firmware
3. Select uploaded firmware bundle and click OK
4. **Monitor FI reboot and comeback (20-30 minutes)**

**Phase 2C - Critical Validation Before Primary FI Upgrade:**

```bash
# 1. Verify cluster state and HA status:
UCSM-A# show cluster extended-state
# Verify: Both FIs show UP, mgmt services UP, HA READY

# 2. Verify Fibre Channel data paths (if applicable):
UCSM-A# connect nxos a
UCSM-A(nxos)# show npv flogi-table
UCSM-A(nxos)# exit

UCSM-A# connect nxos b  
UCSM-A(nxos)# show npv flogi-table
UCSM-A(nxos)# exit
# Record total flogis for both FIs - numbers should match

# 3. Verify Ethernet data paths:
UCSM-A# connect nxos a
UCSM-A(nxos)# show int br | grep -v down | wc -l
UCSM-A(nxos)# exit

UCSM-A# connect nxos b
UCSM-A(nxos)# show int br | grep -v down | wc -l  
UCSM-A(nxos)# exit

# 4. Check for active faults:
UCSM-A# show fault
# Clear any critical faults before proceeding
```

**Phase 2D - Upgrade Primary FI:**
1. **ONLY proceed if ALL validations pass**
2. Follow same firmware update procedure for primary FI
3. **Monitor primary FI reboot (20-30 minutes)**
4. System will be briefly single-homed during primary FI reboot

**Phase 2E - Final FI Validation:**
```bash
# Verify HA restoration and data paths
UCSM-A# show cluster extended-state
UCSM-A# show fabric-interconnect detail
UCSM-A# show server status
UCSM-A# show fault
```

**Rollback Plan:**
- If FI upgrade fails: Restore previous firmware, open TAC case

---

### Step 3 — Combined UCS, HXDP, and ESXi Upgrades

**Goal**: Perform coordinated upgrade of server firmware, HXDP, and ESXi in sequence

#### Step 3a — UCS Server Firmware (HUU) Rolling Upgrade (4-6 hours for 8 nodes)

**Prerequisites:**
- UCS Infrastructure firmware upgrade completed successfully
- All FI paths operational and validated

**Execution:**
1. **Upload HUU bundle to HX Connect**: `ucs-k9-bundle-b-series.4.2.3o.B.bin` and `ucs-k9-bundle-c-series.4.2.3o.C.bin`
2. **HX Connect → Upgrades → Hardware → Host Firmware Upgrade**
3. **Select rolling upgrade option (one node at a time)**
4. **Monitor each node upgrade (20-40 minutes per node)**

**Per Node Process:**
- Host enters maintenance mode automatically
- Node reboots and firmware applied
- Host returns to cluster
- **Validate server firmware version in UCS Manager before proceeding to next node**

**Validation After Each Node:**
```bash
# Check cluster health
hxcli cluster show

# Verify UCS server firmware
UCSM-A# show server inventory
```

**Rollback Plan:**
- If node fails to boot: Stop HUU job, restore UCS config, open TAC case

#### Step 3b — HXDP Upgrade to 5.5(2b) (60-120 minutes)

**Prerequisites:**
- UCS server firmware upgrade completed successfully
- Cluster health verified: `hxcli cluster show`

**Execution:**
1. **Upload HXDP bundle to HX Connect**: `storfs-packages-5.5.2b-43453.tgz`
2. **HX Connect → Upgrades → Data Platform**
3. **Select management cluster and run prechecks**
4. **Monitor CVM rolling upgrades** (8 CVMs will upgrade sequentially)
5. **Verify cluster health throughout process**

**Validation:**
```bash
# Verify HXDP version and cluster health
hxcli cluster show
hxcli cluster info

# Confirm all CVMs running HXDP 5.5(2b)
hxcli cluster storage-summary
```

**Rollback Plan:**
- If HXDP upgrade fails: Stop upgrade, run hypercheck, open TAC case
- **DO NOT proceed to ESXi upgrades if HXDP fails**

#### Step 3c — ESXi Host Upgrade to 8.0 U3 (8-12 hours for 8 hosts)

**Prerequisites:**
- HXDP upgrade completed successfully
- All CVMs running HXDP 5.5(2b)
- vCenter 8.0 operational

**Execution (Preferred Method - HX Connect):**
1. **Upload Cisco ESXi bundle**: `HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip`
2. **HX Connect → Upgrades → Host Upgrade**
3. **Select all hosts for rolling upgrade**
4. **Choose Cisco ESXi 8.0 U3 bundle**
5. **Monitor host evacuation, upgrade, and return (45-90 minutes per host)**

**Per Host Process:**
- vMotion evacuates VMs from host
- Host enters maintenance mode
- ESXi upgrade applied
- Host reboots and rejoins cluster
- **Validate ESXi version before proceeding to next host**

**Manual esxcli Template (Backup Method):**
```bash
# Put host into maintenance mode
vim-cmd hostsvc/maintenance_mode_enter

# List available profiles
esxcli software sources profile list -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip

# Apply upgrade profile
esxcli software profile update -d /vmfs/volumes/<DATASTORE>/HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7-upgrade-bundle.zip -p HX-ESXi-8.0U3-24674464-Cisco-Custom-8.0.3.7

# Reboot and exit maintenance mode
reboot
vim-cmd hostsvc/maintenance_mode_exit
```

**Validation After Each Host:**
- Host summary shows ESXi 8.0 U3
- Host operational in cluster
- No VM placement issues

**Rollback Plan:**
- If ESXi upgrade fails: Rollback to previous version, open TAC case

---

### Step 4 — Post-Validation and Service Restoration (60-90 minutes)

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
  - **UCS Firmware**: Confirm 4.3.5d (FI) and 4.2.3o (servers)

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
| **Step 1** | vCenter upgrade to v8.0.x | 60-90 minutes | - | Stage 1 & 2 upgrade per VMware KB + buffer |
| **Step 2** | UCS Infrastructure (FI A/B) upgrade | 30-60 minutes | - | Sequential FI upgrade with HA preservation |
| **Step 2B** | Subordinate FI upgrade | 20-30 minutes | - | First FI reboot and validation |
| **Step 2D** | Primary FI upgrade | 20-30 minutes | - | Second FI reboot with brief single-homing |
| **Step 3a** | UCS Server Firmware (HUU) rolling | 4-6 hours | 8 nodes × 20-40 min | **Major time component** - sequential node upgrades |
| **Step 3b** | HXDP upgrade to 5.5(2b) | 60-120 minutes | 8 CVMs sequential | Rolling CVM upgrades + validation |
| **Step 3c** | ESXi host upgrade to 8.0 U3 | 6-12 hours | 8 hosts × 45-90 min | **Largest time component** - rolling host upgrades |
| **Step 4** | Post-validation and service restoration | 60-90 minutes | - | Comprehensive testing and service re-enablement |

**Total Estimated Window:** 14-21 hours (conservative estimate for 8-node cluster)

**Critical Path Components:**
- **UCS Server Firmware (HUU)**: 4-6 hours
- **ESXi Host Upgrades**: 6-12 hours  
- **Combined**: 10-18 hours for core rolling upgrades

**Additional Considerations:**
- Network speed affects bundle upload times
- Any troubleshooting adds time
- Can be split across multiple maintenance windows if needed
- Non-production environment allows for extended weekday windows


**Critical Variables Affecting Time:**
- Number of UCS servers (affects HUU duration)
- Number of ESXi hosts (affects host upgrade duration)
- HyperFlex cluster size (affects HXDP upgrade duration)
- Network speed and bundle upload times
- Any unexpected issues requiring troubleshooting


