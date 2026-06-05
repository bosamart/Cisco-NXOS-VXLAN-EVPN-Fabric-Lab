# 01-VTEP-Loopback.md

## Objective

Configure VTEP source addresses used by VXLAN encapsulation and decapsulation.

The VTEP IP address is assigned to Loopback1 on each leaf switch.

---

# What is a VTEP?

VTEP stands for:

```text
VXLAN Tunnel Endpoint
```

Functions:

* Encapsulates Ethernet frames into VXLAN packets
* Decapsulates VXLAN packets back into Ethernet frames
* Acts as the source and destination of VXLAN tunnels

---

# VTEP Design

In this lab:

```text
Leaf1 = VTEP
Leaf2 = VTEP

Spine1 = Non-VTEP
Spine2 = Non-VTEP
```

The spines provide IP transport only.

---

# VTEP Addressing

| Device | Loopback1          | Purpose     |
| ------ | ------------------ | ----------- |
| Leaf1  | 111.111.111.111/32 | VTEP Source |
| Leaf2  | 222.222.222.222/32 | VTEP Source |

---

# Configuration

## Leaf1

```bash
interface loopback1
  description VXLAN-VTEP-SOURCE
  ip address 111.111.111.111/32
```

---

## Leaf2

```bash
interface loopback1
  description VXLAN-VTEP-SOURCE
  ip address 222.222.222.222/32
```

---

# Advertise VTEP Loopback

The VTEP address must be reachable through the underlay.

---

## Leaf1

```bash
router bgp 65101

  address-family ipv4 unicast
    network 111.111.111.111/32
```

---

## Leaf2

```bash
router bgp 65102

  address-family ipv4 unicast
    network 222.222.222.222/32
```

---

# Verification

## Verify Loopback Interface

### Leaf1

```bash
show ip interface brief | include Lo1
```

Expected:

```text
Lo1    up    up
```

---

### Leaf2

```bash
show ip interface brief | include Lo1
```

Expected:

```text
Lo1    up    up
```

---

# Verify Local Route

### Leaf1

```bash
show ip route 111.111.111.111
```

Expected:

```text
attached
local
```

---

### Leaf2

```bash
show ip route 222.222.222.222
```

Expected:

```text
attached
local
```

---

# Verify Remote VTEP Reachability

### Leaf1

```bash
show ip route 222.222.222.222
```

Expected:

```text
Learned via BGP
```

---

### Leaf2

```bash
show ip route 111.111.111.111
```

Expected:

```text
Learned via BGP
```

---

# Verify Spine Reachability

### Spine1

```bash
show ip route 111.111.111.111
show ip route 222.222.222.222
```

Expected:

```text
Both VTEP routes present
```

---

### Spine2

```bash
show ip route 111.111.111.111
show ip route 222.222.222.222
```

Expected:

```text
Both VTEP routes present
```

---

# Validation Results

Successfully validated:

```text
✓ Loopback1 Configured

✓ VTEP Address Advertised

✓ Underlay Reachability Verified

✓ Spine1 Learns Both VTEPs

✓ Spine2 Learns Both VTEPs

✓ Leaf1 Reaches Leaf2 VTEP

✓ Leaf2 Reaches Leaf1 VTEP
```

---

# Why This Step Matters

Later, when NVE1 is configured:

```bash
interface nve1
  source-interface loopback1
```

VXLAN packets will use:

```text
Leaf1
111.111.111.111

Leaf2
222.222.222.222
```

as the tunnel endpoints.

Without VTEP loopback reachability:

```text
NVE Tunnel = Down
VXLAN = Broken
EVPN = No Data Forwarding
```

---

# Next Phase

```text
03-EVPN
├── 01-VTEP-Loopback
✓

├── 02-EVPN-Control-Plane
NEXT

└── 03-NVE
```
