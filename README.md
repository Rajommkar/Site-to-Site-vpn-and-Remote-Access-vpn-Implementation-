# 🔐 Site-to-Site VPN & Remote Access VPN Implementation

A Cisco Packet Tracer lab demonstrating both **Site-to-Site IPSec VPN** and **Remote Access VPN** configurations.

## 📁 Files

| File | Description |
|---|---|
| `PROJECT WORK.pkt` | Cisco Packet Tracer topology file |
| `VPN_Lab_Configs.md` | Full device configurations & IP plan |

## 🗺️ Topology

```
[PC1] — [SW1] — [R1/HQ] ══IPSec Tunnel══ [ISP] ══IPSec Tunnel══ [R2/Branch] — [SW2] — [PC2]
                                                  |
                                            [REMOTE PC]
                                       (Remote Access VPN Client)
```

## 🌐 IP Addressing

| Device | Interface | IP Address |
|---|---|---|
| PC1 | NIC | 192.168.1.10/24 |
| R1 | Gi0/0 | 192.168.1.1/24 |
| R1 | Gi0/1 | 10.0.0.1/30 |
| ISP | Gi0/0 | 10.0.0.2/30 |
| ISP | Gi0/1 | 10.0.1.1/30 |
| ISP | Gi0/2 | 10.0.2.1/30 |
| R2 | Gi0/0 | 10.0.1.2/30 |
| R2 | Gi0/1 | 192.168.2.1/24 |
| PC2 | NIC | 192.168.2.10/24 |
| REMOTE PC | NIC | 10.0.2.2/30 |

## 🔐 VPN Details

### Site-to-Site VPN
- **Protocol:** IPSec IKEv1
- **Encryption:** AES
- **Hash:** SHA
- **DH Group:** 2
- **Pre-shared Key:** `Site2Site`
- **HQ Peer:** 10.0.0.1 (R1)
- **Branch Peer:** 10.0.1.2 (R2)

### Remote Access VPN
- **Group Name:** `REMOTE-VPN`
- **Group Key:** `RemoteKey123`
- **VPN Server:** 10.0.0.1 (R1)
- **VPN Pool:** 192.168.100.1 – 192.168.100.10
- **Username:** `vpnuser`

## 🛠️ Tools Used
- Cisco Packet Tracer 9.0.0
- Cisco 2911 Routers
- Cisco 2960-24TT Switches

## 👤 Author
Rajommkar
