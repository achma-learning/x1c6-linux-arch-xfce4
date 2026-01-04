# Lenovo ThinkPad X1 Carbon 6th Gen Optimization Guide

This document outlines various optimizations and hardware-specific fixes for your Arch Linux installation on the Lenovo ThinkPad X1 Carbon 6th Gen, focusing on performance, battery life, and stability.

---

# Optimization and Productivity Guide: Lenovo ThinkPad X1 Carbon (Gen 6) on Arch Linux

This guide provides a checklist of tasks to optimize your system for performance, battery life, and productivity.

## 1. Core System & BIOS Configuration

- [ ] **Update BIOS Firmware:** Use the `fwupd` utility to ensure your BIOS (and other firmware like Thunderbolt controller) is up to date.
  - **BIOS Settings for Updates:** In the BIOS setup menu under `Security -> UEFI BIOS Update Option`, ensure both `Flash BIOS Updating by End-Users` and `Windows UEFI Firmware Update` are enabled.
  - **Automatic Update:**
    - `sudo fwupdmgr refresh`
    - `sudo fwupdmgr get-updates`
    - `sudo fwupdmgr update`
  - **Manual Update (`.cab` file):**
    - Download the latest `.cab` file from the [Lenovo support website](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-6th-gen-type-20kh-20kg/downloads/driver-list).
    - `sudo fwupdmgr install /path/to/your/downloaded.cab`
    - Restart the system to apply the update.

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
- [ ] **Install Thermal Daemon (`thermald` or `throttled`):** Prevents premature CPU thermal throttling that can occur on this model.
  - Choose one:
    - **`thermald` (Recommended for general use):**
      - `sudo pacman -S thermald`
      - `sudo systemctl enable --now thermald.service`
    - **`throttled` (Advanced, includes undervolting):**
      - Install from AUR (e.g., using `paru`): `paru -S throttled`
      - `sudo systemctl enable --now throttled.service`
      - *Note: If using `throttled`, consider disabling `thermald` to avoid conflicts.*

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

- [ ] **Fix Microphone Distortion:** If your microphone volume increases automatically and distorts the sound, disable mic boost.
  - Open `pavucontrol` (PulseAudio Volume Control) and navigate to the "Input Devices" tab.
  - Unlock the channels and reduce the "Microphone Boost" level to 0dB, or adjust to a suitable level.

- [ ] **Suspend Fails with USB-C Devices:** If the system immediately resumes from suspend when devices are plugged into USB-C, it might be due to USB devices waking the computer.
  - Check if `grep XHC /proc/acpi/wakeup` shows `enabled`.
  - If so, disable XHC wakeup: `echo XHC > /proc/bus/usb/drivers/usb/unbind` (or `/proc/acpi/wakeup`)
  - To make this permanent, create a systemd service:
    `/etc/systemd/system/disable-xhc-wakeup.service`
    ```
    Description=Disable XHC wakeup to fix suspend issues
    [Service]
    ExecStart=/bin/bash -c 'grep --silent "^XHC.*disabled" /proc/acpi/wakeup || echo XHC > /proc/acpi/wakeup'
    Type=oneshot
    RemainAfterExit=yes
    [Install]
    WantedBy=multi-user.target
    ```
  - Enable and start the service: `sudo systemctl enable --now disable-xhc-wakeup.service`

- [ ] **Fix Slow WiFi:** If your wireless speeds are poor, try this module option.
  - Create a file: `sudo nano /etc/modprobe.d/iwlwifi.conf`
  - Add the line: `options iwlwifi 11n_disable=8`

- [ ] **TrackPoint and Touchpad Issues:** Address issues with simultaneous operation and dead trackpads.
  - **Simultaneous Operation:** To get the TrackPoint and Touchpad to work at the same time, add `synaptics_intertouch=1` to the `psmouse` kernel module options.
    - Create a file: `/etc/modprobe.d/psmouse.conf`
    - Add the line: `options psmouse synaptics_intertouch=1`
  - **Reconnecting Dead Trackpad:** If the trackpad stops responding after suspend, use these commands:
    ```bash
    printf none > /sys/bus/serio/devices/serio1/drvctl
    printf reconnect > /sys/bus/serio/devices/serio1/drvctl
    ```

