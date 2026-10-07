# 🔐 Site-to-Site VPN + Remote Access VPN Lab

## 📋 IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 |
| R1 | Gi0/0 (LAN) | 192.168.1.1 | 255.255.255.0 |
| R1 | Gi0/1 (WAN) | 10.0.0.1 | 255.255.255.252 |
| ISP | Gi0/0 | 10.0.0.2 | 255.255.255.252 |
| ISP | Gi0/1 | 10.0.1.1 | 255.255.255.252 |
| ISP | Gi0/2 | 10.0.2.1 | 255.255.255.252 |
| R2 | Gi0/0 (WAN) | 10.0.1.2 | 255.255.255.252 |
| R2 | Gi0/1 (LAN) | 192.168.2.1 | 255.255.255.0 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 |
| REMOTE PC | NIC | 10.0.2.2 | 255.255.255.252 |

> **Default Gateways:** PC1 → 192.168.1.1 | PC2 → 192.168.2.1 | REMOTE PC → 10.0.2.1

> **VPN Pool (Remote Access):** 192.168.100.1 – 192.168.100.10

---

## ⚙️ STEP 1 — Configure PC IP Addresses

### PC1
```
IP: 192.168.1.10
Mask: 255.255.255.0
Gateway: 192.168.1.1
```

### PC2
```
IP: 192.168.2.10
Mask: 255.255.255.0
Gateway: 192.168.2.1
```

### REMOTE PC
```
IP: 10.0.2.2
Mask: 255.255.255.252
Gateway: 10.0.2.1
```

---

## ⚙️ STEP 2 — Configure ISP Router

> Click ISP → CLI tab → paste this:

```
enable
conf t
hostname ISP
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
interface GigabitEthernet0/1
 ip address 10.0.1.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/2
 ip address 10.0.2.1 255.255.255.252
 no shutdown
end
```

---

## ⚙️ STEP 3 — Configure R1 (HQ Router)

> Click R1 → CLI tab → paste this:

```
enable
conf t
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
! ===== AAA FOR REMOTE ACCESS VPN =====
aaa new-model
aaa authentication login VPN-AUTHEN local
aaa authorization network VPN-AUTHOR local
!
username vpnuser secret vpnpass123
!
! ===== PHASE 1 - ISAKMP POLICY =====
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
!
! ===== SITE-TO-SITE VPN - Pre-Shared Key =====
crypto isakmp key Site2Site address 10.0.1.2
!
! ===== REMOTE ACCESS VPN - Client Group =====
crypto isakmp client configuration group REMOTE-VPN
 key RemoteKey123
 pool REMOTE-POOL
!
ip local pool REMOTE-POOL 192.168.100.1 192.168.100.10
!
! ===== PHASE 2 - TRANSFORM SETS =====
crypto ipsec transform-set SITE-SET esp-aes esp-sha-hmac
 mode tunnel
!
crypto ipsec transform-set REMOTE-SET esp-aes esp-sha-hmac
 mode tunnel
!
! ===== SITE-TO-SITE ACL =====
ip access-list extended SITE-ACL
 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
!
! ===== DYNAMIC MAP FOR REMOTE ACCESS =====
crypto dynamic-map DYN-MAP 10
 set transform-set REMOTE-SET
 reverse-route
!
! ===== CRYPTO MAP =====
crypto map CRYPTO-MAP 10 ipsec-isakmp
 set peer 10.0.1.2
 set transform-set SITE-SET
 match address SITE-ACL
!
crypto map CRYPTO-MAP 20 ipsec-isakmp dynamic DYN-MAP
!
crypto map CRYPTO-MAP client authentication list VPN-AUTHEN
crypto map CRYPTO-MAP isakmp authorization list VPN-AUTHOR
crypto map CRYPTO-MAP client configuration address respond
!
! ===== APPLY CRYPTO MAP TO WAN INTERFACE =====
interface GigabitEthernet0/1
 crypto map CRYPTO-MAP
!
end
```

---

## ⚙️ STEP 4 — Configure R2 (Branch Router)

> Click R2 → CLI tab → paste this:

```
enable
conf t
hostname R2
!
interface GigabitEthernet0/0
 ip address 10.0.1.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.0.1.1
!
! ===== PHASE 1 - ISAKMP POLICY =====
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
!
! ===== SITE-TO-SITE VPN - Pre-Shared Key =====
crypto isakmp key Site2Site address 10.0.0.1
!
! ===== PHASE 2 - TRANSFORM SET =====
crypto ipsec transform-set SITE-SET esp-aes esp-sha-hmac
 mode tunnel
!
! ===== SITE-TO-SITE ACL =====
ip access-list extended SITE-ACL
 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
!
! ===== CRYPTO MAP =====
crypto map CRYPTO-MAP 10 ipsec-isakmp
 set peer 10.0.0.1
 set transform-set SITE-SET
 match address SITE-ACL
!
! ===== APPLY CRYPTO MAP TO WAN INTERFACE =====
interface GigabitEthernet0/0
 crypto map CRYPTO-MAP
!
end
```

---

## ⚙️ STEP 5 — Configure Remote Access VPN Client (REMOTE PC)

> Click REMOTE PC → Desktop tab → VPN

```
Group Name:   REMOTE-VPN
Group Key:    RemoteKey123
Host IP:      10.0.0.1
Username:     vpnuser
Password:     vpnpass123
```

> Click **Connect**

---

## ✅ Verification Commands

### Test Site-to-Site VPN (run on PC1):
```
ping 192.168.2.10
```

### Check VPN tunnel on R1:
```
show crypto isakmp sa
show crypto ipsec sa
```

### Check VPN tunnel on R2:
```
show crypto isakmp sa
show crypto ipsec sa
```

### Test Remote Access VPN:
> After connecting VPN on REMOTE PC → ping 192.168.1.10

---

## 🗺️ Final Topology Summary

```
[PC1: .1.10] — [SW1] — [R1: .1.1 | .0.1] — [ISP: .0.2 | .1.1 | .2.1] — [R2: .1.2 | .2.1] — [SW2] — [PC2: .2.10]
                                                       |
                                               [REMOTE PC: .2.2]
                                         (connects via Remote Access VPN)

Site-to-Site VPN Tunnel: R1 (10.0.0.1) ↔ R2 (10.0.1.2)
Remote Access VPN: REMOTE PC → R1
VPN Client Pool: 192.168.100.0/24
```
