# Lenovo ThinkPad X1 Carbon 6th Gen Optimization Guide

This document outlines various optimizations and hardware-specific fixes for your Arch Linux installation on the Lenovo ThinkPad X1 Carbon 6th Gen, focusing on performance, battery life, and stability.

---

## TLP Configuration for Longevity

The following settings create a balanced AC profile and a power-saving battery profile.

First, apply the main settings by editing `/etc/tlp.conf`:
```bash
# /etc/tlp.conf

# --- AC Profile (Balanced) ---
CPU_SCALING_GOVERNOR_ON_AC=powersave
CPU_ENERGY_PERF_POLICY_ON_AC=balance_performance
CPU_BOOST_ON_AC=1
PCIE_ASPM_ON_AC=default

# --- Battery Profile (Power Save / Longevity) ---
CPU_SCALING_GOVERNOR_ON_BAT=powersave
CPU_ENERGY_PERF_POLICY_ON_BAT=power
CPU_BOOST_ON_BAT=0
PCIE_ASPM_ON_BAT=powersupersave
PLATFORM_PROFILE_ON_BAT=low-power
WIFI_PWR_ON_BAT=on
SOUND_POWER_SAVE_ON_BAT=1
RUNTIME_PM_ON_BAT=auto

# --- General Settings ---
USB_AUTOSUSPEND=0
```

Next, to preserve battery health, set charge thresholds by creating a new file:
```bash
sudo nano /etc/tlp.d/99-battery-thresholds.conf
```
Add the following content to the file:
```
# Start charging at 40%, stop at 80%
START_CHARGE_THRESH_BAT0=40
STOP_CHARGE_THRESH_BAT0=80
```

Finally, apply all TLP settings with:
```bash
sudo tlp start
```

---

## Hardware-Specific Fixes

These are common `modprobe` tweaks for the X1C6.

**WiFi Stability:**
Create `/etc/modprobe.d/iwlwifi.conf` and add:
```
options iwlwifi 11n_disable=8
```

**Audio (Fix for low volume):**
Create `/etc/modprobe.d/alsa-base.conf` and add:
```
options snd-hda-intel model=nofixup
```

---

## SSD Maintenance

Enable periodic TRIM to keep the SSD performance optimal.
```bash
sudo systemctl enable --now fstrim.timer
```

---

## Advanced Fixes & Optimizations (Optional but Recommended)

**Enable Deep Sleep (S3):**
For better power savings during suspend.
```bash
# Check available sleep states (should show [s2idle] and deep)
cat /sys/power/mem_sleep

# Add the kernel parameter to GRUB config
sudo sed -i 's/quiet/quiet mem_sleep_default=deep/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**Improve Bluetooth Stability:**
If Bluetooth is flaky after waking from suspend.
```bash
# Add a kernel parameter to disable BT autosuspend
sudo sed -i 's/quiet/quiet btusb.enable_autosuspend=0/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

# Optimization and Productivity Guide: Lenovo ThinkPad X1 Carbon (Gen 6) on Arch Linux

This guide provides a checklist of tasks to optimize your system for performance, battery life, and productivity.

## 1. Core System & BIOS Configuration

- [ ] **Update BIOS Firmware:** Use the `fwupd` utility to ensure your BIOS is up to date.
  - `sudo fwupdmgr refresh`
  - `sudo fwupdmgr get-updates`
  - `sudo fwupdmgr update`

- [ ] **Check BIOS Settings:** Reboot and enter the BIOS to configure the following for better compatibility and power management.
  - **Config > Power > Sleep State** -> Set to **"Linux"** (Enables S3 deep sleep).
  - **Config > Thunderbolt(TM) 3 > Thunderbolt BIOS Assist Mode** -> Set to **"Enabled"** (Reduces power consumption).
  - **Security > I/O Port Access** -> Disable unused devices like the Fingerprint Reader or WWAN if not needed.

## 2. Power Management

