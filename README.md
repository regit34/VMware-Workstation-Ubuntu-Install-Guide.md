# VMware-Workstation-Ubuntu-Install-Guide.md
VMware Workstation 17.5.2 installation guide on Ubuntu 22.04 with kernel 6.x — includes real error fixes and patch steps. BYOL.
# VMware Workstation 17.5.2 Installation on Ubuntu 22.04 (Linux)
## Complete Guide + Troubleshooting

---

## System Info
- **Machine:** Dell
- **OS:** Ubuntu 22.04 LTS (Jammy)
- **Kernel:** 6.8.0-124-generic
- **VMware Version:** Workstation 17.5.2

---

## PRE-REQUISITES

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential gcc git linux-headers-$(uname -r) -y
```

---

## STEP 1 — Download VMware Workstation

Download from Broadcom portal:
`VMware-Workstation-Full-17.5.2-23775571.x86_64.bundle`

---

## STEP 2 — Install VMware Workstation

```bash
cd ~/Downloads
chmod +x VMware-Workstation-Full-17.5.2-23775571.x86_64.bundle
sudo bash VMware-Workstation-Full-17.5.2-23775571.x86_64.bundle
```

Wait for `Installation was successful.`

---

## STEP 3 — Run vmware-modconfig

```bash
sudo vmware-modconfig --console --install-all
```

---

## ⚠️ ERROR 1 — GCC Version Mismatch

**Error message:**
```
Failed to get gcc information
```

**Popup says:**
```
GNU C Compiler (gcc) version 12.3.0 was not found
```

**Why it happens:**
Ubuntu 22.04 ships with gcc-11 by default. VMware 17.5.2 expects gcc-12.

**Fix:**
```bash
sudo apt install gcc-12
sudo ln -sf /usr/bin/gcc-12 /usr/bin/gcc
gcc --version   # Should show 12.x
sudo vmware-modconfig --console --install-all
```

---

## ⚠️ ERROR 2 — Kernel Module Compilation Failed

**Error message (GUI popup):**
```
Unable to install all modules.
See log /tmp/vmware-user/vmware-9859.log for details. (Exit code 1)
```

**Check the log:**
```bash
cat /tmp/vmware-user/vmware-9859.log | tail -50
```

**What log shows:**
```
Skipping BTF generation for vmnet.ko due to unavailability of vmlinux
Unable to install all modules. See log for details.
```

**Why it happens:**
VMware 17.5.2 kernel modules are not compatible with Linux kernel 6.x out of the box. The `vmlinux` BTF debug file is missing, causing vmnet.ko compilation to fail.

---

## STEP 4 — Fix Using vmware-host-modules Patch

```bash
cd /tmp
git clone https://github.com/mkubecek/vmware-host-modules.git
cd vmware-host-modules
```

---

## ⚠️ ERROR 3 — Branch Not Found

**Error message:**
```
error: pathspec 'workstation-17.5.2' did not match any file(s) known to git
```

**Why it happens:**
The repo does not have a branch for 17.5.2 yet. The closest available branch is `workstation-17.5.1` which is fully compatible.

**Check available branches:**

```bash
git branch -a | grep workstation
```

You will see the latest is `workstation-17.5.1` — use that.

---

## STEP 5 — Checkout, Build and Install Patch

```bash
git checkout workstation-17.5.1
sudo make
sudo make install
```

---

## STEP 6 — Load Kernel Modules

```bash
sudo modprobe vmmon
sudo modprobe vmnet
```

---

## STEP 7 — Start VMware Services

```bash
sudo systemctl enable vmware
sudo systemctl start vmware
```

Note: If you see `vmware-networks.service not found` — this is harmless. Networking still works inside VMs.

---

## STEP 8 — Launch VMware

```bash
vmware &
```

VMware Workstation 17 GUI will open with the setup wizard. ✅

---

## HARMLESS WARNINGS (Can be ignored)

| Warning | Meaning |
|---|---|
| `Use shipped Linux kernel AIO access library` | VMware uses its own libaio — fine |
| `GLib does not have GSettings support` | UI cosmetic only — no impact |
| `I/O warning: failed to load external entity proxy.xml` | Optional config file missing — harmless |
| `vmware-networks.service not found` | VM networking still works fine |

---

## QUICK REFERENCE — All Commands in Order

```bash
# 1. Install dependencies
sudo apt update
sudo apt install build-essential gcc-12 git linux-headers-$(uname -r) -y

# 2. Fix gcc version
sudo ln -sf /usr/bin/gcc-12 /usr/bin/gcc

# 3. Install VMware
sudo bash VMware-Workstation-Full-17.5.2-23775571.x86_64.bundle

# 4. Clone patch repo
cd /tmp
git clone https://github.com/mkubecek/vmware-host-modules.git
cd vmware-host-modules
git checkout workstation-17.5.1

# 5. Build and install patch
sudo make
sudo make install

# 6. Load modules
sudo modprobe vmmon
sudo modprobe vmnet

# 7. Start services
sudo systemctl enable vmware && sudo systemctl start vmware

# 8. Launch
vmware &
```

---

## Environment Summary

| Item | Detail |
|---|---|
| Host OS | Ubuntu 22.04 LTS |
| Kernel | 6.8.0-124-generic |
| GCC used | gcc-12 |
| Patch branch | workstation-17.5.1 |
| VMware version | Workstation Pro 17.5.2 |
| Status | ✅ Working |

---

