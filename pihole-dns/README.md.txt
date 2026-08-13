# Pi-hole (Network-wide DNS Ad Blocking)

My first real service running on the homelab: a network-wide ad and tracker blocker running in an LXC container.

## What it is

Pi-hole works as a DNS server. Instead of blocking ads by modifying web pages (like a browser extension does), it blocks them at the DNS level, refusing to resolve known ad and tracker domains before your device can even connect to them. Any device configured to use it as DNS gets ad blocking automatically, with no extension installed.

## Setup

- Created an LXC container in Proxmox (1 vCPU, 512MB RAM, 8GB disk) using a Debian template
- Installed Pi-hole using the official install script:

```bash
curl -sSL https://install.pi-hole.net | bash
```

- Set the container's IP as static (`192.168.1.91`), since a DNS server needs a fixed address to be reliable
- Enabled "Start at boot" so the container comes up automatically with the host

## Testing it

Configured my PC's network adapter to use `192.168.1.91` as its DNS server. Confirmed it was working by checking Pi-hole's client list, and watching blocked queries show up in real time in the query log while browsing.

## What I learned

- The difference between LXC containers and full VMs (shared kernel vs. full isolation), and why LXC is a better fit for lightweight single-purpose services
- How DNS-level blocking differs from browser-based ad blockers (harder to detect, since nothing runs on the page itself)
- What `curl | bash` actually does under the hood (downloading a script and piping it directly into an interpreter), and the security tradeoff of running installers this way versus downloading and reviewing the script first
- Why Pi-hole doesn't block everything: it can't distinguish first-party ads (like YouTube's own ads served from youtube.com) from content, since both come from the same domain

## Next steps

- Set up Pi-hole as the DNS server for the whole network (currently only testing on one device)
- Explore DNS over HTTPS for the upstream resolver