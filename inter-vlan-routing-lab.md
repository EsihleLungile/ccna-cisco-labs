# 🔷 Lab 4 — Inter-VLAN Routing (IVR & Router-on-a-Stick)

**Topics:** Inter-VLAN routing, L3 Switch (IVR), Router-on-a-Stick (ROAS), sub-interfaces  
**Tools:** GNS3 / Packet Tracer / EVE-NG  
**Difficulty:** 🟡 Intermediate

---

## 📖 Theory Summary

By default, devices in different VLANs cannot communicate — each VLAN is a separate broadcast domain. **Inter-VLAN Routing** uses a Layer 3 device (router or L3 switch) to route traffic between VLANs.

There are two main methods:

| Method | Device Needed | Notes |
|---|---|---|
| **IVR (Inter-VLAN Routing)** | Layer 3 Switch | Modern, preferred approach |
| **ROAS (Router-on-a-Stick)** | Router + L2 Switch | Legacy but still common in CCNA |

---

## 🧪 METHOD 1 — Inter-VLAN Routing (L3 Switch)

### Topology
```
[PC1 - VLAN 10]──┐
                 [L3 Switch] ── routes between VLANs
[PC2 - VLAN 20]──┘
```

### Tasks
- Task 4.1 — Create VLAN 10 (Sales) and VLAN 20 (Marketing)
- Task 4.2 — Assign VLANs to ports
- Task 4.3 — Configure SVI (Switched Virtual Interface) as gateway for each VLAN

---

### Task 4.1 — Create VLANs

```cisco
Switch> enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name MRKT
Switch(config-vlan)# exit
```

### Task 4.2 — Assign VLANs to Ports

```cisco
! Assign VLAN 10 to GigabitEthernet 0/0 and 0/1
Switch(config)# interface range gigabitethernet 0/0-1
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit

! Assign VLAN 20 to GigabitEthernet 0/2 and 0/3
Switch(config)# interface range gigabitethernet 0/2-3
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit
```

### Task 4.3 — Create SVIs (Default Gateways)

```cisco
! SVI for VLAN 10 — acts as the default gateway for VLAN 10 hosts
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.254 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! SVI for VLAN 20
Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.254 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Enable IP routing on the L3 switch
Switch(config)# ip routing
```

> 💡 The SVI IP becomes the default gateway for all PCs in that VLAN. PC1 would set its gateway to 192.168.10.254.

---

## 🧪 METHOD 2 — Router-on-a-Stick (ROAS)

### Topology
```
[PC1 - VLAN 10]──┐
                 [L2 Switch] ── trunk ── [Router R1]
[PC2 - VLAN 20]──┘
```

### Tasks
- Task 5.1 — Create VLANs on the switch
- Task 5.2 — Assign VLANs to access ports
- Task 5.3 — Configure trunk port toward the router
- Task 5.4 — Configure router sub-interfaces

---

### Task 5.1 — Create VLANs on Switch

```cisco
SW(config)# vlan 10
SW(config-vlan)# name SALES
SW(config-vlan)# exit

SW(config)# vlan 20
SW(config-vlan)# name MRKT
SW(config-vlan)# exit
```

### Task 5.2 — Assign VLANs to Ports

```cisco
! Ports for VLAN 10
SW(config)# interface range ethernet 0/1-2
SW(config-if-range)# switchport mode access
SW(config-if-range)# switchport access vlan 10
SW(config-if-range)# exit

! Ports for VLAN 20
SW(config)# interface range ethernet 0/3, ethernet 1/0
SW(config-if-range)# switchport mode access
SW(config-if-range)# switchport access vlan 20
SW(config-if-range)# exit
```

### Task 5.3 — Configure Trunk Port to Router

```cisco
SW(config)# interface ethernet 0/0
SW(config-if)# switchport trunk encapsulation dot1q
SW(config-if)# switchport mode trunk
SW(config-if)# exit
```

### Task 5.4 — Configure Router Sub-Interfaces

```cisco
R1(config)# interface ethernet 0/0
R1(config-if)# no shutdown
R1(config-if)# exit

! Sub-interface for VLAN 10
R1(config)# interface ethernet 0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.254 255.255.255.0
R1(config-subif)# no shutdown
R1(config-subif)# exit

! Sub-interface for VLAN 20
R1(config)# interface ethernet 0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.254 255.255.255.0
R1(config-subif)# no shutdown
R1(config-subif)# exit
```

---

## ✅ Verification Commands

```cisco
! On the switch — verify VLANs and trunks
Switch# show vlan brief
Switch# show interfaces trunk
Switch# show ip interface brief

! On the router — verify sub-interfaces
Router# show ip interface brief
Router# show running-config interface ethernet 0/0.10

! Test connectivity from PC1 (VLAN 10) to PC2 (VLAN 20)
! PC1 should be able to ping 192.168.20.x after proper gateway configuration
```

---

## 📋 IVR vs ROAS Comparison

| Feature | IVR (L3 Switch) | ROAS (Router) |
|---|---|---|
| Hardware needed | L3 Switch | Router + L2 Switch |
| Performance | High (hardware routing) | Lower (software routing) |
| Complexity | Simpler | Slightly more complex |
| Cost | Higher switch cost | More flexible use of existing hardware |
| Recommended for | Modern networks | Legacy or small environments |

---

## 📘 Key Takeaways

- VLANs cannot communicate without a Layer 3 device
- IVR uses SVIs on an L3 switch — requires `ip routing` to be enabled
- ROAS uses router sub-interfaces with 802.1Q encapsulation
- The SVI or sub-interface IP becomes the default gateway for hosts in that VLAN
- Trunk links must be configured between the switch and router for ROAS
