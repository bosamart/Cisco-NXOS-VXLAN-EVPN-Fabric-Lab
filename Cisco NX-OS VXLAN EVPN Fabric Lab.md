# Cisco NX-OS VXLAN EVPN Fabric Lab (Working Design)

## Topology

```
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

# Design

## Underlay

* OSPF Area 0
* Point-to-Point links
* ECMP enabled automatically by OSPF
* Loopback0 advertised
* Loopback1 advertised

## Overlay

* iBGP EVPN
* All switches in AS65000
* Spine1 = Route Reflector
* Spine2 = Route Reflector
* Leaf1 and Leaf2 = RR Clients

## Data Plane

* VXLAN
* Ingress Replication (IR)
* VLAN 10 → VNI 10010

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

# EVPN Configuration

## VNI

```
VLAN 10
VNI 10010
RT 10010:10010
RD Auto
```

---

# Verification Checklist

## Underlay

Verify OSPF neighbors:

```bash
show ip ospf neighbor
```

Verify loopback reachability:

```bash
show ip route ospf
```

Expected:

```
1.1.1.1
2.2.2.2
11.11.11.11
22.22.22.22
111.111.111.111
222.222.222.222
```

reachable from every switch.

---

# EVPN Verification

Verify EVPN neighbors:

```bash
show bgp l2vpn evpn summary
```

Expected:

```
PfxRcd = 2
```

from both route reflectors.

---

Verify Type-2 routes:

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```
Host MACs learned locally and remotely
```

---

Verify Type-3 routes:

```bash
show bgp l2vpn evpn route-type 3
```

Expected:

```
Tunnel Type = Ingress Replication
```

---

# VXLAN Verification

Verify VNI:

```bash
show nve vni
```

Expected:

```
VNI 10010
State Up
```

---

Verify NVE peers:

```bash
show nve peers
```

Expected:

Leaf1

```
222.222.222.222
```

Leaf2

```
111.111.111.111
```

---

Verify MAC learning:

```bash
show mac address-table dynamic
```

Expected on Leaf1:

```
Local:
0050.0000.0700

Remote:
0050.0000.0800
```

Expected on Leaf2:

```
Local:
0050.0000.0800

Remote:
0050.0000.0700
```

---

Verify EVPN MAC database:

```bash
show l2route evpn mac all
```

Expected:

Leaf1

```
0050.0000.0800
Next-Hop 222.222.222.222
```

Leaf2

```
0050.0000.0700
Next-Hop 111.111.111.111
```

---

# Host Configuration

Host1

```bash
ip addr add 192.168.10.11/24 dev eth0
ip link set eth0 up
```

Host2

```bash
ip addr add 192.168.10.22/24 dev eth0
ip link set eth0 up
```

No default gateway required for pure L2VXLAN testing.

---

# End-to-End Test

From Host1:

```bash
ping 192.168.10.22
```

From Host2:

```bash
ping 192.168.10.11
```

Expected:

```
Success
```

---

# Lessons Learned

## Failed Design

```
OSPF Underlay
eBGP EVPN Overlay
```

Symptoms observed:

* Strange NVE peer behavior
* Remote MACs pointing toward spine
* Dataplane inconsistencies
* Difficult troubleshooting

---

## Working Design

```
OSPF Underlay
iBGP EVPN Overlay
Spine Route Reflectors
```

Results:

* Clean EVPN control plane
* Correct remote VTEP learning
* Correct MAC learning
* Stable VXLAN dataplane
* Successful host-to-host communication

This is the recommended lab design for learning Cisco VXLAN EVPN and aligns closely with Cisco reference architectures.
