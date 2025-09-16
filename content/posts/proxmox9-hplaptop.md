---
title: "My Journey: Proxmox VE 9 on HP Elitebook 2570p with WiFi - Real Home Lab Experience"
date: 2025-08-12T22:27:24+05:30
draft: true
tags: ["Proxmox", "Home Lab", "HP Elitebook 2570p", "WiFi", "Virtualization"]
categories: ["Tech", "Homelab"]
author: "Kanthi"
---

## Introduction

After months of wanting to build a proper home lab but being constrained by space and budget, I decided to repurpose my old HP Elitebook 2570p that was collecting dust. What started as a "let's see if this works" experiment turned into a surprisingly capable Proxmox VE 9 setup that I've been running for several months now.

The biggest challenge? Making it work entirely over WiFi since I don't have ethernet access where the laptop sits. This post chronicles my actual experience - the stumbling blocks, the victories, and the lessons learned from running Proxmox on a laptop with wireless as the primary network connection.

## Why I Chose the HP Elitebook 2570p

Honestly, the choice was made for me - it's what I had. But after working with it extensively, I've come to appreciate several aspects:

- **Cost:** Free (already owned)
- **Power consumption:** My kill-a-watt meter shows it draws about 45-65W under normal load
- **Form factor:** Fits perfectly on my desk shelf without taking up precious space
- **Built-in UPS:** The battery has saved me from several power blips
- **Surprisingly capable:** For a 2012-era machine, it handles multiple VMs better than expected

## My Hardware Configuration

Here's exactly what I'm working with:

- **Processor:** Intel Core i7-3520M (2.90GHz, 2 cores, 4 threads with HT)
- **RAM:** 16GB DDR3 (upgraded from original 8GB - crucial upgrade!)
- **Storage:** 128GB Samsung SSD (replaced the original 320GB HDD)
- **Network:** Intel Centrino Advanced-N 6205 WiFi card
- **Display:** Permanently closed (headless operation)

The SSD upgrade made a massive difference in performance, and maxing out the RAM to 16GB allows me to run 3-4 VMs comfortably simultaneously.

## The WiFi Challenge - What Actually Happened

Let me be honest: getting WiFi working properly with Proxmox took me three attempts and a lot of forum diving. Here's what I actually encountered:

### Attempt 1: The Naive Approach
I initially tried to configure the WiFi interface directly during installation. Big mistake. The installer couldn't connect to my WPA2 network, and I ended up with a partially configured system.

### Attempt 2: Post-Install WiFi Setup
After a fresh installation (using ethernet temporarily), I tried to simply bridge the WiFi interface. This resulted in connectivity issues and the WiFi dropping every few hours. Clearly, standard bridging doesn't play nice with wireless.

### Attempt 3: The NAT Solution (What Actually Works)
This is my current setup that's been running stable for 4+ months:

## My Working Configuration

After much trial and error, here's the configuration that actually works:

### Network Interface Configuration
```/dev/null/network-interfaces#L1-20
# /etc/network/interfaces - My working config
auto lo
iface lo inet loopback

# WiFi interface - connects to home network
auto wlp3s0
iface wlp3s0 inet dhcp
    wpa-ssid "MyHomeNetwork"
    wpa-psk "MyActualPassword"

# Internal bridge for VMs - NAT setup
auto vmbr0
iface vmbr0 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    post-up echo 1 > /proc/sys/net/ipv4/ip_forward
    post-up iptables -t nat -A POSTROUTING -s '192.168.100.0/24' -o wlp3s0 -j MASQUERADE
    post-up iptables -A FORWARD -i wlp3s0 -o vmbr0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    post-up iptables -A FORWARD -i vmbr0 -o wlp3s0 -j ACCEPT
    post-down iptables -t nat -D POSTROUTING -s '192.168.100.0/24' -o wlp3s0 -j MASQUERADE
    post-down iptables -D FORWARD -i wlp3s0 -o vmbr0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    post-down iptables -D FORWARD -i vmbr0 -o wlp3s0 -j ACCEPT
```

