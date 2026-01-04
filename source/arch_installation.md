# Arch Linux Installation Guide for Lenovo ThinkPad X1 Carbon 6th Gen

This guide covers the initial installation of Arch Linux and core system setup for a Lenovo ThinkPad X1 Carbon 6th Gen (20KG).

---

## Phase 0: Base System Installation (From Live Environment)

This section covers the initial installation of the Arch Linux base system onto your target disk. It is assumed you have already partitioned your disk and mounted the root partition to `/mnt`.

```bash
pacstrap -K /mnt linux linux-firmware linux-headers base base-devel nano \
		 networkmanager grub efibootmgr os-prober bash-completion iwd intel-ucode
```

**Tip:** You will need to add your CPU microcode package: for Intel, add `intel-ucode`; for AMD, add `amd-ucode`.

**Explanation of Packages:**

*   **`linux`**: The Linux kernel.
*   **`linux-firmware`**: Firmware files for various hardware devices.
*   **`linux-headers`**: Kernel headers for building modules against the kernel.
*   **`base`**: Essential packages for a minimal Arch Linux system.
*   **`base-devel`**: Development tools for compiling software (make, gcc, etc.).
*   **`nano`**: Simple terminal text editor.
*   **`networkmanager`**: Network management daemon and CLI tools.
*   **`grub`**: Bootloader to start the OS.
*   **`efibootmgr`**: EFI boot manager to configure UEFI boot entries.
*   **`os-prober`**: Detects other OS installations for bootloader configuration.
*   **`bash-completion`**: Bash completions for core commands.
*   **`iwd`**: Wireless daemon for managing Wi-Fi connections.
*   **`intel-ucode`**: Microcode updates for Intel CPUs (essential for X1C6).
*   **`amd-ucode`**: Microcode updates for AMD CPUs (only if applicable, not for X1C6).

---

## Phase 1: Core System Setup & Optimization

These commands should be run *after* booting into your newly installed Arch Linux system.

### 1. BIOS Configuration

Before starting, ensure the following BIOS settings are configured:

- **Security → Secure Boot →** `Disabled`
- **Config → Power → Sleep State →** `Linux`
- **Config → Thunderbolt BIOS Assist Mode →** `Enabled`

### 2. Initial System Update & Essential Packages

First, update the system and install a comprehensive set of packages for a fully featured, stable system.

```bash
# 1. Update all packages
sudo pacman -Syu

# 2. Install developer tools, required for some drivers and packages
sudo pacman -S --needed base-devel

# 3. Install firmware, microcode, and graphics drivers
sudo pacman -S --needed linux-firmware intel-ucode mesa vulkan-intel intel-media-driver

# 4. Install core utilities for power, networking, and hardware interaction
sudo pacman -S --needed tlp tlp-rdw ufw thermald networkmanager network-manager-applet bluez bluez-utils acpi acpi_call

# 5. Install a modern audio stack (PipeWire)
sudo pacman -S --needed pipewire pipewire-pulse wireplumber pavucontrol alsa-utils

# 6. Install essential fonts for wide compatibility
sudo pacman -S --needed noto-fonts noto-fonts-emoji noto-fonts-cjk noto-fonts-extra

# 7. Install common utilities and filesystem support
sudo pacman -S --needed wget curl git xdg-utils gvfs openssh unzip ntfs-3g rsync reflector cups
```

### GRUB Dual Boot (Optional)

*Note: This section is particularly relevant if you are planning to dual boot with Windows 11 IoT LTSC.*

If you intend to dual boot with another operating system, follow these steps to ensure GRUB can detect it. You can skip this if you only plan to run Arch Linux.

```bash
sudo nano /etc/default/grub
```
Open the GRUB configuration file. Find the line `#GRUB_DISABLE_OS_PROBER=true` and change it to `GRUB_DISABLE_OS_PROBER=false`.

```bash
# Uncomment the line by removing the hashtag (#)
GRUB_DISABLE_OS_PROBER=false
```

After installing `intel-ucode`, regenerate the GRUB config to load it:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### 3. Service Configuration

Configure the core services for power management, security, networking, and thermal control.

```bash
# Enable TLP for power management
sudo systemctl enable --now tlp.service

# Mask systemd-rfkill to prevent conflicts with TLP (recommended for ThinkPads)
sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket

# Enable the UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable

# Enable the thermal daemon to prevent overheating
sudo systemctl enable --now thermald.service

# Enable NetworkManager for handling network connections
sudo systemctl enable --now NetworkManager

# Enable Bluetooth service
sudo systemctl enable --now bluetooth.service

# Enable systemd services for time synchronization and DNS resolution
sudo systemctl enable --now systemd-timesyncd
sudo systemctl enable --now systemd-resolved
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

# (Optional) Enable the CUPS printing service
sudo systemctl enable --now cups.service

# (Optional but Recommended) Disable the cpupower service if it was installed, as TLP handles this
sudo systemctl disable --now cpupower.service
```

### 4. Swap File for System Stability

Create a 16GB swap file to prevent the system from running out of memory.

```bash
# Allocate a 16GB file
sudo fallocate -l 16G /swapfile

# Set correct permissions
sudo chmod 600 /swapfile

# Format the file as swap
sudo mkswap /swapfile

# Enable the swap file
sudo swapon /swapfile

# Add to /etc/fstab to make it permanent
echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab
```

### 5. TLP Configuration for Longevity

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

### 6. Hardware-Specific Fixes

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

### 7. SSD Maintenance

Enable periodic TRIM to keep the SSD performance optimal.
```bash
sudo systemctl enable --now fstrim.timer
```

### 8. Advanced Fixes & Optimizations (Optional but Recommended)

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

### 9. Final System Sanity Checks

After applying all settings and rebooting, run these commands to check for errors.

```bash
# Check for any critical system log errors since boot
journalctl -p 3 -xb

# Check if any systemd services have failed to start
systemctl --failed
```