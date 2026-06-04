# 02-Underlay.md

## Objective

Build a stable IP underlay network for VXLAN EVPN.

The underlay is responsible for:

* IP reachability between all switches
* Reachability to Loopback0 (Router-ID)
* Reachability to Loopback1 (VTEP)
* Transporting VXLAN traffic between VTEPs

---

# Topology

```text
                 Spine1
                /      \
               /        \
              /          \
             /            \
          Leaf1          Leaf2
             \            /
              \          /
               \        /
                \      /
                 Spine2
```

---

# Device Roles

| Device | Role  | ASN   |
| ------ | ----- | ----- |
| Spine1 | Spine | 65000 |
| Spine2 | Spine | 65000 |
| Leaf1  | Leaf  | 65101 |
| Leaf2  | Leaf  | 65102 |

---

# Addressing Plan

## Loopback0 (Router-ID)

| Device | Address        |
| ------ | -------------- |
| Spine1 | 1.1.1.1/32     |
| Spine2 | 2.2.2.2/32     |
| Leaf1  | 11.11.11.11/32 |
| Leaf2  | 22.22.22.22/32 |

---

## Loopback1 (VTEP)

| Device | Address            |
| ------ | ------------------ |
| Leaf1  | 111.111.111.111/32 |
| Leaf2  | 222.222.222.222/32 |

---

## Point-to-Point Links

| Link           | Network      |
| -------------- | ------------ |
| Leaf1 ↔ Spine1 | 10.0.11.0/31 |
| Leaf1 ↔ Spine2 | 10.0.12.0/31 |
| Leaf2 ↔ Spine1 | 10.0.21.0/31 |
| Leaf2 ↔ Spine2 | 10.0.22.0/31 |

---

# Design Decision

## Underlay Protocol

eBGP IPv4 Unicast

Reason:

* Simple configuration
* No OSPF required
* No IS-IS required
* Common modern Clos fabric design
* Fast convergence
* Scales well

---

# Base Configuration

## Common Features

```bash
feature bgp
```

---

# Spine Configuration

## Spine1

```bash
router bgp 65000
  router-id 1.1.1.1

  address-family ipv4 unicast
    network 1.1.1.1/32

  neighbor 10.0.11.0
    remote-as 65101
    address-family ipv4 unicast

  neighbor 10.0.21.0
    remote-as 65102
    address-family ipv4 unicast
```

---

## Spine2

```bash
router bgp 65000
  router-id 2.2.2.2

  address-family ipv4 unicast
    network 2.2.2.2/32

  neighbor 10.0.12.0
    remote-as 65101
    address-family ipv4 unicast

  neighbor 10.0.22.0
    remote-as 65102
    address-family ipv4 unicast
```

---

# Leaf Configuration

## Leaf1

```bash
router bgp 65101
  router-id 11.11.11.11

  address-family ipv4 unicast
    network 11.11.11.11/32
    network 111.111.111.111/32

  neighbor 10.0.11.1
    remote-as 65000
    address-family ipv4 unicast

  neighbor 10.0.12.1
    remote-as 65000
    address-family ipv4 unicast
```

---

## Leaf2

```bash
router bgp 65102
  router-id 22.22.22.22

  address-family ipv4 unicast
    network 22.22.22.22/32
    network 222.222.222.222/32

  neighbor 10.0.21.1
    remote-as 65000
    address-family ipv4 unicast

  neighbor 10.0.22.1
    remote-as 65000
    address-family ipv4 unicast
```

---

# Verification

## Verify BGP Neighbors

```bash
show bgp ipv4 unicast summary
```

Expected:

```text
Leaf1 -> Spine1 Established
Leaf1 -> Spine2 Established

Leaf2 -> Spine1 Established
Leaf2 -> Spine2 Established
```

---

## Verify Router-ID Reachability

### Leaf1

```bash
show ip route 1.1.1.1
show ip route 2.2.2.2
show ip route 22.22.22.22
```

---

### Leaf2

```bash
show ip route 1.1.1.1
show ip route 2.2.2.2
show ip route 11.11.11.11
```

---

## Verify VTEP Reachability

### Leaf1

```bash
show ip route 222.222.222.222
```

Expected:

```text
Route learned through BGP
```

---

### Leaf2

```bash
show ip route 111.111.111.111
```

Expected:

```text
Route learned through BGP
```

---

## Verify Complete Routing Table

```bash
show ip route bgp
```

Expected:

```text
Loopback0 routes present
Loopback1 routes present
```

---

# Validation Results

Successfully validated:

```text
✓ eBGP Underlay Established

✓ Router-ID Reachability

✓ VTEP Reachability

✓ Spine1 Reachable

✓ Spine2 Reachable

✓ Leaf1 Reachable

✓ Leaf2 Reachable
```

---

# Notes

The Underlay does NOT carry:

```text
MAC Addresses
VXLAN VNIs
EVPN Route Types
```

Those functions belong to:

```text
03-EVPN
```

The Underlay only provides IP connectivity between VTEPs.

---

# Next Phase

```text
03-EVPN
├── 01-VTEP-Loopback
├── 02-EVPN-Control-Plane
└── 03-NVE
```
