# 🔧 Troubleshooting Log

Real issues encountered during deployment and how they were resolved.

---

## 1. DNS Not Resolving in LXC Container
**Error:** `curl: (6) Could not resolve host: install.pi-hole.net`

**Root Cause:** Fresh LXC container had empty `/etc/resolv.conf` — 
no DNS nameserver configured.

**Fix:**
```bash
tee /etc/resolv.conf << EOF
nameserver 8.8.8.8
nameserver 1.1.1.1
EOF
```

---

## 2. resolv.conf Overridden by Proxmox
**Error:** Changes to resolv.conf inside container not persisting after reboot.

**Root Cause:** Proxmox host controls LXC container DNS via
`/etc/pve/lxc/101.conf` and overwrites resolv.conf on every reboot.

**Fix:**
```bash
nano /etc/pve/lxc/101.conf
# Change:
searchdomain: local
nameserver: 8.8.8.8 1.1.1.1
pct stop 101 && pct start 101
```

---

## 3. Search Domain Conflict
**Error:** DNS lookups failing — `google.com` being appended to hostnames.

**Root Cause:** `search google.com` in resolv.conf caused lookups for
`install.pi-hole.net.google.com` instead of `install.pi-hole.net`.

**Fix:** Changed searchdomain to `local` in Proxmox node DNS settings.

---

## 4. IP Conflict Between PVE Host and LXC Container
**Error:** LXC container assigned same IP as Proxmox host (192.168.1.130).

**Root Cause:** IP was manually set to same address during container creation.

**Fix:**
```bash
nano /etc/pve/lxc/101.conf
# Changed:
# ip=192.168.1.131/24  →  ip=192.168.1.60/24
pct stop 101 && pct start 101
```

---

## 5. Pi-hole v6 Gravity Download Timeout
**Error:** `Resolving timed out after 10000 milliseconds` on all blocklists.

**Root Cause:** Pi-hole v6 pihole.toml had no upstream DNS configured.
FTL couldn't resolve external hostnames during gravity downloads.

**Fix:**
```bash
nano /etc/pihole/pihole.toml
# Added under [dns]:
# upstreams = ["8.8.8.8", "1.1.1.1"]
systemctl restart pihole-FTL
```

---

## 6. Blocklist Still Failing After DNS Fix
**Error:** Gravity timeout persisted even after upstream DNS was configured.

**Root Cause:** Pi-hole v6 gravity spawns a child process with its own
DNS stack and a hardcoded 10 second timeout — separate from FTL config.

**Fix:** Manual download and SQLite import:
```bash
# Download blocklist directly
curl -sSL https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts \
  -o /tmp/blocklist.txt --max-time 120

# Import into Pi-hole v6 database
sqlite3 /etc/pihole/gravity.db \
  "INSERT OR REPLACE INTO adlist (address, enabled, comment) \
  VALUES ('file:///tmp/blocklist.txt', 1, 'StevenBlack manual import');"

pihole -g
```
**Result:** 84,752 domains loaded successfully ✅

---

## 7. AT&T BGW530 Not Exposing DNS Settings
**Problem:** AT&T BGW530-900 Subnets & DHCP page has no DNS server field.

**Root Cause:** AT&T deliberately locks DNS settings on their gateway.

**Workaround:** Set Pi-hole DNS manually on each device:
```bash
# Mac Terminal
networksetup -setdnsservers Wi-Fi 192.168.1.60 1.1.1.1
```
**Long term fix:** Third party router with IP Passthrough mode.

---

## 8. Network Outage From Dual DHCP Servers
**Problem:** All devices lost internet after enabling Pi-hole DHCP.

**Root Cause:** AT&T router and Pi-hole both running DHCP simultaneously
caused IP assignment conflicts across all devices.

**Fix:**
```bash
# Disable Pi-hole DHCP immediately
pihole-FTL --config dhcp.active false
systemctl restart pihole-FTL
```
Then re-enabled AT&T DHCP via router admin page.

**Lesson:** Always enable new DHCP server and confirm it works BEFORE
disabling the existing one. Never run two DHCP servers on the same subnet.

---

## Key Lessons Learned
- Always check one layer up when container-level fixes don't stick
- Proxmox host controls LXC DNS — not the container itself  
- Pi-hole v6 uses TOML config — v5 guides don't apply
- ISP routers are deliberately locked down
- Never disable DHCP before confirming replacement is working
- `No route to host` = no IP configured, not just unreachable destination
- Put infrastructure IPs below the DHCP pool to avoid conflicts
