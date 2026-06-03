# 🖥️ LXC Container Setup on Proxmox

## Why LXC Instead of a Full VM?

| | LXC Container | Full VM |
|---|---|---|
| **RAM Usage** | ~100MB | ~512MB+ |
| **Boot Time** | ~3 seconds | ~30+ seconds |
| **Disk Space** | ~2GB | ~8GB+ |
| **Overhead** | Shares host kernel | Full OS emulation |
| **Best For** | Lightweight services ✅ | Full OS isolation |

LXC was chosen because Pi-hole doesn't require kernel isolation —
sharing the Proxmox host kernel cuts resource usage by 80%.

---

## Prerequisites
- Proxmox VE installed and accessible at `https://192.168.1.130:8006`
- Debian 12 LXC template downloaded

## Step 1 — Download LXC Template
In Proxmox web UI:

Click **Download** and wait for completion.

---

## Step 2 — Create the LXC Container
Click **"Create CT"** and configure:

### General Tab
| Setting | Value |
|---|---|
| CT ID | `101` |
| Hostname | `Pihole` |
| Unprivileged container | ✅ Yes |

### Template Tab
| Setting | Value |
|---|---|
| Template | `debian-12-standard` |

### CPU Tab
| Setting | Value |
|---|---|
| Cores | `1` |

### Memory Tab
| Setting | Value |
|---|---|
| RAM | `512 MB` |
| Swap | `256 MB` |

### Network Tab
| Setting | Value |
|---|---|
| Bridge | `vmbr0` |
| IPv4 | `Static` |
| IPv4/CIDR | `192.168.1.60/24` |
| Gateway | `192.168.1.254` |

### DNS Tab
| Setting | Value |
|---|---|
| DNS server | `8.8.8.8` |

---

## Step 3 — Critical LXC Config Tweak
Pi-hole requires a capability adjustment for unprivileged containers.

In the **Proxmox host shell**:
```bash
echo "lxc.cap.drop:" >> /etc/pve/lxc/101.conf
```

---

## Step 4 — Fix DNS Configuration
Proxmox controls container DNS via the host config file:
```bash
nano /etc/pve/lxc/101.conf
```

Ensure these lines are set:

---

## Step 5 — Start the Container
```bash
pct start 101
```

Or via Proxmox web UI → select container → **Start**

---

## Step 6 — Verify Container is Running
```bash
# Check container status
pct status 101

# Enter container shell
pct enter 101

# Verify network
ip a
ping -c 3 google.com
```

---

## Important Notes

### IP Address Strategy
Pi-hole's static IP `192.168.1.60` is intentionally set **below** the
AT&T router's DHCP pool (`192.168.1.64 - 192.168.1.253`).

This ensures the router can never accidentally assign this IP to
another device via DHCP — eliminating IP conflict risk.

### Container vs Host Shell
| Task | Where |
|---|---|
| Install software, run Pi-hole commands | LXC container shell |
| Start/stop containers, edit .conf files | Proxmox host shell |

### Useful Proxmox Commands
```bash
pct list              # List all containers
pct start 101         # Start container 101
pct stop 101          # Stop container 101
pct enter 101         # Enter container shell
pct status 101        # Check container status
cat /etc/pve/lxc/101.conf  # View container config
```
