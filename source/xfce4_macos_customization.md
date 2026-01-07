# XFCE4 and macOS-like Customization

This guide provides step-by-step instructions to configure your Arch Linux XFCE4 desktop environment to mimic macOS, based on the provided YouTube video chapters.

---

![macOS-like XFCE4 Desktop](img/2026-01-04_09-22.png)

## 1. Initial Setup

### Install Essential Tools
```bash
sudo pacman -S git lxappearance pamac # pamac if not already installed, for AUR packages
```

## 2. Installing and Configuring Themes, Icons, Cursors, and Fonts

### 2.1 Installing MacTahoe-GTK-Theme
Source: https://github.com/vinceliuice/MacTahoe-gtk-theme

```bash
git clone https://github.com/vinceliuice/MacTahoe-gtk-theme.git
cd MacTahoe-gtk-theme
./install.sh # Run the installation script, follow prompts for options
cd ..
rm -rf MacTahoe-gtk-theme # Clean up
```
**Apply the GTK Theme:**
1.  Open `lxappearance` from the terminal or `Settings Manager` -> `Appearance`.
2.  In the "Widget" tab, select `MacTahoe-Dark` (or preferred MacTahoe variant).

### 2.2 Installing MacTahoe-Icon-Theme
Source: https://github.com/vinceliuice/MacTahoe-icon-theme

```bash
git clone https://github.com/vinceliuice/MacTahoe-icon-icon-theme.git
cd MacTahoe-icon-theme
./install.sh # Run the installation script
cd ..
rm -rf MacTahoe-icon-theme # Clean up
```
**Apply the Icon Theme:**
1.  Open `lxappearance`.
2.  In the "Icon Theme" tab, select `MacTahoe-Dark` (or preferred MacTahoe variant).

### 2.3 Installing WhiteSur-Cursors
Source: https://github.com/vinceliuice/WhiteSur-cursors

```bash
git clone https://github.com/vinceliuice/WhiteSur-cursors.git
cd WhiteSur-cursors
./install.sh # Run the installation script
cd ..
rm -rf WhiteSur-cursors # Clean up
```
**Apply the Cursor Theme:**
11. Open `lxappearance`.
2.  In the "Cursor Theme" tab, select `WhiteSur-Cursors`.
3.  Also, go to `Settings Manager` -> `Mouse and Touchpad` -> `Theme` tab and select `WhiteSur-Cursors`.

### 2.4 Installing Fonts
*Specific fonts are not mentioned, but generally, you would install fonts like San Francisco or similar clean sans-serif fonts.*

```bash
# Example: If using AUR helper like yay (replace pamac build with yay -S if you use yay)
# pamac build ttf-apple-fonts # Or search for specific macOS-like fonts
# pamac build otf-sanfrancisco
```
**Configure Fonts:**
1.  Go to `Settings Manager` -> `Appearance` -> `Fonts` tab.
2.  Adjust "Default Font," "Monospace Font," etc., to your desired macOS-like fonts.
3.  Also, check `Settings Manager` -> `Window Manager` -> `Font` for window title fonts.

## 3. Installing Global Menu

This section specifically uses AUR packages, which require an AUR helper like 'yay'. If 'yay' is not installed, please refer to the "AUR Helpers" section at the end of this document.

```bash
yay -S vala-panel-appmenu-registrar vala-panel-appmenu-xfce --mflags "_build_mate=false _build_vala=false _build_budgie=false"
sudo pacman -S appmenu-gtk-module
```

**After enabling vala-appmenu in XFCE Panel (Panel Preferences -> Items -> Add 'Applications Menu' and 'Appmenu Plugin'):**

```bash
xfconf-query -c xsettings -p /Gtk/ShellShowsMenubar -n -t bool -s true
xfconf-query -c xsettings -p /Gtk/ShellShowsAppmenu -n -t bool -s true
```

## 4. Configuring XFCE Panel | Download Xpple Menu

The video suggests downloading "Xpple Menu" from Pling.com. You will likely need to follow the instructions provided on the Pling page for installation and configuration.

Source: https://www.pling.com/p/1529470/

### Adding Sound Picker and Battery Indicator to Panel

To ensure you have a sound picker and battery indicator, you'll need the respective XFCE panel plugins.

**1. Install necessary plugins (if not already installed):**
```bash
sudo pacman -S xfce4-pulseaudio-plugin xfce4-battery-plugin
```
*Note: If you are using PipeWire instead of PulseAudio, the `xfce4-pulseaudio-plugin` might still work, or you might need an alternative like `xfce4-volumed-pulse` from AUR if available and preferred.*

