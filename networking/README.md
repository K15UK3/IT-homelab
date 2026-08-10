# Network Troubleshooting

A connectivity issue I ran into after installing Proxmox, and how I diagnosed and fixed it.

## The problem

After installing Proxmox VE, the installer showed a URL to access the web interface (`https://192.168.100.2:8006`). But from another device on the network, the page wouldn't load, and `ping 192.168.100.2` just timed out.

## Diagnosis

Checked the physical stuff first, cable connections, port lights, all fine. So the issue had to be on the network config side.

Logged into Proxmox's local console (`root` login on the monitor connected directly to the server) and ran:

```bash
ip a
```

That showed the bridge interface (`vmbr0`) had a static IP of `192.168.100.2/24`, but my actual home network runs on `192.168.1.x`. That mismatch explained the timeout: the server was configured for a network that didn't exist on my LAN.

**Root cause:** the Proxmox installer was run without an Ethernet cable connected, so it fell back to a default/placeholder static IP instead of pulling real network info.

## Fix

Edited the network config directly:

```bash
nano /etc/network/interfaces
```

First, switched the interface to DHCP temporarily just to confirm the router would hand out a real, working IP: