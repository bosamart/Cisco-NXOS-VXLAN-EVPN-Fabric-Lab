# CONCEPTS.md

# VXLAN EVPN Concepts

This document explains the fundamental concepts used in the Cisco NX-OS VXLAN EVPN Fabric Lab.

---

# Traditional Network Problem

Traditional Layer 2 networks rely on VLANs and Spanning Tree Protocol (STP).

Challenges:

* Limited to ~4094 VLANs
* Blocked links due to STP
* Large Layer 2 failure domains
* Difficult workload mobility

Modern data centers require:

* Scale
* Multi-tenancy
* Workload mobility
* ECMP forwarding

VXLAN EVPN solves these challenges.

---

# VXLAN

VXLAN (Virtual eXtensible LAN) is an overlay technology.

It extends Layer 2 networks across a Layer 3 IP fabric.

Instead of carrying Ethernet frames directly across switches, VXLAN encapsulates them inside UDP packets.

```text
Original Ethernet Frame
        ↓
VXLAN Encapsulation
        ↓
UDP
        ↓
IP
        ↓
Underlay Network
```

VXLAN uses UDP port:

```text
4789
```

---

# VNI

VNI = VXLAN Network Identifier

Equivalent to a VLAN in traditional networks.

```text
VLAN 100
   ↓
VNI 10100
```

Benefits:

* 24-bit identifier
* Approximately 16 million segments

Compared to:

```text
VLAN
12-bit
≈ 4094 segments
```

---

# VTEP

VTEP = VXLAN Tunnel Endpoint

A VTEP performs:

* Encapsulation
* Decapsulation

Functions:

```text
Ethernet → VXLAN
VXLAN → Ethernet
```

In this lab:

```text
Leaf1 = VTEP
Leaf2 = VTEP
```

---

# NVE Interface

NVE = Network Virtualization Edge

Logical VXLAN tunnel interface.

Example:

```bash
interface nve1
 source-interface loopback1
 host-reachability protocol bgp
```

Think of NVE as:

```text
Tunnel Interface for VXLAN
```

---

# Underlay

The underlay is the IP fabric.

Responsibilities:

* Reachability between VTEPs
* ECMP forwarding
* Fast convergence

The underlay knows:

```text
VTEP IP Addresses
```

The underlay does NOT know:

```text
MAC Addresses
VNIs
Hosts
Tenants
```

---

# Overlay

The overlay is VXLAN EVPN.

Responsibilities:

* Host reachability
* Tenant separation
* Layer 2 extension
* Layer 3 services

The overlay knows:

```text
MAC Addresses
IP Addresses
VNIs
VRFs
```

---

# EVPN

EVPN = Ethernet VPN

EVPN is the control plane for VXLAN.

VXLAN provides:

```text
Data Plane
```

EVPN provides:

```text
Control Plane
```

EVPN distributes:

* MAC addresses
* IP addresses
* VTEP information
* IP prefixes

using MP-BGP.

---

# MP-BGP EVPN

BGP distributes overlay information between VTEPs.

Benefits:

* Reduced flooding
* Host mobility support
* Scalable control plane
* Faster learning

---

# L2VNI

L2VNI represents a Layer 2 segment.

Example:

```text
VLAN 10
  ↓
L2VNI 10010
```

Endpoints within the same L2VNI communicate at Layer 2.

---

# L3VNI

L3VNI represents a tenant routing domain.

Each VRF is associated with an L3VNI.

Example:

```text
Tenant-A
VRF Tenant-A
L3VNI 50000
```

L3VNI enables routing between L2VNIs.

---

# VRF

VRF = Virtual Routing and Forwarding

Provides Layer 3 separation.

Example:

```text
Tenant-A
10.1.1.0/24

Tenant-B
10.1.1.0/24
```

Overlapping addressing becomes possible.

---

# Anycast Gateway

Every leaf switch provides the same default gateway.

Example:

```text
Gateway IP:
192.168.10.1

Gateway MAC:
0000.2222.3333
```

Benefits:

* Local routing
* Host mobility
* Optimal forwarding

Hosts always use the same gateway regardless of location.

---

# EVPN Route Types

## Type-2

Advertises:

* MAC Address
* IP Address

Purpose:

```text
Where is this host?
```

Example:

```text
192.168.10.11
↓
VTEP 10.1.1.2
```

---

## Type-3

IMET Route

Advertises:

* VTEP membership

Purpose:

```text
Which VTEPs belong to this VNI?
```

Used for:

* Broadcast
* Unknown Unicast
* Multicast

(BUM Traffic)

---

## Type-5

IP Prefix Route

Advertises:

* IP Prefixes

Purpose:

```text
How do I reach this subnet?
```

Commonly used with:

* Border Leafs
* External Routing
* Tenant Prefix Advertisement

---

# Packet Walk Summary

Known Unicast:

```text
Host A
 ↓
Type-2 Lookup
 ↓
Destination VTEP
 ↓
VXLAN Encapsulation
 ↓
Underlay Routing
 ↓
Remote VTEP
 ↓
Destination Host
```

BUM Traffic:

```text
Host A
 ↓
Type-3 Lookup
 ↓
Replication List
 ↓
Remote VTEPs
```

---

# Key Interview Answers

Q: What is VXLAN?

A: VXLAN is an overlay technology that extends Layer 2 networks across a Layer 3 IP fabric using UDP encapsulation.

Q: What is EVPN?

A: EVPN is the control plane used by VXLAN to distribute MAC, IP, VTEP, and route information using MP-BGP.

Q: Difference between VTEP and NVE?

A:

* VTEP = Device performing encapsulation/decapsulation
* NVE = Logical VXLAN tunnel interface

Q: Difference between Type-2 and Type-3?

A:

* Type-2 = Host reachability
* Type-3 = VTEP membership and BUM replication

Q: What does the underlay know?

A:

Only VTEP IP addresses and routing information.

Q: What does the overlay know?

A:

MAC addresses, IP addresses, VNIs, VRFs, and endpoint location information.
