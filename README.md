# Cisco NX-OS VXLAN EVPN Fabric Lab

## Overview

This repository documents the design, configuration, verification, and troubleshooting of a Cisco Nexus VXLAN EVPN fabric built in EVE-NG.

The purpose of this project is to learn and demonstrate modern Data Center networking concepts including:

* Spine-Leaf Architecture
* VXLAN Overlay Networking
* MP-BGP EVPN Control Plane
* L2VNI and L3VNI
* Distributed Anycast Gateway
* Symmetric IRB
* Tenant Segmentation with VRFs

---

## Lab Topology

```text
                 +---------+
                 | Spine1  |
                 +---------+
                  /       \
                 /         \
                /           \
       +---------+       +---------+
       | Leaf1   |       | Leaf2   |
       +---------+       +---------+
            |                 |
         Host-A            Host-B
```

---

## Project Objectives

* Build a VXLAN EVPN fabric using Cisco NX-OS
* Understand the separation of underlay and overlay networks
* Configure BGP EVPN as the VXLAN control plane
* Implement Layer 2 and Layer 3 services
* Verify host-to-host communication across the fabric
* Learn EVPN route types and packet forwarding behavior

---

## Technologies Used

| Technology      | Purpose                     |
| --------------- | --------------------------- |
| VXLAN           | Overlay Data Plane          |
| EVPN            | Overlay Control Plane       |
| MP-BGP          | Route Distribution          |
| OSPF / IS-IS    | Underlay Routing            |
| Anycast Gateway | Distributed Default Gateway |
| VRF             | Tenant Segmentation         |
| ECMP            | Load Balancing              |

---

## Features Implemented

### Underlay

* Spine-Leaf IP Fabric
* Loopback Reachability
* ECMP Routing

### Overlay

* VTEP Configuration
* NVE Interface
* VXLAN Encapsulation
* MP-BGP EVPN

### Tenant Services

* L2VNI
* L3VNI
* VRF
* Anycast Gateway

### Verification

* BGP EVPN Neighbor Establishment
* NVE Peer Discovery
* End-to-End Host Connectivity
* EVPN Route Learning

---

## Repository Structure

```text
Cisco-NXOS-VXLAN-EVPN-Fabric-Lab
│
├── README.md
├── CONCEPTS.md
├── PACKET-WALKS.md
├── TROUBLESHOOTING.md
│
├── 01-Topology
├── 02-Underlay
├── 03-Overlay
├── 04-Verification
└── configs
```

---

## Verification Commands

```bash
show bgp l2vpn evpn summary
show bgp l2vpn evpn
show nve peers
show nve vni
show l2route evpn mac all
show forwarding vxlan tunnel
```

---

## Key Concepts

See:

* CONCEPTS.md
* PACKET-WALKS.md
* TROUBLESHOOTING.md

---

## Current Status

### Completed

* Underlay Fabric
* BGP EVPN Control Plane
* L2VNI
* L3VNI
* Anycast Gateway
* End-to-End Connectivity

### Planned Enhancements

* EVPN Type-5 Routes
* Border Leaf Connectivity
* External Routing
* vPC Anycast VTEP
* Multi-Pod VXLAN
* Multi-Site VXLAN

---

## Author

Sam Ath Bo

Network Engineer focused on:

* MPLS
* Data Center Networking
* VXLAN EVPN
* Cisco Technologies
* EVE-NG Labs

GitHub:
https://github.com/bosamart/Cisco-NXOS-VXLAN-EVPN-Fabric-Lab
