# 01 · pfSense Firewall & Lab Isolation

## Goal

Build a safe, isolated network for security testing on one Proxmox host. The lab must reach the internet but must **not** be able to reach my home network.

## What I built

- **pfSense 2.8.1** as a virtual machine on Proxmox VE
- Two virtual bridges:
  - `vmbr0`: the home network (pfSense **WAN**)
  - `vmbr1`: a bridge with no physical port (pfSense **LAN**, the isolated lab)
- pfSense handles DHCP, DNS and NAT for the lab (`192.168.1.0/24`)
- A Debian test container (`lab-test`) on `vmbr1` to prove it works
- A logged firewall rule blocking the lab from the home network

## Steps

1. Created `vmbr1` in Proxmox with no bridge ports and no IP.
2. Built the pfSense VM: 2 cores, 2 GB RAM, 16 GB disk, two VirtIO NICs (`vtnet0` = WAN, `vtnet1` = LAN).
3. Installed pfSense on **UFS** instead of ZFS (ZFS wants 8 GB+ RAM; the VM has 2 GB).
4. Gave the WAN a static address so it never changes.
5. Took a Proxmox snapshot, then upgraded pfSense from 2.7.2 to 2.8.1.
6. Created a Debian 13 container on `vmbr1` and confirmed it received a lab IP and reached the internet.
7. Added a LAN rule at the top of the list:

| Action | Protocol | Source | Destination | Log |
| --- | --- | --- | --- | --- |
| Block | Any | LAN subnets | Home network /24 | ✅ |

## Verification

From the lab container:

```bash
ping -c 3 <home-server-ip>   # blocked ✅
ping -c 3 1.1.1.1            # works ✅
```

The pfSense firewall log shows each blocked ping tagged with my rule, **"block lab to home network"**, protocol ICMP.

![Firewall log](../../screenshots/pfsense-firewall-log.jpg)

## Problems & fixes

| Problem | Cause | Fix |
| --- | --- | --- |
| Locked out of the pfSense web page | Applying a change reloaded the firewall before an allow rule existed | Disabled the firewall from the console with `pfctl -d`, then added a WAN rule allowing HTTPS from the home subnet only |
| pfSense showed "up to date" on 2.7.2 | Branch dropdown was set to the previous stable version | Switched the branch to the latest stable release |
| Container failed with `Exec format error` | Corrupted template download | Deleted the container and template, re-downloaded the amd64 Debian template |
| Constant "Gateway alarm" log entries | ISP gateway rate-limits ping monitoring | Pointed gateway monitoring at a public DNS server |

## Reading the logs

While checking the firewall log, I also identified normal home network traffic that pfSense was dropping on WAN:

| Port | Traffic | Likely source |
| --- | --- | --- |
| 5353/UDP | mDNS | Phones, printers, smart devices |
| 1900/UDP | SSDP/UPnP | Smart TVs and streaming devices |
| 57621/UDP | Spotify Connect discovery | Spotify app looking for speakers |

## What I learned

- Firewall rules are read top to bottom, so order matters.
- Always create your allow rule **before** applying changes that reload the firewall.
- Snapshots before upgrades make mistakes cheap to undo.
- Identifying traffic by port is a core SOC skill.