### What I Had to Install
The WiFi drivers worked out of the box, but I needed to install:
```bash
apt update
apt install wpasupplicant
apt install iptables-persistent  # To save NAT rules
```

## Real-World Performance and Limitations

After running this setup for several months, here's what I've learned:

### What Works Well:
- **Ubuntu Server 24.04.3 LTS VMs:** Run smoothly with 2-4GB RAM allocated
- **Alpine Linux containers:** Incredibly lightweight and fast
- **Docker containers:** Work great in LXC containers
- **Web browsing from VMs:** Perfectly usable for development work
- **SSH access:** Rock solid, never had connectivity issues

### Performance Reality Check:
- **WiFi throughput:** I get about 40-60 Mbps to VMs (my internet is 100 Mbps)
- **VM boot times:** Ubuntu 24.04.3 VM boots in about 45 seconds from the SSD
- **Memory pressure:** Becomes noticeable with more than 3 VMs running simultaneously
- **CPU limitations:** The dual-core i7 is the bottleneck for CPU-intensive workloads

### Actual Issues I've Encountered:
1. **WiFi occasionally drops** during heavy network activity (large downloads). Solution: I set up a cron job to restart networking if ping fails.
2. **Heat management:** The laptop gets warm during heavy VM usage. I propped it up for better airflow.
3. **VM network isolation:** VMs can talk to each other by default. Had to add iptables rules for proper isolation.

## My Current VM Setup

Here's what I'm actually running:
- **VM 1:** Ubuntu 24.04.3 LTS with Docker (4GB RAM, 20GB disk) - my main development environment
- **VM 2:** pfSense VM (2GB RAM, 8GB disk) - for learning network security
- **VM 3:** Windows 10 LTSC (6GB RAM, 40GB disk) - occasionally spun up for testing
- **LXC 1:** Alpine Linux with Nginx (512MB RAM) - hosts my internal docs

## Power Management Tweaks That Actually Matter

These settings prevent the laptop from going to sleep and causing VM downtime:

```bash
# Disable lid close action
echo "HandleLidSwitch=ignore" >> /etc/systemd/logind.conf

# Prevent WiFi power management from causing drops
echo 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlp3s0", RUN+="/sbin/iw dev %k set power_save off"' > /etc/udev/rules.d/70-wifi-powersave.rules
```

## Lessons Learned and Real Recommendations

### What I Wish I'd Known Before Starting:
1. **The 128GB SSD is tight.** I'm constantly managing disk space. 256GB would be much more comfortable.
2. **NAT is simpler than bridging** for WiFi setups, despite what most tutorials suggest.
3. **VM snapshots eat disk space fast** on a small SSD. I use them sparingly.
4. **The battery only lasts about 2 hours** under load, so it's really just UPS functionality.

### This Setup is Great For:
- Learning Proxmox and virtualization
- Development environments
- Testing network configurations
- Small web services for personal use

### This Setup is NOT Great For:
- Production workloads
- High-bandwidth applications
- CPU-intensive tasks
- Running more than 4 VMs simultaneously

## Conclusion

Six months in, I'm genuinely impressed with how well this old laptop has performed as a Proxmox host. While it's not going to replace a proper server, it's been perfect for learning and development. The WiFi setup, once properly configured with NAT, has been rock solid.

The key insight is that WiFi bridging is problematic, but NAT works beautifully. Don't fight the limitations - work with them. For anyone with a similar old laptop gathering dust, this is absolutely worth trying.

Total investment: $0 (using existing hardware)
Learning value: Priceless

---

**Key Resources That Actually Helped:**
- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)
- [Debian Wiki: WiFi](https://wiki.debian.org/WiFi)
- [Proxmox Forum WiFi NAT Thread](https://forum.proxmox.com) - Several community solutions
