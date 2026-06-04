# 02-EVPN-Control-Plane.md

## Objective

Establish the EVPN control plane between Leaf and Spine switches using BGP EVPN.

At the end of this phase:

* EVPN capability is negotiated between all Leafs and Spines
* Underlay BGP remains operational
* VTEP loopbacks are reachable
* EVPN address-family is active
* Fabric is ready for NVE and VNI configuration

---

# Topology

```text
                 Spine1 (AS65000)
                /               \
               /                 \
              /                   \
             /                     \
            /                       \
 Leaf1 (AS65101)               Leaf2 (AS65102)
            \                       /
             \                     /
              \                   /
               \                 /
                \               /
                 Spine2 (AS65000)
```

---

# Design

## Underlay

eBGP IPv4 Unicast

| Device | ASN   |
| ------ | ----- |
| Spine1 | 65000 |
| Spine2 | 65000 |
| Leaf1  | 65101 |
| Leaf2  | 65102 |

---

## Overlay

eBGP EVPN

Existing underlay neighbors are reused for EVPN.

No Route Reflectors are used.

No additional loopback EVPN sessions are required.

---

# Prerequisites

The following must already be working:

* eBGP Underlay
* Loopback0 Reachability
* VTEP Loopback Reachability

## VTEP Addresses

| Device | Loopback1          |
| ------ | ------------------ |
| Leaf1  | 111.111.111.111/32 |
| Leaf2  | 222.222.222.222/32 |

---

# Feature Configuration

## Leaf1

```bash
feature bgp
feature nv overlay
feature vn-segment-vlan-based

nv overlay evpn
```

## Leaf2

```bash
feature bgp
feature nv overlay
feature vn-segment-vlan-based

nv overlay evpn
```

## Spine1

```bash
feature bgp
feature nv overlay

nv overlay evpn
```

## Spine2

```bash
feature bgp
feature nv overlay

nv overlay evpn
```

---

# EVPN Neighbor Configuration

## Leaf1

```bash
router bgp 65101

  neighbor 10.0.11.1
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 10.0.12.1
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

## Leaf2

```bash
router bgp 65102

  neighbor 10.0.21.1
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 10.0.22.1
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

## Spine1

```bash
router bgp 65000

  neighbor 10.0.11.0
    address-family l2vpn evpn

  neighbor 10.0.21.0
    address-family l2vpn evpn
```

---

## Spine2

```bash
router bgp 65000

  neighbor 10.0.12.0
    address-family l2vpn evpn

  neighbor 10.0.22.0
    address-family l2vpn evpn
```

---

# Verification

## Verify EVPN Neighbors

```bash
show bgp l2vpn evpn summary
```

Expected:

```text
capable peers 2
```

on all devices.

---

## Verify BGP Configuration

```bash
show run bgp
```

Confirm:

```text
address-family l2vpn evpn
```

exists under all EVPN neighbors.

---

## Verify VTEP Reachability

### Leaf1

```bash
show ip route 222.222.222.222
```

Expected:

```text
Route learned via BGP
```

---

### Leaf2

```bash
show ip route 111.111.111.111
```

Expected:

```text
Route learned via BGP
```

---

### Spine1

```bash
show ip route 111.111.111.111
show ip route 222.222.222.222
```

Expected:

```text
Both VTEP loopbacks reachable
```

---

### Spine2

```bash
show ip route 111.111.111.111
show ip route 222.222.222.222
```

Expected:

```text
Both VTEP loopbacks reachable
```

---

# Validation Results

EVPN capability successfully negotiated:

```text
Leaf1  <-> Spine1
Leaf1  <-> Spine2

Leaf2  <-> Spine1
Leaf2  <-> Spine2
```

All devices report:

```text
capable peers 2
```

---

# Notes

PfxRcd may remain:

```text
0
```

at this stage.

This is expected because:

* NVE interface not configured
* VNI not configured
* VLAN-to-VNI mapping not configured
* No EVPN routes exist yet

The EVPN control plane is operational and ready for the next phase.

---

# Next Phase

03-EVPN

```text
03-NVE
```

Tasks:

1. Create NVE1
2. Configure source-interface Loopback1
3. Configure VNIs
4. Configure VLAN-to-VNI mapping
5. Verify EVPN Route Type-3 advertisements

```
```