**2. Add to your XFCE Panel:**
1.  Right-click on your XFCE panel and select `Panel` -> `Panel Preferences...`.
2.  Go to the `Items` tab.
3.  Click the `+` (Add new item) button.
4.  In the dialog, search for and add:
    *   `PulseAudio Plugin` (for sound picker)
    *   `Battery Monitor` (for battery indicator)
5.  You can select each added item in the `Items` list and use the Up/Down arrows to change their position on the panel.
6.  Close the Panel Preferences window. You should now see the sound picker and battery indicator on your panel.

### Adding Network Plugin to Panel

To manage your network connections and see network status on your panel, you'll typically use the NetworkManager Applet.

**1. Install necessary package (if not already installed):**
```bash
sudo pacman -S network-manager-applet
```

**2. Add to your XFCE Panel:**
1.  Right-click on your XFCE panel and select `Panel` -> `Panel Preferences...`.
2.  Go to the `Items` tab.
3.  Click the `+` (Add new item) button.
4.  In the dialog, search for and add:
    *   `NetworkManager Applet`
5.  You can select the added item in the `Items` list and use the Up/Down arrows to change its position on the panel.
6.  Close the Panel Preferences window. You should now see the network icon on your panel, allowing you to manage Wi-Fi, wired connections, etc.

**Troubleshooting NetworkManager Applet not appearing:**

The output `1833` confirms that `nm-applet` is running in the background. This means the NetworkManager Applet itself is active, but it's not appearing or being recognized correctly by your XFCE panel.

Let's consolidate the troubleshooting steps, focusing on getting it to appear on your panel:

1.  **Carefully check the "Add New Items" dialog again, looking for generic indicators:**
    *   Right-click on your XFCE panel and select `Panel` -> `Panel Preferences...`.
    *   Go to the `Items` tab.
    *   Click the `+` (Add new item) button.
    *   Look for anything that sounds like a generic status area where applets might embed themselves, such as:
        *   "Status Notifier Plugin"
        *   "Notification Area"
        *   "Indicator Plugin"
    *   Sometimes, if one of these generic plugins is added, the `nm-applet` will automatically appear within it. Ensure you *don't* have multiple instances of these generic plugins, as that can cause conflicts. If you see more than one "Status Notifier Plugin" or "Notification Area", try removing all but one.

