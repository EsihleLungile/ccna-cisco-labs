# 🔷 Lab 3 — EtherChannel (LACP & PAgP)

**Topics:** Link aggregation, LACP, PAgP, Port-Channel configuration  
**Tools:** GNS3 / Packet Tracer / EVE-NG  
**Difficulty:** 🟡 Intermediate

---

## 📖 Theory Summary

**EtherChannel** bundles multiple physical links between two switches into a single logical link. This increases bandwidth and provides redundancy — if one physical link fails, traffic continues on the remaining links.

**Example:** Three 10 Mbps links bundled = 30 Mbps combined throughput.

---

## 📋 EtherChannel Protocols

### LACP — Link Aggregation Control Protocol
- Defined by **IEEE 802.3ad** — open standard, works with all vendors
- Adds LACPDU frames to negotiate EtherChannel

| LACP Mode | Behaviour |
|---|---|
| `active` | Actively initiates LACP negotiation |
| `passive` | Only responds to LACP — does not initiate |

> 💡 At least one side must be `active` for LACP to form.

### PAgP — Port Aggregation Protocol
- **Cisco proprietary** — only works between Cisco devices

| PAgP Mode | Behaviour |
|---|---|
| `desirable` | Actively initiates PAgP negotiation |
| `auto` | Only responds to PAgP — does not initiate |

> 💡 At least one side must be `desirable` for PAgP to form.

---

## 🧪 Lab Tasks

### Topology
```
[SW1] ── eth0/0, eth0/1, eth0/2 (bundled) ── [SW2]
```

- **Task 3.1** — Configure EtherChannel on SW1 (ports eth0/0–0/2)
- **Task 3.2** — Configure EtherChannel on SW2 (ports eth0/0–0/2)
- **Task 3.3** — Configure the bundled link as a trunk

---

## ✍️ Solution

### Task 3.1 — SW1 EtherChannel (LACP)

```cisco
sw1> enable
sw1# configure terminal

sw1(config)# interface range ethernet 0/0-2
sw1(config-if-range)# channel-protocol lacp
sw1(config-if-range)# channel-group 1 mode active
! Output: Creating a port-channel interface Port-channel 1
sw1(config-if-range)# exit
```

### Task 3.2 — SW2 EtherChannel (LACP)

```cisco
sw2> enable
sw2# configure terminal

sw2(config)# interface range ethernet 0/0-2
sw2(config-if-range)# channel-protocol lacp
sw2(config-if-range)# channel-group 1 mode active
sw2(config-if-range)# exit
```

### Task 3.3 — Configure Port-Channel as Trunk

#### On SW1
```cisco
! Option A: Configure directly on the port-channel interface (recommended)
sw1(config)# interface port-channel 1
sw1(config-if)# switchport trunk encapsulation dot1q
sw1(config-if)# switchport mode trunk
sw1(config-if)# exit

! Option B: Configure on the physical range
sw1(config)# interface range ethernet 0/0-2
sw1(config-if-range)# switchport trunk encapsulation dot1q
sw1(config-if-range)# switchport mode trunk
sw1(config-if-range)# exit
```

#### On SW2
```cisco
sw2(config)# interface port-channel 1
sw2(config-if)# switchport trunk encapsulation dot1q
sw2(config-if)# switchport mode trunk
sw2(config-if)# exit
```

---

## 📋 PAgP Alternative Configuration

```cisco
! SW1 — PAgP Desirable (active initiator)
sw1(config)# interface range ethernet 0/0-2
sw1(config-if-range)# channel-protocol pagp
sw1(config-if-range)# channel-group 1 mode desirable
sw1(config-if-range)# exit

! SW2 — PAgP Auto (responds only)
sw2(config)# interface range ethernet 0/0-2
sw2(config-if-range)# channel-protocol pagp
sw2(config-if-range)# channel-group 1 mode auto
sw2(config-if-range)# exit
```

---

## ✅ Verification Commands

```cisco
! View EtherChannel summary
Switch# show etherchannel summary

! View detailed EtherChannel info
Switch# show etherchannel detail

! View port-channel interface status
Switch# show interfaces port-channel 1

! Check if trunk is active on the port-channel
Switch# show interfaces port-channel 1 trunk

! View physical member ports
Switch# show etherchannel port-channel
```

### Expected `show etherchannel summary` Output
```
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Et0/0(P)  Et0/1(P)  Et0/2(P)
```

`SU` = Layer **2**, in **U**se. `P` next to each port = bundled and active.

---

## ⚠️ EtherChannel Requirements

All member ports must have identical configuration:
- [ ] Same speed and duplex
- [ ] Same VLAN assignment (or all trunk)
- [ ] Same trunk encapsulation type
- [ ] Same allowed VLANs

If any setting mismatches, the port will be suspended and won't bundle.

---

## 📘 Key Takeaways

- EtherChannel increases bandwidth and provides link redundancy
- LACP (802.3ad) works with all vendors; PAgP is Cisco-only
- At least one side of the negotiation must be in active/desirable mode
- Configure trunk settings on the port-channel interface, not individual ports
- All physical member ports must have identical configuration
- STP sees the port-channel as a single link — no blocking occurs on member ports
