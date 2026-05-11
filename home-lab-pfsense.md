# 🏠 Home Lab — pfSense Firewall & Network Security Setup

> A fully functional home network security lab built from scratch using a Mini PC, managed switch, and wireless access point — configured with pfSense firewall, VPN routing, DNS-based ad/threat blocking, and Dynamic DNS. Built and tested twice: first in a virtual machine, then deployed on real hardware.

---

## 📸 Physical Setup

The lab runs on real hardware installed at home — not a simulation or cloud environment.

**Hardware Used:**
- 🖥️ Mini PC (firewall/router running pfSense)
- 🔀 TP-Link 5-port network switch
- 📡 Ceiling-mounted wireless access point
- 🌐 ISP modem/router (WAN source)
- 🔌 CAT 6 ethernet cables throughout

---

## 🔌 Physical Network Wiring

One of the most practical challenges in building this lab was figuring out the **correct physical wiring** — something most tutorials skip entirely.

### Wiring Topology

```
ISP Modem/Router
       │
       │ CAT6 cable
       ▼
  Mini PC (pfSense)
  ┌────────────────┐
  │ ETH0 = WAN  ◄──┼── From ISP/Router
  │ ETH1 = LAN  ──►┼── To TP-Link Switch
  └────────────────┘
       │
       │ CAT6 cable
       ▼
  TP-Link 5-Port Switch
  ┌────────────────────┐
  │ Port 1 ◄── From pfSense LAN (ETH1)
  │ Port 2 ──► Desktop PC
  │ Port 3 ──► Wireless Access Point
  └────────────────────┘
       │                    │
       ▼                    ▼
  Desktop PC         Access Point
                    (Ceiling mounted)
                    Provides WiFi to
                    all wireless devices
```

### Key Learning — Interface Assignment
The first real challenge was correctly identifying which physical ethernet port maps to WAN and which to LAN inside pfSense:
- **ETH0** → assigned as **WAN** (connects to ISP router)
- **ETH1** → assigned as **LAN** (connects to internal switch)

Getting this wrong means no internet connectivity — required careful testing and reassignment through the pfSense console.

---

## ⚙️ pfSense Installation & Configuration

### Installation Steps
1. Downloaded pfSense 2.8.1-RELEASE ISO
2. Created bootable USB using Rufus/Balena Etcher
3. Booted mini PC from USB and ran the installer
4. Configured disk partitioning and completed base install
5. Assigned network interfaces via console menu (ETH0=WAN, ETH1=LAN)
6. Accessed web GUI via LAN IP from browser

### Interface Configuration Result
After installation, all three interfaces came up successfully:

```
WAN  (wan)  → em0  → DHCP from ISP
LAN  (lan)  → em1  → Static LAN gateway
PIA  (opt1) → VPN tunnel interface (added later)
```

---

## 🌐 DHCP Management

Configured pfSense as the DHCP server for the entire home network:
- Defined IP address range for dynamic allocation to devices
- Set **static DHCP mappings** for specific devices (PC, access point) so they always get the same IP
- This replaces the ISP router's DHCP — pfSense becomes the single source of truth for all IP assignments

---

## 🔁 Port Forwarding

Configured inbound port forwarding rules to allow external access to internal services:
- Created NAT rules in pfSense to forward specific ports from WAN to internal devices
- Useful for hosting services accessible from outside the home network
- Combined with firewall rules to restrict access to only required ports

---

## 🌍 Dynamic DNS — Cloudflare Integration

This was the **most challenging part** of the entire setup — and the part that required the most independent problem solving beyond what any tutorial covered.

### The Challenge
Home internet connections have **dynamic public IPs** — the ISP changes your IP periodically. Without DDNS, any domain pointing to your home network breaks every time the IP changes.

### Why It Was Difficult
Most tutorials show the pfSense DDNS settings screen but skip the full Cloudflare setup. The actual process required:

1. **Purchasing a domain** on Cloudflare (not just using a free subdomain)
2. **Creating a DNS A record** pointing the domain to the current public IP
3. **Generating a Cloudflare Global API Key** — this was the most confusing step, requiring navigation through Cloudflare's account security settings to generate and copy the correct API token
4. **Entering credentials in pfSense** under Services → Dynamic DNS → Add Client

