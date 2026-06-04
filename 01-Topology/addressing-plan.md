Based on the finalized topology, this is the version I'd save as `01-Topology/addressing-plan.md`.

# Addressing Plan

## Project

Cisco-NXOS-VXLAN-EVPN-Fabric-Lab

---

# Topology

```text
                  Spine1
                 /      \
                /        \
               /          \
           Leaf1          Leaf2
               \          /
                \        /
                 \      /
                  Spine2
```

---

# Device Roles

| Device  | Role               |
| ------- | ------------------ |
| Spine1  | Spine              |
| Spine2  | Spine              |
| Leaf1   | Leaf / VTEP        |
| Leaf2   | Leaf / VTEP        |
| Border1 | Future Border Leaf |
| Border2 | Future Border Leaf |

---

# Loopback Addressing

## Router IDs

| Device  | Interface | IP Address     |
| ------- | --------- | -------------- |
| Spine1  | Lo0       | 1.1.1.1/32     |
| Spine2  | Lo0       | 2.2.2.2/32     |
| Leaf1   | Lo0       | 11.11.11.11/32 |
| Leaf2   | Lo0       | 22.22.22.22/32 |
| Border1 | Lo0       | 33.33.33.33/32 |
| Border2 | Lo0       | 44.44.44.44/32 |

Purpose:

* Router-ID
* BGP Router-ID

---

# Underlay Point-to-Point Links

## Link 1

| Device | Interface   | IP Address   |
| ------ | ----------- | ------------ |
| Leaf1  | Ethernet1/1 | 10.0.11.0/31 |
| Spine1 | Ethernet1/2 | 10.0.11.1/31 |

Network:

```text
10.0.11.0/31
```

---

## Link 2

| Device | Interface   | IP Address   |
| ------ | ----------- | ------------ |
| Leaf1  | Ethernet1/3 | 10.0.12.0/31 |
| Spine2 | Ethernet1/2 | 10.0.12.1/31 |

Network:

```text
10.0.12.0/31
```

---

## Link 3

| Device | Interface   | IP Address   |
| ------ | ----------- | ------------ |
| Leaf2  | Ethernet1/2 | 10.0.21.0/31 |
| Spine1 | Ethernet1/1 | 10.0.21.1/31 |

Network:

```text
10.0.21.0/31
```

---

## Link 4

| Device | Interface   | IP Address   |
| ------ | ----------- | ------------ |
| Leaf2  | Ethernet1/4 | 10.0.22.0/31 |
| Spine2 | Ethernet1/3 | 10.0.22.1/31 |

Network:

```text
10.0.22.0/31
```

---

# BGP Autonomous System Numbers

| Device  | ASN   |
| ------- | ----- |
| Spine1  | 65000 |
| Spine2  | 65000 |
| Leaf1   | 65101 |
| Leaf2   | 65102 |
| Border1 | 65103 |
| Border2 | 65104 |

---

# VXLAN Planning

## VTEP Loopbacks (Future)

| Device  | Interface | IP Address         |
| ------- | --------- | ------------------ |
| Leaf1   | Lo1       | 111.111.111.111/32 |
| Leaf2   | Lo1       | 222.222.222.222/32 |
| Border1 | Lo1       | 133.133.133.133/32 |
| Border2 | Lo1       | 144.144.144.144/32 |

Purpose:

* NVE Source Interface
* VTEP Address

---

# Layer 2 VNI Planning

| VLAN | VNI   | Tenant   |
| ---- | ----- | -------- |
| 10   | 10010 | Tenant-A |
| 20   | 10020 | Tenant-B |

---

# Layer 3 VNI Planning

| VRF      | L3VNI |
| -------- | ----- |
| Tenant-A | 50001 |
| Tenant-B | 50002 |

---

# Software

| Item     | Value                  |
| -------- | ---------------------- |
| Platform | Cisco Nexus 9000v      |
| Image    | nxos64-cs.10.5.2.F.bin |

---

# Design Notes

* Underlay routing protocol: eBGP
* Overlay control plane: BGP EVPN
* Data plane: VXLAN
* Point-to-point links use /31 addressing
* Border1 and Border2 are reserved for future phases
* Initial deployment consists of Spine1, Spine2, Leaf1 and Leaf2 only

One note for tomorrow: before configuring VXLAN, we'll create a separate `interface-plan.md` and `base-config.md` so every interface in the topology has a documented purpose. That's how we'll keep this lab looking like a real deployment project rather than just a collection of configs.
