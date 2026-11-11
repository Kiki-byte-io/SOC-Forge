# Phase 0 — Host & Lab Inventory Report

**Date:** 2026-11-11
**Status:** Complete — read-only inventory; no software installed, no VMs started, stopped or modified.
**Only change made:** VirtualBox default machine folder moved from the system drive to the data drive (reversible; see §6).

> Legend: **[Observed]** = measured on the host · **[Decision]** = owner-approved choice · **[Assumption]** = not yet verified

---

## 1. Host

| Item | Value | Type |
|---|---|---|
| OS | Windows 11 Home, build 26300 | Observed |
| CPU | AMD Ryzen 9 7940HX — 16 cores / 32 threads, hardware virtualization enabled | Observed |
| RAM | 15.2 GiB total; ~2.4 GiB free with normal desktop apps open | Observed |
| Hypervisor layer | Windows hypervisor active (WSL2 in use) — VirtualBox runs on top of it | Observed |
| VirtualBox | 7.2.16 | Observed |

**Key constraint:** RAM, not CPU or disk.

## 2. Storage

| Drive | Size | Free | Role |
|---|---|---|---|
| C: (system) | 326.2 GB | 14.9 GB | Not used for lab data |
| D: (data) | 326.8 GB | 123.9 GB | **All SOCForge VMs** |
| K: (data) | 273.4 GB | 24.4 GB | Overflow only |

Other partitions: EFI/system, Microsoft reserved, two recovery partitions, a 26 GB vendor factory-restore partition. None are touched.

## 3. Existing virtual machines

| VM | Purpose | Location | SOCForge use |
|---|---|---|---|
| Kali VM | Owner's separate home lab | data drive | None |
| Ubuntu VM | **HoneyWatch / Cowrie project** | data drive | **Off-limits** |
| Ubuntu 22.10 VM | Older lab VM (OS end-of-life) | data drive | None |
| Broken registry entry | Points to a missing `.vbox` file | — | None |

**Decision:** SOCForge uses only new VMs, so it stays separate from other projects and starts from a clean, reproducible baseline.

## 4. Networking

| Network | Details | Used by |
|---|---|---|
| Host-only #1 | 192.168.56.0/24, host = .1, DHCP off | Existing VMs (incl. HoneyWatch) |
| Internal network | `intnet` | — |
| NAT | Per-VM | Existing VMs |

**Decision:** create a **new, dedicated host-only network** for SOCForge (subnet chosen in Phase 1). Enable NAT only when a VM needs internet for installation or activation, then disable it.

## 5. Feasibility

### Wazuh requirements
All-in-one (manager + indexer + dashboard), 1–25 agents: **4 vCPU, 8 GiB RAM, 50 GB storage** (90 days of alerts). Supported OS includes Ubuntu 22.04 / 24.04. Current version: 4.14.

### Windows endpoint requirements (Windows Server 2025)
2 GB RAM minimum (4 GB recommended with Desktop Experience), 32 GB storage absolute minimum, boot disk must not use IDE.

### RAM budget

| VM | RAM |
|---|---|
| Wazuh all-in-one | 8 GiB |
| Windows Server 2025 endpoint | 4 GiB |
| Linux endpoint (Ubuntu Server) | 1–2 GiB |
| Host OS reserve | ~3–4 GiB |
| **All at once** | **~16–18 GiB > 15.2 GiB available** |

**Verdict:** an all-in-one Wazuh deployment is feasible, but **not** with every VM at full size at the same time.

### Chosen topology — Option B [Decision]
Wazuh (8 GiB) plus **one endpoint at a time**. None of the initial detection scenarios needs both endpoints running together. Running all three with a reduced Wazuh allocation (Option A) may be tested later as a resource experiment.

### Storage plan [Decision]

| VM | Disk (max, dynamically allocated) | Location |
|---|---|---|
| Wazuh | 50 GB | `D:\SOCForge-VMs\` |
| Windows Server 2025 | 50 GB | `D:\SOCForge-VMs\` |
| Ubuntu Server | 20 GB | `D:\SOCForge-VMs\` (or K: if D: tightens) |

Worst case ≈ 120 GB if every disk fills completely; actual use is expected to be well below that.

### Windows endpoint [Decision]
**Windows Server 2025 Evaluation (180 days).** Chosen over Windows 11 Enterprise Evaluation (90 days) because it is lighter, needs no virtual TPM, and the trial covers the project timeline. It must be activated online within the first 10 days.

## 6. Change log

| Date | Change | Rollback |
|---|---|---|
| 2026-11-11 | VirtualBox default machine folder: system drive → `D:\VirtualBox VMs` | `VBoxManage setproperty machinefolder "<previous path>"` |

## 7. Known limitations & assumptions

- **[Observed]** The Windows hypervisor layer is active because WSL2 is needed for other work. VirtualBox VMs may run slower than they would on bare metal. This is recorded as an environmental limitation.
- **[Assumption]** Wazuh can be capped somewhat below 8 GiB if needed. This is unverified; it is only relevant to Option A.
- **[Assumption]** Ubuntu Server runs comfortably in 1–2 GiB as a lightweight endpoint. To be confirmed in Phase 3.

## 8. Sources

- Wazuh, "Quickstart" — https://documentation.wazuh.com/current/quickstart.html (accessed 2026-11-11)
- Microsoft Learn, "Hardware Requirements for Windows Server" — https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements (accessed 2026-11-11)
- Microsoft Evaluation Center, "Windows Server 2025" — https://www.microsoft.com/en-in/evalcenter/download-windows-server-2025 (accessed 2026-11-11)