- [ ] **Install and Enable TLP:** This is the primary tool for managing power consumption.
  - `sudo pacman -S tlp tlp-rdw`
  - `sudo systemctl enable --now tlp.service`

- [ ] **Prevent Service Conflicts:** Mask the `systemd-rfkill` service to avoid conflicts with `tlp`.
  - `sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket`

- [ ] **Set Battery Charge Thresholds:** To preserve battery health, configure `tlp` to stop charging at 80%.
  - Edit `/etc/tlp.conf`:
    - `START_CHARGE_THRESH_BAT0=75`
    - `STOP_CHARGE_THRESH_BAT0=80`

- [ ] **(Optional) Disable Memory Card Reader:** The card reader can draw significant power.
  - **Option A (BIOS):** Disable the "Memory Card Slot" in the BIOS I/O Port Access menu.
  - **Option B (Software):** Create a script to unbind the USB driver.

## 3. Performance Tuning

### CPU
- [ ] **Install Thermal Daemon:** Prevents premature CPU thermal throttling that can occur on this model.
  - `sudo pacman -S thermald`
  - `sudo systemctl enable --now thermald.service`

### SSD
- [ ] **Enable Periodic TRIM:** Maintains SSD performance over time.
  - `sudo systemctl enable --now fstrim.timer`

- [ ] **Set I/O Scheduler (for NVMe):** `none` is the recommended scheduler for modern NVMe SSDs.
  - Create a file: `sudo nano /etc/udev/rules.d/60-ioscheduler.rules`
  - Add the line: `ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"`

### Graphics
- [ ] **Check X11/Wayland Session:** Determine your current display server protocol.
  - `echo $XDG_SESSION_TYPE`

- [ ] **Prevent GPU Hangs (Especially on Wayland):** It is strongly recommended to *disable* GuC/HuC firmware loading to prevent freezes.
  - Create a file: `sudo nano /etc/modprobe.d/i915.conf`
  - Add the line: `options i915 enable_guc=0`

- [ ] **Enable Framebuffer Compression (Power Saving):**
  - Edit `/etc/default/grub` and add `i915.enable_fbc=1` to `GRUB_CMDLINE_LINUX_DEFAULT`.
  - Example: `GRUB_CMDLINE_LINUX_DEFAULT="... quiet i915.enable_fbc=1"`
  - Run `sudo grub-mkconfig -o /boot/grub/grub.cfg` afterwards.

## 4. Hardware-Specific Fixes & Tweaks

- [ ] **Fix Low Speaker Volume:** If you experience quiet speakers, apply a model fix.
  - Create a file: `sudo nano /etc/modprobe.d/alsa-base.conf`
  - Add the line: `options snd-hda-intel model=nofixup`
  - *Note: This may disable the mute button LED.*

- [ ] **Fix Slow WiFi:** If your wireless speeds are poor, try this module option.
  - Create a file: `sudo nano /etc/modprobe.d/iwlwifi.conf`
  - Add the line: `options iwlwifi 11n_disable=8`

- [ ] **(Optional) Disable ThinkPad Lid LED:** Save a tiny amount of power by turning off the blinking red light.
  - Add `ec_sys.write_support=1` to your kernel parameters (in `/etc/default/grub`).
  - Create a systemd service to run the command to turn it off at boot.

## 5. Productivity Enhancements

- [ ] **Install a Clipboard Manager:** Keep a history of your copied items.
  - `sudo pacman -S copyq`

- [ ] **Use an Application Launcher:** Quickly launch applications with the keyboard.
  - Popular options: `rofi`, `dmenu`.

- [ ] **Explore a Tiling Window Manager:** For a keyboard-centric workflow, consider alternatives to a full desktop environment.
  - Popular options: `i3-wm`, `sway` (for Wayland).

## 6. System Maintenance

- [ ] **Clean Package Cache:** Periodically remove old package versions to free up disk space.
  - `sudo paccache -r`

- [ ] **Remove Orphaned Packages:** Uninstall dependencies that are no longer required by any package.
  - `sudo pacman -Rns $(pacman -Qtdq)`

