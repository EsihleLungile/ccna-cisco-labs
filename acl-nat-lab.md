# 🔷 Lab 7 — ACL & NAT/PAT

**Topics:** Standard ACL, Extended ACL, NAT, PAT  
**Tools:** GNS3 / Packet Tracer / EVE-NG  
**Difficulty:** 🔴 Advanced

---

## 📖 Part A — Access Control Lists (ACL)

ACLs are ordered sets of rules that permit or deny traffic based on IP addresses, protocols, and ports.

### ACL Types

| Type | Number Range | Filters On |
|---|---|---|
| **Standard** | 1–99, 1300–1999 | Source IP only |
| **Extended** | 100–199, 2000–2699 | Source IP, Destination IP, Protocol, Port |

> 💡 **Golden Rule:** Place Standard ACLs close to the **destination**. Place Extended ACLs close to the **source**.

---

## 🧪 Lab 7A — Standard ACL

### Scenario
Block PC1 (192.168.10.10) from reaching the server network (192.168.30.0/24). Allow everyone else.

```cisco
! Create standard ACL 10
Router(config)# access-list 10 deny host 192.168.10.10
Router(config)# access-list 10 permit any

! Apply to the interface closest to the destination (inbound or outbound)
Router(config)# interface gigabitethernet 0/1
Router(config-if)# ip access-group 10 out
Router(config-if)# exit
```

### Named Standard ACL (Easier to manage)

```cisco
Router(config)# ip access-list standard BLOCK_PC1
Router(config-std-nacl)# deny host 192.168.10.10
Router(config-std-nacl)# permit any
Router(config-std-nacl)# exit

Router(config)# interface gigabitethernet 0/1
Router(config-if)# ip access-group BLOCK_PC1 out
```

---

## 🧪 Lab 7B — Extended ACL

### Scenario
Block VLAN 10 (192.168.10.0/24) from accessing a web server (192.168.30.5) on port 80 only. Allow all other traffic.

```cisco
! Create extended ACL 110
Router(config)# access-list 110 deny tcp 192.168.10.0 0.0.0.255 host 192.168.30.5 eq 80
Router(config)# access-list 110 permit ip any any

! Apply close to the source (VLAN 10 interface — inbound)
Router(config)# interface gigabitethernet 0/0
Router(config-if)# ip access-group 110 in
Router(config-if)# exit
```

### Named Extended ACL

```cisco
Router(config)# ip access-list extended WEB_RESTRICT
Router(config-ext-nacl)# deny tcp 192.168.10.0 0.0.0.255 host 192.168.30.5 eq 80
Router(config-ext-nacl)# deny tcp 192.168.10.0 0.0.0.255 host 192.168.30.5 eq 443
Router(config-ext-nacl)# permit ip any any
Router(config-ext-nacl)# exit
```

### Common Port Numbers for ACLs

| Port | Protocol | Application |
|---|---|---|
| 20, 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP (Email) |
| 53 | UDP/TCP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3389 | TCP | RDP |

### ACL Wildcards Reference

| Wildcard | Meaning |
|---|---|
| `0.0.0.0` | Match exact host (same as `host` keyword) |
| `0.0.0.255` | Match any host in /24 subnet |
| `0.0.255.255` | Match any host in /16 subnet |
| `255.255.255.255` | Match any host (same as `any` keyword) |

---

## ✅ ACL Verification

```cisco
! View all ACLs and their hit counts
Router# show access-lists

! View ACLs applied to an interface
Router# show ip interface gigabitethernet 0/0

! View a specific named ACL
Router# show ip access-lists WEB_RESTRICT
```

---

## 📖 Part B — NAT & PAT

**NAT (Network Address Translation)** translates private IP addresses to public IPs — allowing internal devices to reach the internet.

### NAT Types

| Type | Description |
|---|---|
| **Static NAT** | One-to-one mapping: one private IP → one public IP |
| **Dynamic NAT** | Pool of public IPs assigned to private IPs as needed |
| **PAT (Overload)** | Many private IPs → one public IP (uses port numbers to distinguish) |

---

## 🧪 Lab 7C — PAT (Most Common in Real Networks)

### Topology
```
[192.168.10.x] ── Router ── [Internet / Public IP: 203.0.113.1]
```

### Step 1 — Define Inside and Outside Interfaces

```cisco
! Interface facing the internal network
Router(config)# interface gigabitethernet 0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

! Interface facing the internet
Router(config)# interface gigabitethernet 0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
```

### Step 2 — Create ACL to Define Which IPs to Translate

```cisco
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255
Router(config)# access-list 1 permit 192.168.20.0 0.0.0.255
```

### Step 3 — Configure PAT (Overload)

```cisco
! Translate using the router's outside interface IP
Router(config)# ip nat inside source list 1 interface gigabitethernet 0/1 overload
```

---

## 🧪 Lab 7D — Static NAT

Map a private server (192.168.10.100) to a fixed public IP (203.0.113.50):

```cisco
Router(config)# ip nat inside source static 192.168.10.100 203.0.113.50
```

---

## ✅ NAT Verification

```cisco
! View active NAT translations
Router# show ip nat translations

! View NAT statistics
Router# show ip nat statistics

! Clear NAT translation table
Router# clear ip nat translation *

! Debug NAT in real time
Router# debug ip nat
```

### Example `show ip nat translations` Output

```
Pro Inside global      Inside local       Outside local      Outside global
tcp 203.0.113.1:1025   192.168.10.5:1025  8.8.8.8:53        8.8.8.8:53
tcp 203.0.113.1:1026   192.168.10.8:1026  93.184.216.34:80  93.184.216.34:80
```

---

## 📘 Key Takeaways

**ACL:**
- ACLs process rules top-to-bottom — order matters
- There is an implicit `deny all` at the end of every ACL
- Standard ACL → near destination; Extended ACL → near source
- Named ACLs are easier to edit and manage than numbered ACLs
- Use `permit ip any any` at the end to avoid blocking everything accidentally

**NAT/PAT:**
- PAT is the most common form — almost every home router uses it
- Always define `ip nat inside` and `ip nat outside` on the correct interfaces
- Static NAT is used for servers that need a consistent public IP
- `show ip nat translations` is your primary troubleshooting tool
