# Arch Linux Installation & Post-Install Troubleshooting Guide

A highly technical log detailing the comprehensive `archinstall` configuration parameters, wireless/hardware failures, resolution procedures, and shell environments.

---

## 1. Pre-Installation Network Setup

Before executing the installer, wireless connectivity must be established manually via the internet wireless daemon (`iwd`) command-line interface.

```bash
# Launch the interactive wireless control utility
iwctl

# List available network interfaces (typically wlan0)
device list

# Scan for available access points
station wlan0 scan

# Display discovered networks
station wlan0 get-networks

# Connect to the target Access Point
station wlan0 connect "WiFi-Name"

# Exit the utility
exit

# Verify network stack routing via ICMP echo requests
ping -c 4 google.com

```

---

## 2. Complete Archinstall Configuration Profile

Execute the installer:

```bash
archinstall

```

To prevent post-install driver or boot failures, the interactive prompts must be configured as follows:

| Menu Option | Selection / Configuration | Rationale |
| --- | --- | --- |
| **Language & Keyboard** | `en_US` / `us` | Standard layout mapping. |
| **Mirrors** | Region: `Selected Region` | Restricts package fetching to local high-throughput mirrors. |
| **Disk configuration** | Choose Target Drive -> Partition Strategy | Wipe drive and create partition layout. |
| **Filesystem type** | `Btrfs` | Chosen over `Ext4` for copy-on-write integrity, subvolumes, and native snapshotting. |
| **Disk encryption** | `Optional` | LUKS container encryption setup. |
| **Bootloader** | `systemd-boot` or `GRUB` | Core EFI boot executable placement. |
| **Unified Kernel Image** | `No` (Default) | Unless specifically required for Secure Boot chains. |
| **Hostname** | Define system name (e.g., `arch-pc`) | Network identifier string. |
| **Root password** | Specify alpha-numeric string | System administrator execution gate. |
| **User account** | Add a user -> Define username & password -> **Sudo: Yes** | Standard user provisioning with elevation privileges. |
| **Profile** | Type: `Desktop` -> DE: `KDE Plasma` | Installs the core Plasma interface. |
| **Graphics driver** | Select vendor driver (e.g., `AMD`, `Intel`, `NVIDIA Open/Proprietary`) | Prevents display server black screens on first boot. |
| **Audio** | `PipeWire` | Modern, low-latency sound server architecture. |
| **Kernels** | `linux` (and optionally `linux-lts` for fallback stability) | Base operating system kernel. |
| **Additional packages** | `git`, `vim`, `neovim`, `base-devel` | Base development tools missing from default profiles. |
| **Network configuration** | Select `NetworkManager` | Installs user-space network control tools. |
| **Timezone** | Select local region (e.g., `Asia/Dhaka`) | Synchronizes hardware clock (`hwclock`). |
| **Automatic NTP** | `True` | Network Time Protocol synchronization. |

---

## 3. Post-Install Troubleshooting: Network & Bluetooth Subsystem Failures

### Symptom

Upon completing the installation and rebooting into the KDE Plasma desktop environment:

* No Wi-Fi or Bluetooth applet icons appear in the system tray.
* Running network commands reveals interfaces exist (`ip link`), but they sit in a `DOWN` state.
* Hardware is recognized by the kernel (`lspci`, `lsusb`), but remains unmanaged.

### Root Cause

1. **NetworkManager Service State:** Even if `NetworkManager` was selected during `archinstall`, the backend daemon service is frequently left disabled or unstarted in the systemd configuration.
2. **Missing UI Meta-Packages:** The `archinstall` profile pulls down the core desktop components but often misses the desktop-agnostic system tray applets (`network-manager-applet`) and full hardware middleware dependencies (`bluez`, `bluez-utils`).

### Resolution Procedure

> **Prerequisite:** If no local networking exists, connect an Android/iOS device via USB and enable **USB Tethering**, or connect a physical Ethernet cable. The kernel will automatically mount this interface as a wired device (e.g., `enp0s20u2`), restoring temporary package access.

#### Part 1: Network Stack Restoration

```bash
# Explicitly sync databases and install the network manager plus its system tray wrapper
sudo pacman -Sy networkmanager network-manager-applet

# Force systemd to register the service and execute it immediately
sudo systemctl enable --now NetworkManager

```

*Effect: The network daemon takes control of the wireless interface, and the Wi-Fi icon initializes in the KDE Plasma system tray.*

#### Part 2: Bluetooth Subsystem Initialization

```bash
# Fetch the Linux Bluetooth protocol stack and CLI controller binaries
sudo pacman -S bluez bluez-utils

# Force systemd to register and start the hardware Bluetooth service
sudo systemctl enable --now bluetooth

```

*Effect: KDE's Bluez integration layer automatically hooks into the active service, rendering the status applet in the tray.*

---

## 4. Shell Environment Configuration

Append these explicit blocks to the shell configuration profile (`~/.bashrc` or `~/.zshrc`).

### Prompt Definition (PS1)

Overrides the default multi-variable prompt string with an optimized, lightweight terminal interface layout.

```bash
# Standard system fallback definition:
# PS1='[\u@\h \W]\$ '

# Production layout: Bold White directory path followed by a Green arrow indicator
export PS1="\[\e[1;37m\]\W \[\e[1;32m\]❯ \[\e[0m\]"

```

### Environment Path Expansion

Instructs the shell runtime to index the user-space local binary directories prior to scanning system-wide binary paths.

```bash
export PATH="$HOME/.local/bin:$PATH"

```

### Static Execution Aliases

Abstracts remote execution payloads into local, deterministic macro aliases.

```bash
# Fetches, inspects, and executes the client-side installer script for Vencord
alias vencord='sh -c "$(curl -sS https://vencord.dev/install.sh)"'
