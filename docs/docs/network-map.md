# 🗺️ Network Map

## IP Address Allocation

| IP Address | Device | Type | Status |
|---|---|---|---|
| 192.168.1.254 | AT&T BGW530-900 | Gateway/Router | Static |
| 192.168.1.130 | Proxmox Host | Hypervisor | Fixed |
| 192.168.1.60 | Pi-hole LXC (CT 101) | DNS Server | Static |
| 192.168.1.223 | MacBook Air | Workstation | Fixed |
| 192.168.1.64-253 | DHCP Pool | Client Devices | Dynamic |

## Network Diagram

## DHCP Configuration
| Setting | Value |
|---|---|
| DHCP Server | AT&T BGW530 |
| Pool Start | 192.168.1.64 |
| Pool End | 192.168.1.253 |
| Lease Time | 1 day |

## Infrastructure Zone (Below DHCP Pool)

## DNS Configuration
| Device | DNS Server | Method |
|---|---|---|
| MacBook Air | 192.168.1.60 | Manual |
| Other devices | 192.168.1.254 | DHCP (AT&T default) |

## Known Limitations
- AT&T BGW530 does not expose DNS settings in DHCP configuration
- Network-wide DNS enforcement requires third party router
- Pi-hole IP (192.168.1.60) is outside DHCP pool — conflict-safe

## Future Network Plan
