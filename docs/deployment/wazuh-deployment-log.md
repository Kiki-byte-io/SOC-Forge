# Wazuh Deployment Log (Phase 2)

Running record of every change made to build the Wazuh VM, so the lab can be rebuilt identically. Each step was approved by the owner before it was carried out.

| Step | Date | What | Status |
|---|---|---|---|
| 1 | 2026-11-12 | Download Ubuntu Server ISO | ✅ Done |
| 2 | 2026-11-12 | Create SOCForge host-only network | ✅ Done |
| 3 | 2026-11-12 | Create `socf-wazuh` VM | ✅ Done |
| 4 | — | Install Ubuntu Server | ⏳ Next |
| 5 | — | Install Wazuh all-in-one | — |
| 6 | — | Health check + dashboard access | — |
| 7 | — | Snapshot | — |

---

## Step 1 — Ubuntu Server ISO

- **File:** `ubuntu-24.04.5-live-server-amd64.iso` (4,080,486,400 bytes)
- **Source:** https://releases.ubuntu.com/24.04/
- **Stored at:** `D:\SOCForge-VMs\ISO\`
- **Integrity:** SHA256 `97f3d7ffb032c3eb3b23d2c8be9cc76e60c2c1f2c0146ba5ba9fe01cafae0fd8`, which **matches** the official `SHA256SUMS` file.
- **Note:** the campus network blocked certificate-revocation servers, so curl needed `--ssl-revoke-best-effort`. Integrity was confirmed by the checksum match.

## Step 2 — Host-only network

Created through VirtualBox → File → Tools → Network Manager → Host-only Networks → Create.

| Setting | Value |
|---|---|
| Adapter | `VirtualBox Host-Only Ethernet Adapter #2` |
| Host IP | 192.168.60.1 |
| Mask | 255.255.255.0 |
| DHCP server | Disabled |

The existing `192.168.56.0/24` adapter (used by other projects) was not modified.

**Equivalent CLI** (admin prompt expected on Windows):
```
VBoxManage hostonlyif create
VBoxManage hostonlyif ipconfig "VirtualBox Host-Only Ethernet Adapter #2" --ip 192.168.60.1 --netmask 255.255.255.0
```
**Rollback:** Network Manager → select adapter → Remove.

## Step 3 — `socf-wazuh` VM

Created through the VirtualBox "New Virtual Machine" wizard. Unattended installation was **disabled** so the OS install is done by hand.

| Setting | Value |
|---|---|
| OS type | Ubuntu 24.04 LTS (64-bit) |
| Config file | `D:\SOCForge-VMs\socf-wazuh\socf-wazuh.vbox` |
| vCPU / RAM | 4 / 8192 MB |
| Firmware | BIOS |
| Disk | `socf-wazuh.vdi`, 50 GB dynamically allocated, SATA port 0 |
| Optical | Ubuntu ISO on IDE primary |
| NIC 1 | Host-only → `VirtualBox Host-Only Ethernet Adapter #2` |
| NIC 2 | NAT (**connected only during Phase 2 installs**) |
| Audio | Off |

**Equivalent CLI** (for rebuilding):
```
VBoxManage createvm --name socf-wazuh --ostype Ubuntu24_LTS_64 --basefolder "D:\SOCForge-VMs" --register
VBoxManage modifyvm socf-wazuh --cpus 4 --memory 8192 --audio-driver none --usb off \
  --nic1 hostonly --hostonlyadapter1 "VirtualBox Host-Only Ethernet Adapter #2" --nic2 nat
VBoxManage createmedium disk --filename "D:\SOCForge-VMs\socf-wazuh\socf-wazuh.vdi" --size 51200 --variant Standard
VBoxManage storagectl socf-wazuh --name SATA --add sata --controller IntelAhci
VBoxManage storageattach socf-wazuh --storagectl SATA --port 0 --device 0 --type hdd --medium "D:\SOCForge-VMs\socf-wazuh\socf-wazuh.vdi"
VBoxManage storagectl socf-wazuh --name IDE --add ide
VBoxManage storageattach socf-wazuh --storagectl IDE --port 0 --device 0 --type dvddrive --medium "D:\SOCForge-VMs\ISO\ubuntu-24.04.5-live-server-amd64.iso"
```
These CLI equivalents were not executed. The `--ostype` identifier should be confirmed with `VBoxManage list ostypes` before reuse.

**Rollback:** VirtualBox → right-click `socf-wazuh` → Remove → Delete all files.
