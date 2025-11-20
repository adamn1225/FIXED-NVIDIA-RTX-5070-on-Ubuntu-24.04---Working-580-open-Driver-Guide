# FIXED: NVIDIA RTX 5070 on Ubuntu 24.04 / 24.10 — Working 580-open Driver Guide

**Author:** Adam N.

**Why this exists:** No one should suffer through the NVIDIA 50xx driver hell that we did.

This guide documents the **exact steps** required to make the RTX 5070 fully functional on Ubuntu using the correct `nvidia-driver-580-open` package. This resolves black screens, broken boot, missing monitors, nvidia-smi failures, and partial/failed installs.

---

# Problem Summary

The NVIDIA RTX 5070 is part of the 5000-series, which **requires the Open Kernel Modules** and **requires driver 580 or newer**.

All of the following WILL FAIL:

- Installing `nvidia-driver-550`  
- Installing `nvidia-driver-560`  
- Installing `nvidia-driver-570-open`  
- Installing from the graphics-drivers PPA  
- Installing the .run installer  
- Installing mixed 570 + 580 components  
- Booting with Secure Boot enabled  

Symptoms include:

- Black screen with blinking `_`
- Stuck on Wayland with no Xorg option
- Only 1 monitor detected
- `nvidia-smi` says "no devices found"
- Kernel module fails to load
- Impossible dependency loops (`570-open` depends on `580-open`)

---

# Working Solution: Install **nvidia-driver-580-open** Cleanly

> This is the **ONLY fully supported branch** for RTX 5070 cards on Ubuntu as of 2025.

---

## 1. Purge ALL NVIDIA components

```bash
sudo pkill -f nvidia
sudo apt purge 'nvidia-*' '*nvidia*' 'libnvidia*' -y
sudo apt autoremove --purge -y
```

## If apt/dpkg is locked:

```bash
sudo pkill -f apt
sudo pkill -f dpkg
sudo rm -f /var/lib/dpkg/lock-frontend
sudo rm -f /var/lib/dpkg/lock
sudo rm -f /var/cache/apt/archives/lock
sudo dpkg --configure -a
```

## 2. Remove any APT pinning blocking NVIDIA packages

```bash
sudo rm -f /etc/apt/preferences.d/no-580.pref
```
_list everything just to be sure:_

```console
ls /etc/apt/preferences.d/
```

## 3. Update and Install the correct driver

```bash
sudo apt update
sudo apt install -y nvidia-driver-580-open
```

This pulls in:

- Open kernel modules (DKMS)
- Firmware
- Compute libs
- libnvidia-gl
- xserver nvidia driver
- powerd + suspend hooks
- Wayland EGL support

## 4. Reboot

```bash
sudo reboot
```

## 5. verify it works
**Check Nvidia-smi:*

```bash
nvidia-smi
```

expected:

```yaml
Driver Version: 580.82.07
CUDA Version: 13.0
GPU: NVIDIA GeForce RTX 5070
```

**check Kernel Modules**

```bash
lsmod | grep nvidia
```

Should show:

- nvidia
- nvidia_uvm
- nvidia modeset
- nvidia_drm

---

# Monitors + Wayland

- Triple-monitor support works out of the box.
- Wayland session works (Xwayland shows in nvidia-smi).

Check session:
```bash
echo $XDG_SESSION_TYPE
```

Should say:

```nginx
wayland
```

# Why This Works
1. RTX 50xx cards require Open Kernel Modules.

2. Ubuntu packages <580 are not compatible with the card.

3. Ubuntu’s packaging for 570-open still depends on 580 driver bits, causing:

    * broken dependency trees
    * partial installs
    * boot failure

4. Installing 580-open directly avoids this completely.

---

Common Failure Cases This Fixes

- Black screen / stuck on boot
- Missing monitors
- Xorg unavailable
- nvidia-smi not working
- DKMS build failures
- PPA version mismatches
- Stuck on Nouveau
- Kernel 6.11 – 6.17 module ABI mismatch

# Contribute

If you run into variations of this issue or have other 50xx success notes, PRs welcome.


