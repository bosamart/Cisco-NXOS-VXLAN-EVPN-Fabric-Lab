# 02-EVPN-Control-Plane.md

# Objective

Establish the EVPN control plane between Leaf and Spine switches using BGP EVPN.

This phase is responsible for:

* EVPN route exchange
* VTEP discovery
* IMET (Type-3) route advertisement
* Route Target (RT) import/export
* Building the VXLAN control plane

No MAC learning or host traffic is tested in this phase.

---

# Topology

## Underlay

eBGP IPv4 Underlay

| Device | ASN   |
| ------ | ----- |
| Spine1 | 65000 |
| Spine2 | 65000 |
| Leaf1  | 65101 |
| Leaf2  | 65102 |

Underlay provides reachability to:

* Loopback0 (BGP Router-ID)
* Loopback1 (VTEP)

---

# Loopbacks

## Leaf1

| Interface | Address            |
| --------- | ------------------ |
| Lo0       | 11.11.11.11/32     |
| Lo1       | 111.111.111.111/32 |

## Leaf2

| Interface | Address            |
| --------- | ------------------ |
| Lo0       | 22.22.22.22/32     |
| Lo1       | 222.222.222.222/32 |

## Spine1

| Interface | Address    |
| --------- | ---------- |
| Lo0       | 1.1.1.1/32 |

## Spine2

| Interface | Address    |
| --------- | ---------- |
| Lo0       | 2.2.2.2/32 |

---

# Design Decision

Initial testing used EVPN peering across directly connected underlay links.

Example:

```text
Leaf1 -> 10.0.11.1
Leaf1 -> 10.0.12.1

Leaf2 -> 10.0.21.1
Leaf2 -> 10.0.22.1
```

The final design moved EVPN to loopback-based overlay peering.

Benefits:

* Independent underlay and overlay
* Better scalability
* Closer to production EVPN deployments
* Consistent VTEP reachability

---

# EVPN Overlay Peering

## Leaf1

```cisco
neighbor 1.1.1.1
  remote-as 65000
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
    send-community
    send-community extended

neighbor 2.2.2.2
  remote-as 65000
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
    send-community
    send-community extended
```

## Leaf2

```cisco
neighbor 1.1.1.1
  remote-as 65000
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
    send-community
    send-community extended

neighbor 2.2.2.2
  remote-as 65000
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
    send-community
    send-community extended
```

## Spine1

```cisco
neighbor 11.11.11.11
  remote-as 65101
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn

neighbor 22.22.22.22
  remote-as 65102
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
```

## Spine2

```cisco
neighbor 11.11.11.11
  remote-as 65101
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn

neighbor 22.22.22.22
  remote-as 65102
  update-source loopback0
  ebgp-multihop 2
  address-family l2vpn evpn
```

---

# EVPN VNI Configuration

Configured on both leaves.

```cisco
evpn
  vni 10010 l2
    rd auto
    route-target import 10010:10010
    route-target export 10010:10010
```

---

# Route Target Strategy

Explicit Route Target configuration was used.

```text
Import RT = 10010:10010
Export RT = 10010:10010
```

This guarantees both leaves import EVPN routes into the same VNI.

---

# Retain Route Target

Configured on all EVPN speakers.

```cisco
router bgp <ASN>

address-family l2vpn evpn
  retain route-target all
```

Purpose:

* Retain EVPN routes even before local VNI import evaluation
* Simplifies EVPN route visibility and troubleshooting

---

# Verification

## EVPN Neighbors

```cisco
show bgp l2vpn evpn summary
```

Expected:

```text
Leaf1
  1.1.1.1 Up
  2.2.2.2 Up

Leaf2
  1.1.1.1 Up
  2.2.2.2 Up
```

---

## Type-3 Routes

```cisco
show bgp l2vpn evpn route-type 3
```

Expected:

```text
Local IMET Route
Remote IMET Route
```

Example:

```text
111.111.111.111
222.222.222.222
```

---

## EVPN Route Import

Expected output:

```text
Imported paths list: L2-10010
```

This confirms RT import into the local VNI.

---

# Success Criteria

The control plane is considered operational when:

* EVPN neighbors are Established
* Type-3 routes are exchanged
* Remote VTEPs are learned
* EVPN routes are imported into L2RIB

Verification:

```cisco
show bgp l2vpn evpn
show bgp l2vpn evpn route-type 3
show nve peers
```

Expected:

```text
Leaf1 learns VTEP 222.222.222.222
Leaf2 learns VTEP 111.111.111.111
```

---

# Result

Control plane operational.

Working:

* EVPN BGP Sessions
* Type-3 IMET Routes
* Route Target Import
* L2RIB Import
* Dynamic VTEP Discovery

Design Note

Because the EVPN fabric uses multiple ASNs
(65101, 65102, 65000), explicit Route Targets
are configured:

RT Import: 10010:10010
RT Export: 10010:10010

This guarantees consistent EVPN route import
across all VTEPs regardless of local ASN.


Next phase:

03-NVE

Verify VXLAN tunnel establishment and NVE peer state.
