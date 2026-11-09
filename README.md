# 🏠 Homelab: Networking & Cybersecurity Lab

A small 10" rack homelab I'm building to practice networking, virtualization, and security hands-on after earning **CompTIA Security+**. Everything here runs on used business hardware and free, open-source software.

![Homelab rack](screenshots/rack.jpg)

## Hardware

| Device | Role | Specs |
| --- | --- | --- |
| Dell OptiPlex 7070 Micro (`pve1`) | Main hypervisor | i5-8500T (6 cores), 32 GB DDR4, 512 GB NVMe |
| ASUS laptop (`pbs`) | Backup server | Intel Core i3, 94 GB system disk |
| TP-Link TL-SG108E | Managed switch | 8-port gigabit, VLANs, port mirroring |
| 12-port Cat6 patch panel | Cabling | Ports mapped 1:1 to switch ports |
| ElecVoztile PDU | Power | 125 V / 15 A |

## Network design

```
Internet
   │
ISP gateway (home network)
   │
TL-SG108E switch
   ├── pve1 (Proxmox VE)
   │     ├── vmbr0 ── home network
   │     │     └── pfSense WAN
   │     └── vmbr1 ── isolated lab network (192.168.1.0/24)
   │           └── pfSense LAN → lab VMs / containers
   └── pbs (Proxmox Backup Server)
```

The lab network sits behind its own pfSense firewall, so anything I test there can't reach devices on my home network.

## Projects

| # | Project | Skills |
| --- | --- | --- |
| 01 | [pfSense firewall & lab isolation](projects/01-pfsense-lab-isolation/README.md) | Virtualization, firewalls, network segmentation, log analysis |
| 02 | [Secure backups with Proxmox Backup Server](projects/02-secure-backups/README.md) | Backup & recovery, least privilege, troubleshooting |

## Roadmap

- [x] Proxmox VE on the OptiPlex 7070
- [x] pfSense firewall with an isolated lab network
- [x] Nightly backups to a separate server
- [ ] Jellyfin media server
- [ ] Wazuh SIEM + Kali Linux for attack-and-detect practice
- [ ] Suricata IDS using switch port mirroring
- [ ] VLANs on the managed switch
- [ ] NAS for media, photo backups and file shares

## Certifications

CompTIA A+ · Network+ · Security+ · Google IT Support Professional Certificate