- [ ] **Fingerprint Reader:** Enable the integrated fingerprint reader.
  - Install `fprintd` and `python-validity` (from AUR).
    - `sudo pacman -S fprintd`
    - `paru -S python-validity` (assuming `paru` AUR helper is installed)
  - Configure `fprintd` to enroll fingerprints.
    - `fprintd-enroll`

- [ ] **(Optional) Disable ThinkPad Lid LED:** Save a tiny amount of power by turning off the blinking red light in the ThinkPad logo on the cover.
  - **Enable EC Write Support:** Add the kernel parameter `ec_sys.write_support=1` to your GRUB configuration (`/etc/default/grub`).
    - Example: `GRUB_CMDLINE_LINUX_DEFAULT="... quiet ec_sys.write_support=1"`
    - Run `sudo grub-mkconfig -o /boot/grub/grub.cfg` afterwards.
  - **Create Systemd Service:** Create a service to turn off the LED at boot.
    `/etc/systemd/system/disable-lid-led.service`
    ```
    Description=Disabling ThinkPad Lid LED
    [Service]
    ExecStart=/bin/bash -c "printf '\\x0a' | dd of=/sys/kernel/debug/ec/ec0/io bs=1 seek=12 count=1 conv=notrunc 2> /dev/null"
    [Install]
    WantedBy=multi-user.target
    ```
  - Enable and start the service: `sudo systemctl enable --now disable-lid-led.service`

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

## 7. Hardware Warnings

- [ ] **NVMe Disk Failure:** There is a known issue with the NVMe disk installed in some ThinkPad X1 Carbon (Gen 6) models that can result in device failure. It is recommended to check for and apply firmware updates or contact Lenovo support for a replacement if you experience issues.

## 8. Thunderbolt Dock Configuration

For users with Thunderbolt docks, specific configurations might be needed to ensure stability and proper functionality.

### Plugable USB-C Mini Docking Station with 85W Power Delivery (UD-CAM)

If you experience random disconnections (external monitor, Bluetooth, Ethernet), check the following:

- **BIOS Settings:**
  - `Wake by Thunderbolt`: `Enabled`
  - `Security Level`: `No Security`
  - `Pre-boot ACL option`: `Enabled`

- **TLP Blacklisting from USB autosuspend:**
  - Edit `/etc/tlp.conf` and exclude dock devices from USB autosuspend. You'll need to find the correct Vendor:Product IDs for your dock (e.g., `lsusb`).
  - Example: `USB_DENYLIST=="0000:1111 2222:3333 4444:5555"`

### Lenovo Dock

Some problems can be caused by outdated dock firmware. Updates are typically not supplied by LVFS; use "Firmware for Windows" from the dock support page.

## 3) Graphics Stack

Intel graphics on X1 Carbon Gen6 prefer modesetting:

```bash
sudo pacman -S --needed mesa vulkan-intel

```

Optional (can help with older apps but not required):

```bash
sudo pacman -S xf86-video-intel

```

Ensure Xorg basics:

```bash
sudo pacman -S --needed xorg-server xf86-input-libinput

```

---

### Enable Laptop-Specific Kernel Modules

#### ThinkPad ACPI support:
```bash
sudo pacman -S acpi acpi_call
```

Load ThinkPad module:
```bash
sudo modprobe thinkpad_acpi
```

Confirm:
```bash
ls /proc/acpi/ibm
```

---

### Improve SSD Stability & Lifespan

#### Enable periodic TRIM:
```bash
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

Check:
```bash
systemctl status fstrim.timer
```

---

### Basic Stability Check
```bash
journalctl -p 3 -xb
```

## Optional TrackPoint / Touchpad Tuning

Install:

```bash
sudo pacman -S --needed xf86-input-synaptics
```

Then configure:

```bash
sudo nano /etc/X11/xorg.conf.d/90-trackpoint.conf
```

Example:

```
Section "InputClass"
    Identifier "TrackPoint"
    MatchProduct "DualPoint Stick"
    Option "AccelerationProfile" "2"
    Option "AccelerationScheme" "predictable"
    Option "Sensitivity" "200"
EndSection
```
(Shows only serious errors)

---

## Sources

- [Lenovo ThinkPad X1 Carbon (Gen 6) - ArchWiki](https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_6))