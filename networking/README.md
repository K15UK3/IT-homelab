# Network Troubleshooting

A connectivity issue I ran into after installing Proxmox, and how I diagnosed and fixed it.

## The problem

After installing Proxmox VE, the installer showed a URL to access the web interface (`https://192.168.100.2:8006`). But from another device on the network, the page wouldn't load, and `ping 192.168.100.2` just timed out.

## Diagnosis

Checked the physical stuff first, cable connections, port lights, all fine. So the issue had to be on the network config side.

Logged into Proxmox's local console (`root` login on the monitor connected directly to the server) and ran:

ip a

That showed the bridge interface (`vmbr0`) had a static IP of `192.168.100.2/24`, but my actual home network runs on `192.168.1.x`. That mismatch explained the timeout: the server was configured for a network that didn't exist on my LAN.

**Root cause:** the Proxmox installer was run without an Ethernet cable connected, so it fell back to a default/placeholder static IP instead of pulling real network info.

## Fix

Edited the network config directly:

nano /etc/network/interfaces

First, switched the interface to DHCP temporarily just to confirm the router would hand out a real, working IP:

auto vmbr0
iface vmbr0 inet dhcp
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0

Restarted networking and checked again:

systemctl restart networking
ip a

This time `vmbr0` picked up `192.168.1.97`, a real address on my actual LAN, confirmed working with the web UI.

## Making it permanent

DHCP confirmed the right address, but a server should have a fixed IP, it needs to always be reachable at the same address. Tried reserving that IP from the router's admin panel first, but the UI was unresponsive/broken in the browser. Switched to setting a static IP directly on Proxmox instead, using the same address DHCP had already assigned:

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.97/24
    gateway 192.168.1.254
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0

systemctl restart networking

Confirmed with `ip a`, same IP, but no longer marked `dynamic`, meaning it's fixed regardless of router/DHCP behavior going forward.

## What I learned

- How to read and interpret `ip a` output (interfaces, bridges, dynamic vs. static addressing)
- Editing Debian network config directly (`/etc/network/interfaces`)
- Using DHCP as a diagnostic tool to find the correct network parameters before locking in a static config
- Basic terminal editing with `nano`