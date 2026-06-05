# 6-L2VXLAN-End-to-End.md

## Objective

Validate Layer-2 VXLAN bridging between two hosts connected to different leaf switches.

At the end of this phase:

* Local MAC learning works
* EVPN Type-2 routes are exchanged
* Remote MAC learning works
* NVE peer relationship is established
* End-to-end ping succeeds

---

# Topology

```text
+-------------+                  +-------------+
|    VPC1     |                  |    VPC2     |
|10.10.10.11  |                  |10.10.10.12  |
+------+------+
       |                                |
       |                                |
    Leaf1                            Leaf2
       |                                |
       +--------- VXLAN EVPN ----------+
```

---

# Step 1 - Connect Hosts

## Leaf1

```bash
interface ethernet1/10
  switchport
  switchport mode access
  switchport access vlan 10
  no shutdown
```

---

## Leaf2

```bash
interface ethernet1/10
  switchport
  switchport mode access
  switchport access vlan 10
  no shutdown
```

---

# Step 2 - Configure VPCS

## VPC1

```bash
ip 10.10.10.11/24
save
```

---

## VPC2

```bash
ip 10.10.10.12/24
save
```

---

# Step 3 - Generate Traffic

From VPC1:

```bash
ping 10.10.10.12
```

First ping may fail.

This is normal.

The fabric must learn MAC addresses first.

---

# Verification 1 - Local MAC Learning

## Leaf1

```bash
show mac address-table dynamic
```

Expected:

```text
VLAN 10
0050.xxxx.xxxx
Eth1/10
```

---

## Leaf2

```bash
show mac address-table dynamic
```

Expected:

```text
VLAN 10
0050.xxxx.xxxx
Eth1/10
```

---

# Verification 2 - EVPN Type-2 Routes

## Leaf1

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
Remote MAC from Leaf2
```

---

## Leaf2

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
Remote MAC from Leaf1
```

---

# Verification 3 - EVPN MAC Database

## Leaf1

```bash
show l2route evpn mac all
```

Expected:

```text
Local MAC
Remote MAC
```

---

## Leaf2

```bash
show l2route evpn mac all
```

Expected:

```text
Local MAC
Remote MAC
```

---

# Verification 4 - NVE Peers

## Leaf1

```bash
show nve peers
```

Expected:

```text
222.222.222.222 Up
```

---

## Leaf2

```bash
show nve peers
```

Expected:

```text
111.111.111.111 Up
```

---

# Verification 5 - NVE VNI

```bash
show nve vni
```

Expected:

```text
VNI 10010
State Up
Type L2
```

---

# Verification 6 - End-to-End Ping

## VPC1

```bash
ping 10.10.10.12
```

Expected:

```text
Success rate 100%
```

---

## VPC2

```bash
ping 10.10.10.11
```

Expected:

```text
Success rate 100%
```

---

# Troubleshooting Commands

## Underlay

```bash
show ip ospf neighbor
show ip route
show ip route 111.111.111.111
show ip route 222.222.222.222
```

---

## EVPN Control Plane

```bash
show bgp l2vpn evpn summary
show bgp l2vpn evpn route-type 2
show bgp l2vpn evpn route-type 3
```

---

## VXLAN Data Plane

```bash
show nve peers
show nve peers detail
show nve vni
show l2route evpn mac all
show mac address-table dynamic
```

---

# Success Criteria

```text
OSPF Neighbor            UP
EVPN Neighbor            UP
Type-3 Routes            Present
NVE Peer                 UP
Type-2 Routes            Present
Remote MAC Learned       YES
VPC1 <-> VPC2 Ping       SUCCESS
```

---

# Lab Complete

You now have a working:

```text
OSPF Underlay
+
BGP EVPN Control Plane
+
VXLAN Data Plane
+
Layer-2 VNI (10010)
+
End-to-End Host Connectivity
```

---

# Next Labs

7-Distributed-Anycast-Gateway.md

Topics:

* SVI VLAN10
* Anycast Gateway MAC
* Anycast Gateway IP
* Symmetric IRB
* L3VNI
* Inter-VLAN Routing over VXLAN

```
```
