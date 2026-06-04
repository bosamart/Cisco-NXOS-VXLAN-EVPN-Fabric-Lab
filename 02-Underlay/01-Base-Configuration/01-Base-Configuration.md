# 01-Base-Configuration.md

## Objective

Prepare all Nexus switches with the minimum configuration required before building the VXLAN EVPN fabric.

---

# Device Naming

| Device | Hostname |
| ------ | -------- |
| Spine1 | Spine1   |
| Spine2 | Spine2   |
| Leaf1  | Leaf1    |
| Leaf2  | Leaf2    |

---

# Features

## Spine1 / Spine2

```bash
feature bgp
```

## Leaf1 / Leaf2

```bash
feature bgp
feature nv overlay
feature vn-segment-vlan-based

nv overlay evpn
```

---

# Loopback0 (Router-ID)

| Device | Address        |
| ------ | -------------- |
| Spine1 | 1.1.1.1/32     |
| Spine2 | 2.2.2.2/32     |
| Leaf1  | 11.11.11.11/32 |
| Leaf2  | 22.22.22.22/32 |

---

# Loopback1 (VTEP)

| Device | Address            |
| ------ | ------------------ |
| Leaf1  | 111.111.111.111/32 |
| Leaf2  | 222.222.222.222/32 |

---

# Interface Descriptions

Use descriptions on all links.

Example:

```bash
description TO-SPINE1
description TO-SPINE2
description TO-LEAF1
description TO-LEAF2
```

---

# Save Configuration

```bash
copy running-config startup-config
```