```
Service:     Cloudflare
Interface:   WAN
Hostname:    [custom domain]
API Key:     [Cloudflare Global API Key]
Status:      ✅ Active — automatically updates when public IP changes
```

### Result
pfSense now automatically detects public IP changes and updates the Cloudflare DNS record — keeping the domain always pointing to the correct IP.

---

## 🔒 VPN Integration — PIA (Private Internet Access)

Configured PIA VPN as a third network interface in pfSense, allowing selective traffic routing through the VPN tunnel.

### What Was Configured
- Installed PIA OpenVPN client configuration inside pfSense
- Created **PIA as a separate gateway** (opt1 interface)
- Configured **policy-based routing** — specific devices or traffic types route through VPN while others use direct WAN
- VPN gateway shows ~131ms RTT with 4% packet loss — normal for VPN tunneling

### Why This Matters (Security)
- Separates VPN traffic from regular traffic at the **network level** — not just the device level
- Provides privacy for specific devices without slowing down the entire network
- Demonstrates understanding of gateway policies and traffic routing

---

## 🛡️ pfBlockerNG — Threat Intelligence Blocking

Installed and configured pfBlockerNG package on pfSense for network-wide threat blocking.

### What It Does
- Blocks malicious IPs and domains **before they reach any device** on the network
- Works at DNS level (DNSBL) and IP level simultaneously
- Updates blocklists automatically on a schedule

### Current Status
```
IP Blocklist:    Active
DNSBL:           Active
Alias (pfB_PRI1_v4):  14,424 blocked entries
Last Updated:    Automatic scheduled updates
```

### Why This Is Better Than Device-Level Blocking
Traditional ad/threat blockers work on individual devices. pfBlockerNG works at the **network gateway** — every device on the network (phones, laptops, IoT devices) is protected automatically without any software installation.

---

## 📊 Final Dashboard — All Systems Running

All configured components running simultaneously:

```
┌─────────────────────────────────────────┐
│  pfSense 2.8.1-RELEASE                  │
├─────────────────────────────────────────┤
│  Interfaces                             │
│  WAN  ✅  1000baseT full-duplex         │
│  LAN  ✅  100baseTX full-duplex         │
│  PIA  ✅  VPN Tunnel Active             │
├─────────────────────────────────────────┤
│  Gateways                    Status     │
│  WAN_DHCP6    0.6ms RTT  →  Online ✅  │
│  WAN_DHCP     0.6ms RTT  →  Online ✅  │
│  PIA_VPNV4  131.8ms RTT  →  Online ✅  │
├─────────────────────────────────────────┤
│  pfBlockerNG                            │
│  IP Blocking    ✅  Active              │
│  DNSBL          ✅  Active              │
│  Blocked IPs    14,424 entries          │
├─────────────────────────────────────────┤
│  Dynamic DNS                            │
│  Cloudflare DDNS  ✅  Active on WAN    │
└─────────────────────────────────────────┘
```

---

## 🧠 Key Skills Demonstrated

| Skill | Evidence |
|-------|---------|
| Network architecture & wiring | Physical lab with CAT6, switch, AP |
| Firewall configuration | pfSense with WAN/LAN/VPN interfaces |
| DHCP management | Static mappings + dynamic range |
| VPN routing | PIA gateway with policy-based routing |
| DNS security | pfBlockerNG with 14,424 blocked entries |
| Dynamic DNS | Cloudflare API integration |
| Independent problem solving | Figured out Cloudflare API setup without tutorial guidance |
| VM → Physical deployment | Tested in VirtualBox first, deployed on real hardware |

---

## 📚 What I Learned Beyond the Tutorial

- Physical interface assignment (ETH0/ETH1) requires testing — not always obvious
- Cloudflare DDNS requires domain purchase + API key generation — not covered in most guides
- pfSense acts as a complete network stack replacement — DHCP, DNS, firewall, VPN, all in one
- Running VPN at router level protects all devices simultaneously vs per-device VPN apps
- pfBlockerNG's network-level blocking is far more effective than browser extensions

---

*Built as part of hands-on cybersecurity learning — currently running live 24/7*
