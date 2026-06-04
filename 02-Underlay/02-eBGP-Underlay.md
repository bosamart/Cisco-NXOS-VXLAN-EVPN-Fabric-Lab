# 02-eBGP-Underlay.md

## Objective

Provide IP connectivity between all switches using eBGP IPv4 Unicast.

---

# ASN Assignment

| Device | ASN   |
| ------ | ----- |
| Spine1 | 65000 |
| Spine2 | 65000 |
| Leaf1  | 65101 |
| Leaf2  | 65102 |

---

# Point-to-Point Links

| Link           | Network      |
| -------------- | ------------ |
| Leaf1 ↔ Spine1 | 10.0.11.0/31 |
| Leaf1 ↔ Spine2 | 10.0.12.0/31 |
| Leaf2 ↔ Spine1 | 10.0.21.0/31 |
| Leaf2 ↔ Spine2 | 10.0.22.0/31 |

---

# Spine1

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

# Spine2

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

# Leaf1

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

# Leaf2

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

# Expected Result

```text
All eBGP neighbors Established

Loopback0 Reachable

Loopback1 Reachable
```
