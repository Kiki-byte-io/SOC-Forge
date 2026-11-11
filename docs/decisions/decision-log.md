# Decision Log

Short records of significant design decisions: **what** was decided, **why**, and **what was rejected**. New decisions are appended, never edited in place. If a decision changes, a new entry supersedes the old one.

| ID | Date | Status |
|---|---|---|
| D-001 – D-011 | 2026-11-11 | Accepted |

---

### D-001 — Wazuh is the only SIEM
- **Decision:** Use Wazuh (manager + indexer + dashboard) as the single SIEM.
- **Why:** Open source, ships both endpoint agents and a detection-rule engine, and has documented single-node sizing. One SIEM keeps the focus on detection engineering rather than tool integration.
- **Rejected:** Running Elastic or Splunk alongside it, which would double the resource cost on a 15.2 GiB host for no research gain.

### D-002 — Isolated VirtualBox lab
- **Decision:** Build the lab as VirtualBox VMs on the owner's laptop.
- **Why:** VirtualBox is already installed and familiar, costs nothing, and supports snapshots for rollback.
- **Rejected:** Cloud hosting (cost, and exposure risk for management interfaces).

### D-003 — Option B: Wazuh + one endpoint at a time
- **Decision:** Run the Wazuh VM (8 GiB) with either the Windows or the Linux endpoint, not both.
- **Why:** All three at recommended sizes need ~16–18 GiB, more than the host has. No initial scenario needs both endpoints running together.
- **Rejected:** Option A (all three with Wazuh below recommended RAM). It may be revisited later as a resource experiment.

### D-004 — All SOCForge VMs on D:
- **Decision:** Store VMs in `D:\SOCForge-VMs\`. The VirtualBox default machine folder was moved off C:.
- **Why:** C: has very little free space; D: has ~124 GB.
- **Rejected:** C: (space), K: as primary (only ~24 GB).

### D-005 — New VMs only; existing VMs untouched
- **Decision:** Build fresh VMs and do not reuse the existing Kali, HoneyWatch or older Ubuntu VMs.
- **Why:** Keeps SOCForge separate from other projects, gives a clean reproducible baseline, and avoids an end-of-life OS.

### D-006 — Windows endpoint: Windows Server 2025 Evaluation
- **Decision:** 2 vCPU, 4 GiB RAM, 50 GB SATA disk.
- **Why:** The 180-day trial covers the project; it is lighter than Windows 11 and needs no virtual TPM; Sysmon supports Server 2019+.
- **Rejected:** Windows 11 Enterprise Evaluation (90 days, 64 GB disk minimum, needs TPM/Secure Boot).
- **Note:** It must be activated online within 10 days.

### D-007 — Ubuntu Server 24.04 LTS for the Wazuh VM and the Linux endpoint
- **Why:** It appears on Wazuh's list of recommended operating systems; one distribution for both VMs means one set of conventions to learn.
- **Open:** The exact auth-log source on the endpoint is verified in Phase 3.

### D-008 — Dedicated host-only network 192.168.60.0/24, static IPs
- **Decision:** New host-only network; Wazuh .10, Windows .20, Linux .30, host .1. NAT adapter disconnected by default.
- **Why:** Isolates the lab from the home LAN and from the existing 192.168.56.0/24 lab network. Static IPs keep the agent→manager address stable and make investigations unambiguous.
- **Rejected:** Bridged networking (exposes VMs to the home LAN); reusing 192.168.56.0/24 (shared with HoneyWatch).

### D-009 — Native Wazuh dashboard; no custom SIEM UI
- **Why:** Building a dashboard duplicates Wazuh and inflates scope. A small validation/reporting tool may be considered later, only if it adds distinct academic value.

### D-010 — Diagrams as Mermaid in the repository
- **Why:** Text-based diagrams are version-controlled and rendered by GitHub, so they stay in sync with the docs.
- **Rejected:** External whiteboard tools as the source of truth.

### D-011 — Graphify deferred to ~Phase 4
- **Decision:** Keep Graphify uncommitted, with its hooks disabled, until the repository contains substantial rules, tests and scripts.
- **Why:** The repository is too small to benefit, and the current install is incomplete and needs repair and review first.
