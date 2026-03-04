# 🔒 Secure VPS Setup — Tailscale + Pi-hole on UpCloud

![Security](https://img.shields.io/badge/Security-Hardened-green)
![Tailscale](https://img.shields.io/badge/Network-Tailscale-blue)
![Pi--hole](https://img.shields.io/badge/DNS-Pi--hole-red)
![Platform](https://img.shields.io/badge/Platform-UpCloud-orange)
![OS](https://img.shields.io/badge/OS-Debian-informational)

A hardened, private VPS setup using **Tailscale** and **Pi-hole** (network-wide DNS filtering). All services are locked behind a private Tailscale network — nothing is exposed to the public internet.

---

## 📐 Architecture Overview

```
[ Your Devices ]
      │
      │  Tailscale VPN (WireGuard encrypted)
      │
[ UpCloud VPS — Debian ]
      ├── Pi-hole (DNS filtering + ad blocking)
      ├── Tailscale (Exit Node + Private Network)
      └── UFW Firewall (deny all public, allow Tailscale only)
```

- All traffic between devices and the VPS flows through **Tailscale's encrypted WireGuard tunnel**
- The VPS acts as an **exit node**, routing internet traffic privately
- Pi-hole handles **DNS filtering** for all devices on the Tailscale network
- UFW enforces a **deny-by-default** policy on all public interfaces

---

## 🛠️ Stack

| Component | Purpose |
|-----------|---------|
| [UpCloud](https://upcloud.com) | Cloud VPS hosting |
| [Debian Linux](https://debian.org) | Server OS |
| [Tailscale](https://tailscale.com) | Zero-trust VPN / private network |
| [Pi-hole](https://pi-hole.net) | DNS-level ad & tracker blocking |
| [UFW](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29) | Host-based firewall |

---

## 🔐 Security Hardening Applied

### 1. UFW Firewall — Deny All Public, Allow Tailscale Only

All sensitive ports are denied publicly and only accessible through the Tailscale interface (`tailscale0`):

| Port | Service | Public | Tailscale |
|------|---------|--------|-----------|
| 22 | SSH | ❌ Denied | ✅ Allowed |
| 53 | DNS (Pi-hole) | ❌ Denied | ✅ Allowed |
| 80 | HTTP (Pi-hole) | ❌ Denied | ✅ Allowed |
| 443 | HTTPS (Pi-hole) | ❌ Denied | ✅ Allowed |
| 41641/udp | Tailscale | ✅ Required | — |

### 2. SSH Hardening
- Root login disabled (`PermitRootLogin no`)
- Password authentication disabled (`PasswordAuthentication no`)
- Key-based authentication only
- SSH only reachable via Tailscale IP (`100.x.x.x`)

### 3. Tailscale Zero-Trust Networking
- All devices require authentication before joining the network
- Device authorization enabled — new devices require manual approval
- Key expiry enabled to force periodic re-authentication
- Exit node configured for private internet routing
- MagicDNS enabled for easy hostname resolution across devices

### 4. Pi-hole DNS Security
- DNS queries filtered at network level
- Only accessible from within the Tailscale network
- Blocks ads, trackers, and malicious domains before they load

---

## 🚀 Setup Guide

### Prerequisites
- An UpCloud account with a Debian VPS
- A Tailscale account ([free tier available](https://tailscale.com/pricing))
- SSH access to your VPS

### Step 1 — Install Tailscale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --advertise-exit-node
```

### Step 2 — Install Pi-hole
```bash
curl -sSL https://install.pi-hole.net | bash
```
During setup, bind Pi-hole to your Tailscale interface only.

### Step 3 — Configure UFW
```bash
sudo ufw enable

# SSH — Tailscale only
sudo ufw deny 22
sudo ufw allow in on tailscale0 to any port 22

# Pi-hole DNS — Tailscale only
sudo ufw deny 53
sudo ufw allow in on tailscale0 to any port 53

# Pi-hole Web UI — Tailscale only
sudo ufw deny 80
sudo ufw deny 443
sudo ufw allow in on tailscale0 to any port 80
sudo ufw allow in on tailscale0 to any port 443

sudo ufw reload
```

### Step 4 — Harden SSH
```bash
sudo nano /etc/ssh/sshd_config
```
Set the following:
```
PermitRootLogin no
PasswordAuthentication no
```
Then restart SSH:
```bash
sudo systemctl restart sshd
```

### Step 5 — Verify
```bash
# Check firewall rules
sudo ufw status numbered

# Check open ports
ss -tulpn

# Check Tailscale status
tailscale status
```

---

## 🔧 Maintenance

### Regular Tasks

| Task | Command | Frequency |
|------|---------|-----------|
| Update system | `sudo apt update && sudo apt upgrade -y` | Weekly |
| Check firewall | `sudo ufw status numbered` | Monthly |
| Review Tailscale devices | Tailscale admin console | Monthly |
| Run security audit | `sudo lynis audit system` | Monthly |
| Check open ports | `ss -tulpn` | Monthly |
| Update Pi-hole | `pihole -up` | Monthly |

### Useful Commands
```bash
# Check who's logged in
last -n 20

# Check failed SSH attempts
sudo journalctl -u ssh | grep Failed

# Check Tailscale connected devices
tailscale status

# Check UFW logs
sudo tail -f /var/log/ufw.log

# Run full security audit
sudo lynis audit system
```

---

## 🚨 Incident Response

If you suspect unauthorized access:

```bash
# Check active sessions
who

# Check recent logins
last -n 20

# Check suspicious cron jobs
crontab -l && ls /etc/cron.*

# Check for unexpected open ports
ss -tulpn

# Immediately revoke a compromised device from Tailscale
# → Go to https://login.tailscale.com/admin/machines and disable/remove it
```

---

## 📁 Related Projects

> Part of a personal portfolio of infrastructure, security, and software engineering projects.

| Repo | Description |
|------|-------------|
| 🔒 **vps-secure-setup** *(this repo)* | Hardened VPS with Tailscale + Pi-hole |
| 🛡️ *Coming soon* | Azure Cloud security projects |
| 📊 *Coming soon* | Power BI dashboards |
| 🌐 *Coming soon* | Network scanner tool (Pscaner) |

---

## 📜 License

MIT License — feel free to use this as a reference for your own setup.

---

> Built and maintained by [Joshua Galloway](https://github.com/jjgalloway06) — Apprentice Software Engineer based in Edinburgh, UK.
