# Cisco NX-OS VXLAN EVPN Fabric Lab

A hands-on Cisco Nexus VXLAN EVPN fabric built in EVE-NG to learn modern Data Center networking concepts including Spine-Leaf architecture, VXLAN overlays, MP-BGP EVPN control plane, VTEP operation, and end-to-end host connectivity.

---

## Project Overview

This lab was built to understand how VXLAN EVPN replaces traditional Layer 2 designs by extending Layer 2 segments across a Layer 3 fabric while maintaining scalability and operational simplicity.

### Technologies Used

* Cisco Nexus NX-OS
* VXLAN
* MP-BGP EVPN
* OSPF Underlay
* Route Reflectors
* Ingress Replication (IR)
* EVE-NG

### Learning Objectives

* Build a Spine-Leaf Data Center Fabric
* Configure OSPF Underlay Routing
* Deploy MP-BGP EVPN Overlay
* Configure VTEPs and NVE Interfaces
* Implement VXLAN Layer 2 Extension
* Understand EVPN Route Types
* Verify End-to-End VXLAN Connectivity

---

# Topology

```text
                 Spine1 (RR)
                Lo0 1.1.1.1
                      |
                      |
      -------------------------------
      |                             |
      |                             |
   Leaf1                         Leaf2
 Lo0 11.11.11.11              Lo0 22.22.22.22
 Lo1 111.111.111.111          Lo1 222.222.222.222
      |                             |
      -------------------------------
                      |
                 Spine2 (RR)
                Lo0 2.2.2.2
```

---

# Architecture

## Underlay

The underlay provides IP connectivity between VTEPs.

Features:

* OSPF Area 0
* Point-to-Point Routed Links
* ECMP Forwarding
* Loopback Reachability

The underlay only carries transport traffic and does not learn tenant MAC addresses.

---

## Overlay

The overlay uses MP-BGP EVPN to distribute endpoint reachability information.

Features:

* iBGP EVPN
* Route Reflectors on Spines
* EVPN Type-2 Routes
* EVPN Type-3 Routes

### BGP Design

```text
AS65000
```

Spines:

```text
Spine1 = Route Reflector
Spine2 = Route Reflector
```

Leaves:

```text
Leaf1 = RR Client
Leaf2 = RR Client
```

---

## VXLAN Data Plane

VXLAN provides Layer 2 extension across the Layer 3 fabric.

Features:

* VLAN 10
* VNI 10010
* Ingress Replication
* NVE Interface
* VTEP Encapsulation / Decapsulation

---

# IP Addressing

## Loopbacks

| Device | Lo0            | Lo1                |
| ------ | -------------- | ------------------ |
| Spine1 | 1.1.1.1/32     | N/A                |
| Spine2 | 2.2.2.2/32     | N/A                |
| Leaf1  | 11.11.11.11/32 | 111.111.111.111/32 |
| Leaf2  | 22.22.22.22/32 | 222.222.222.222/32 |

---

## Underlay Links

| Link           | Network      |
| -------------- | ------------ |
| Leaf1 ↔ Spine1 | 10.0.11.0/31 |
| Leaf1 ↔ Spine2 | 10.0.12.0/31 |
| Leaf2 ↔ Spine1 | 10.0.21.0/31 |
| Leaf2 ↔ Spine2 | 10.0.22.0/31 |

---

# VXLAN EVPN Configuration

## VLAN and VNI Mapping

```text
VLAN 10
↓
VNI 10010
```

### EVPN Parameters

```text
RD Auto
RT 10010:10010
```

---

# Verification Checklist

## Verify OSPF Underlay

```bash
show ip ospf neighbor
```

Verify loopback reachability:

```bash
show ip route ospf
```

Expected routes:

```text
1.1.1.1
2.2.2.2
11.11.11.11
22.22.22.22
111.111.111.111
222.222.222.222
```

---

## Verify EVPN Neighbors

```bash
show bgp l2vpn evpn summary
```

Expected:

```text
Established
```

---

## Verify EVPN Type-2 Routes

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
Local and Remote MAC/IP advertisements
```

---

## Verify EVPN Type-3 Routes

```bash
show bgp l2vpn evpn route-type 3
```

Expected:

```text
Ingress Replication information
```

---

## Verify VNI State

```bash
show nve vni
```

Expected:

```text
VNI 10010
State Up
```

---

## Verify NVE Peers

```bash
show nve peers
```

Expected:

Leaf1:

```text
222.222.222.222
```

Leaf2:

```text
111.111.111.111
```

---

## Verify MAC Learning

```bash
show l2route evpn mac all
```

Expected:

Leaf1:

```text
0050.0000.0800
Next-Hop 222.222.222.222
```

Leaf2:

```text
0050.0000.0700
Next-Hop 111.111.111.111
```

---

# Host Configuration

## Host1

```bash
ip addr add 192.168.10.11/24 dev eth0
ip link set eth0 up
```

## Host2

```bash
ip addr add 192.168.10.22/24 dev eth0
ip link set eth0 up
```

No default gateway is required because this lab validates Layer 2 VXLAN forwarding.

---

# End-to-End Validation

From Host1:

```bash
ping 192.168.10.22
```

From Host2:

```bash
ping 192.168.10.11
```

Expected:

```text
Success
```

---

# Lessons Learned

## Initial Design

```text
OSPF Underlay
eBGP EVPN Overlay
```

Observed Issues:

* Unexpected NVE behavior
* Incorrect remote MAC learning
* Difficult troubleshooting
* Inconsistent dataplane results

---

## Final Working Design

```text
OSPF Underlay
iBGP EVPN Overlay
Spine Route Reflectors
```

Benefits:

* Stable EVPN control plane
* Correct remote VTEP learning
* Proper MAC advertisement
* Predictable VXLAN forwarding
* Successful end-to-end communication

---

# Key Concepts

* VXLAN = Data Plane
* EVPN = Control Plane
* VTEP = VXLAN Tunnel Endpoint
* NVE = Logical VXLAN Tunnel Interface
* Type-2 = Host Reachability
* Type-3 = BUM Replication
* Underlay = VTEP Reachability
* Overlay = Endpoint Reachability

---

# Future Enhancements

* L3VNI
* Anycast Gateway
* Inter-VLAN Routing
* EVPN Type-5 Routes
* Border Leaf Connectivity
* Multi-Pod VXLAN
* Multi-Site VXLAN

---

# References

* Cisco Nexus VXLAN EVPN Configuration Guide
* Cisco VXLAN BGP EVPN Design and Implementation Guide
* Cisco NX-OS Documentation
