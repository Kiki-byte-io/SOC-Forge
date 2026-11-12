# Wazuh Deployment Log (Phase 2)

Running record of every change made to build the Wazuh VM, so the lab can be rebuilt identically. Each step was approved by the owner before it was carried out.

| Step | Date | What | Status |
|---|---|---|---|
| 1 | 2026-11-12 | Download Ubuntu Server ISO | ✅ Done |
| 2 | 2026-11-12 | Create SOCForge host-only network | ✅ Done |
| 3 | 2026-11-12 | Create `socf-wazuh` VM | ✅ Done |
| 4 | 2026-11-12 | Install Ubuntu Server | ✅ Done |
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

### Issue observed during Step 3/4 — host memory commit limit

The first VM start failed (`VERR_UNRESOLVED_ERROR`). `VBox.log` showed Windows error **1455** (paging file too small) while registering guest RAM. With the Windows hypervisor active, VirtualBox must commit all guest RAM (8 GiB) up front. At the time, the host had ~1.1 GiB of RAM free and ~7.4 GiB of commit headroom. Windows then auto-grew `C:\pagefile.sys` to ~22.8 GB, leaving C: with ~0.1 GB free.

- **Resolution:** reboot (page file shrank back, C: ~12.7 GB free), close heavy desktop apps before starting the VM, and plan to move the page file to D:.
- **Relevance:** this confirms the RAM constraint predicted in the Phase 0 inventory, and is recorded as a resource limitation for the evaluation.

## Step 4 — Ubuntu Server 24.04.5 install

Manual install (no unattended setup).

| Choice | Value |
|---|---|
| Install type | Ubuntu Server (not minimized) |
| `enp0s3` (host-only) | Static `192.168.60.10/24`, **no gateway**, no DNS |
| `enp0s8` (NAT) | DHCP (observed `10.0.3.15`) — used for internet during installs |
| Disk | Entire disk, **LVM disabled**, ext4 `/` ≈ 50 GB (observed `/` = 48.91 GB) |
| Hostname | `socf-wazuh` |
| OpenSSH server | Installed |
| Ubuntu Pro / snaps | Skipped / none |

**Verified from the host (2026-11-12):** ICMP to 192.168.60.10 replies in 1–2 ms; TCP 22 open; installer ISO detached (`emptydrive`).