2.  **Restart the XFCE panel (if you haven't already or to re-apply changes):**
    *   Open a terminal and run:
        ```bash
        xfce4-panel --restart
        ```
    *   After the panel restarts, repeat step 1 to see if "NetworkManager Applet" or one of the generic indicators appears.

3.  **Check for an existing system tray or notification area:**
    *   Ensure you have a "Notification Area" or "Status Notifier Plugin" already on your panel. `nm-applet` often embeds itself within one of these if it's not a standalone item. If you have neither, try adding one.

4.  **Consider a full system reboot:**
    *   If the above steps don't work, a full system reboot (`sudo reboot`) can sometimes resolve stubborn graphical environment issues where new applets aren't being picked up correctly.

Please try these steps carefully, focusing on the "Add New Items" dialog and ensuring a proper panel restart. Let me know the outcome.

**General Panel Configuration Steps:**
1.  Right-click on the XFCE panel -> `Panel` -> `Panel Preferences`.
2.  Adjust panel size, position (typically top for macOS feel), and appearance.
3.  Add/remove items to match the video's layout (e.g., Application Menu, Xpple Menu, Indicators, Clock).

## 5. Installing and Configuring Plank Dock

```bash
sudo pacman -S plank
```
**Configuration:**
1.  Run `plank` from the terminal.
2.  Right-click on the Plank dock -> `Preferences` to configure themes, position (bottom), auto-hide behavior, and items.
3.  Add Plank to autostart: `Settings Manager` -> `Session and Startup` -> `Application Autostart` -> `Add`.
    *   Name: `Plank`
    *   Command: `plank`
    *   Comment: `macOS-like dock`

## 6. Installing and Configuring Rofi Launcher

Source for themes: https://github.com/adi1090x/rofi

```bash
sudo pacman -S rofi
```
**Configuration:**
1.  Clone the Rofi themes repository: `git clone https://github.com/adi1090x/rofi.git`
2.  Follow the instructions in the `rofi` repository for installation and applying themes.
3.  Set up a keyboard shortcut for Rofi (e.g., `Alt+Space`) in `Settings Manager` -> `Keyboard` -> `Application Shortcuts`.
    *   Command: `rofi -show drun` (or `rofi -show run` for just commands)

## 7. Installing and Configuring Ulauncher

```bash
yay -S ulauncher
# Or use yay: yay -S ulauncher
```
**Configuration:**
1.  Run Ulauncher once to set it up.
2.  Access preferences by right-clicking on its icon in the system tray or pressing the hotkey (default `Ctrl+Space`).
3.  Configure appearance, extensions, and set it to autostart.

<h2> 8. Installing and Configuring Conky </h2>

```bash
yay -S conky-lua-nv
sudo pacman -S jq curl
```
**Configuration:**
*This usually involves placing `.conkyrc` files in your home directory and configuring Conky to autostart. Refer to the video for specific Conky themes and configurations.*

<h2> 9. Installing and Configuring LightDM-Webkit2-Greeter </h2>

Theme: https://github.com/manilarome/lightdm-webkit2-theme-macos

```bash
sudo pacman -S lightdm-webkit2-greeter
```
**Configuration:**
1.  Edit `sudo nano /etc/lightdm/lightdm.conf` and set `greeter-session=lightdm-webkit2-greeter`.
2.  Clone the theme: `git clone https://github.com/manilarome/lightdm-webkit2-theme-macos.git /usr/share/lightdm-webkit2/themes/macos`
3.  Edit `sudo nano /etc/lightdm/lightdm-webkit2-greeter.conf` and set `webkit_theme=macos`.
4.  Reboot to see the changes.

<h2> 10. Installing and Configuring Picom Compositor </h2>

```bash
yay -S picom-ibhagwan-git
```
**Configuration:**
*Picom requires a configuration file, typically `~/.config/picom/picom.conf`, to enable effects like shadows, transparency, and fading. Refer to the video or Picom documentation for macOS-like settings.*
1.  Add `picom` to autostart: `Settings Manager` -> `Session and Startup` -> `Application Autostart` -> `Add`.
    *   Name: `Picom`
    *   Command: `picom --config ~/.config/picom/picom.conf` (or just `picom` if you put config in default location)
    *   Comment: `Compositor for visual effects`

<h2> 11. Customize Firefox Web Browser </h2>

*The video likely demonstrates specific Firefox themes or CSS modifications. This usually involves installing themes from the Firefox Add-ons store or using `userChrome.css` for advanced customization.*

<h2> 12. Customize XFCE Terminal </h2>

*This involves going into `Terminal` -> `Preferences` and adjusting colors, fonts, transparency, and possibly using a zsh/bash theme with Powerline or similar for prompt customization.*

## Window Management

Here’s what will actually improve your daily experience:

1️⃣ Enable macOS-style window dragging
```bash
xfconf-query -c xfwm4 -p /general/easy_click -s true
```
(Drag windows by clicking anywhere, not just titlebar)

Set shortcuts:

*   Super + Left/Right → tile window
*   Super + Up → maximize
*   Super + Down → restore

(Settings → Window Manager → Keyboard)


<h2> 13. Installing Nautilus File Manager </h2>

```bash
sudo pacman -S nautilus
```
*You might want to set Nautilus as your default file manager in `Settings Manager` -> `Preferred Applications` -> `Utilities` -> `File Manager`.*

<h2> 14. Installing Comice Control Center </h2>

Source: https://github.com/libredeb/comice-control-center

```bash
git clone https://github.com/libredeb/comice-control-center.git
cd comice-control-center
sudo pacman -S util-linux gsettings-desktop-schemas wireless_tools iproute alsa-utils dbus-python
sudo pacman -S python-pip
sudo pip3 install -r requirements.txt
./comice-control-center
```
*Note: Ensure you have `python-pip` installed for `pip3`.*

---

**Final Notes:**

*   **Reboot:** After making significant changes, especially to LightDM or core XFCE components, a reboot might be necessary.
*   **Backups:** Always back up your configuration files (`~/.config/xfce4/`, `~/.config/gtk-3.0/`, etc.) before making major changes.
*   **AUR Helpers:** If `pamac build` is not available, you might need to install `yay` or another AUR helper.
    ```bash
    # Example for yay
    sudo pacman -S --needed base-devel git
    git clone https://aur.archlinux.org/yay.git
    cd yay
    makepkg -si
    ```
    Then replace `pamac build` commands with `yay -S`.
*   **Customization:** The exact look and feel will depend on your personal customization within each application's settings.
