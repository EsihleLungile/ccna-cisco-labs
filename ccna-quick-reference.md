# 📘 CCNA Quick Reference — Concepts & Commands

A consolidated reference of key CCNA concepts and the most important show/debug commands for verification and troubleshooting.

---

## 🔷 VLAN & Trunking

```cisco
show vlan brief                          ! List all VLANs and their ports
show interfaces trunk                    ! Show trunk ports and allowed VLANs
show interfaces fa0/1 switchport         ! Detailed switchport info
show interfaces fa0/1 trunk              ! Trunk status for one port
show dtp                                 ! DTP negotiation info
show running-config interface fa0/1      ! Interface config
```

---

## 🔷 VTP

```cisco
show vtp status                          ! Mode, domain, revision number
show vtp password                        ! VTP password
show vtp counters                        ! VTP message stats
```

---

## 🔷 EtherChannel

```cisco
show etherchannel summary                ! Group status and member ports
show etherchannel detail                 ! Full detail per group
show interfaces port-channel 1           ! Port-channel interface status
show etherchannel port-channel           ! Member port detail
```

---

## 🔷 Spanning Tree

```cisco
show spanning-tree                       ! Full STP info for all VLANs
show spanning-tree vlan 10               ! STP for a specific VLAN
show spanning-tree summary               ! Summary of all instances
show spanning-tree interface fa0/1 detail ! Per-interface STP detail
show spanning-tree interface fa0/1 portfast ! Check PortFast status
```

---

## 🔷 Routing

```cisco
show ip route                            ! Full routing table
show ip route static                     ! Static routes only
show ip route ospf                       ! OSPF routes only
show ip route eigrp                      ! EIGRP routes only
show ip protocols                        ! Routing protocols running
show ip interface brief                  ! All interfaces and IPs
```

---

## 🔷 OSPF

```cisco
show ip ospf neighbor                    ! OSPF neighbour relationships
show ip ospf database                    ! Link-State Database
show ip ospf interface                   ! OSPF interface detail
show ip ospf                             ! OSPF process info
```

---

## 🔷 EIGRP

```cisco
show ip eigrp neighbors                  ! EIGRP neighbour table
show ip eigrp topology                   ! EIGRP topology table (successors/feasibles)
show ip eigrp interfaces                 ! EIGRP interface detail
show ip eigrp traffic                    ! EIGRP packet statistics
```

---

## 🔷 ACL

```cisco
show access-lists                        ! All ACLs with hit counts
show ip access-lists                     ! IP ACLs only
show ip interface fa0/1                  ! ACLs applied to interface
```

---

## 🔷 NAT

```cisco
show ip nat translations                 ! Active NAT/PAT sessions
show ip nat statistics                   ! NAT stats (inside/outside interfaces)
clear ip nat translation *               ! Clear NAT table
debug ip nat                             ! Real-time NAT debug
```

---

## 🔷 DHCP

```cisco
show ip dhcp pool                        ! DHCP pools configured
show ip dhcp binding                     ! Active DHCP leases
show ip dhcp conflict                    ! IP conflicts detected
show ip dhcp server statistics           ! DHCP server stats
clear ip dhcp binding *                  ! Clear all DHCP leases
```

---

## 🔷 Security & Passwords

```cisco
show users                               ! Active console/VTY users
show line                                ! Line configuration summary
show ssh                                 ! Active SSH sessions
show crypto key mypubkey rsa             ! SSH RSA key info
show port-security                       ! Port security status
show port-security interface fa0/1       ! Port security per interface
show port-security address               ! Secure MAC addresses
```

---

## 🔷 General Troubleshooting

```cisco
show version                             ! IOS version, uptime, memory
show running-config                      ! Current configuration
show startup-config                      ! Saved configuration
show interfaces                          ! Full interface stats
show interfaces fa0/1                    ! One interface detail
show controllers                         ! Physical interface info
show cdp neighbors                       ! Directly connected Cisco devices
show cdp neighbors detail                ! IP addresses of neighbours
show lldp neighbors                      ! LLDP neighbour info
show clock                               ! Current time
show ntp status                          ! NTP sync status
show logging                             ! Syslog messages
```

---

## 🔷 Useful Debug Commands

> ⚠️ Debug commands can impact router performance. Use in lab environments or during off-peak hours. Always disable with `no debug all` when done.

```cisco
debug ip rip                             ! RIP updates
debug ip ospf events                     ! OSPF events
debug ip eigrp                           ! EIGRP updates
debug ip nat                             ! NAT translations in real time
debug spanning-tree events               ! STP topology changes
no debug all                             ! Stop all debugging
undebug all                              ! Same as above
```

---

## 🔷 Cisco IOS Mode Reference

| Mode | Prompt | Access |
|---|---|---|
| User EXEC | `Router>` | Default on login |
| Privileged EXEC | `Router#` | `enable` |
| Global Config | `Router(config)#` | `configure terminal` |
| Interface Config | `Router(config-if)#` | `interface <type><num>` |
| Router Config | `Router(config-router)#` | `router ospf 1` |
| VLAN Config | `Switch(config-vlan)#` | `vlan <id>` |
| Line Config | `Router(config-line)#` | `line vty 0 4` |

---

## 🔷 Save & Manage Configuration

```cisco
! Save running config to startup config
Router# write memory
! or
Router# copy running-config startup-config

! Erase startup config (factory reset)
Router# write erase
Router# reload

! Backup config to TFTP server
Router# copy running-config tftp

! View differences between running and startup
Router# show archive config differences
```
