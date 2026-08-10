# Proxmox VE Setup

How I got Proxmox VE up and running on a homelab server I built from scratch.

## Hardware

- CPU: AMD Ryzen 7 5700G
- RAM: 32GB DDR4 (Corsair)
- Storage: Samsung 990 PRO 1TB NVMe
- Motherboard: Gigabyte B550

## What I did

### 1. Built the bootable installer

Downloaded the Proxmox VE 9.2 ISO and used Rufus to write it to a USB drive. Proxmox's ISO uses a hybrid image format, so Rufus automatically forces "DD Image mode" instead of the standard ISO mode, that's expected behavior, not an error.

### 2. First boot troubleshooting

The system powered on but wouldn't boot from USB even though it showed up as the first boot priority. Turned out to be a corrupted ISO download, verifying the file size and re-downloading fixed it. Also had to enable CSM in BIOS to get the USB's boot entries to show up properly.

### 3. Installed Proxmox VE

Ran through the graphical installer, target disk, timezone, root password, and network config. Used the default DHCP-detected settings during install.

### 4. Set up remote management

Once installed, the server runs headless (no monitor/keyboard needed) — everything is managed through the web UI at `https://<server-ip>:8006`.

### 5. Configured repositories

By default, Proxmox points to the `enterprise` repository, which requires a paid subscription to use. Since this is a homelab, I:
- Disabled the `enterprise` and `ceph` enterprise repos
- Enabled the `pve-no-subscription` repo (the standard choice for non-production use)
- Ran a full system update:
```bash
apt update && apt full-upgrade -y
```

## What I learned

- How hypervisors like Proxmox sit directly on hardware (bare-metal), compared to a hosted VM in something like Azure
- Basic BIOS/UEFI boot configuration and troubleshooting boot media issues
- Why Proxmox's licensing model works (full features either way, the subscription is really about support and repo stability)

## Next steps

- Set a static IP (see [`/networking`](../networking))
- Deploy first LXC container
- Set up a Windows Server VM with Active Directory