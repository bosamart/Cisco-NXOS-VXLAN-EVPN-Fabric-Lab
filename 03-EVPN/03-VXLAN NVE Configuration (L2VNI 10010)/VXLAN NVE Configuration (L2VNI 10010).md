# VXLAN NVE Configuration (L2VNI 10010)

## Design

```text
VLAN 10
    ↓
VNI 10010
    ↓
VXLAN Tunnel
    ↓
Remote VTEP
```

Data Plane:

```text
Host1
  ↓
Leaf1
  ↓ VXLAN Encapsulation
IP Underlay
  ↓ VXLAN Decapsulation
Leaf2
  ↓
Host2
```

---

# VLAN Configuration

## Leaf1

```bash
vlan 10
  name USERS
  vn-segment 10010
```

---

## Leaf2

```bash
vlan 10
  name USERS
  vn-segment 10010
```

---

# EVPN VNI Mapping

## Leaf1

```bash
evpn
  vni 10010 l2
    rd auto
    route-target import 10010:10010
    route-target export 10010:10010
```

---

## Leaf2

```bash
evpn
  vni 10010 l2
    rd auto
    route-target import 10010:10010
    route-target export 10010:10010
```

---

# NVE Interface

## Leaf1

```bash
feature nv overlay

interface nve1
  no shutdown
  host-reachability protocol bgp

  source-interface loopback1

  member vni 10010
    ingress-replication protocol bgp
```

---

## Leaf2

```bash
feature nv overlay

interface nve1
  no shutdown
  host-reachability protocol bgp

  source-interface loopback1

  member vni 10010
    ingress-replication protocol bgp
```

---

# Access Ports

## Leaf1

```bash
interface Ethernet1/10
  description HOST1
  switchport
  switchport mode access
  switchport access vlan 10
  spanning-tree port type edge
  no shutdown
```

---

## Leaf2

```bash
interface Ethernet1/10
  description HOST2
  switchport
  switchport mode access
  switchport access vlan 10
  spanning-tree port type edge
  no shutdown
```

---

# VTEP Loopbacks

## Leaf1

```bash
interface loopback1
  description VXLAN-VTEP
  ip address 111.111.111.111/32
```

---

## Leaf2

```bash
interface loopback1
  description VXLAN-VTEP
  ip address 222.222.222.222/32
```

---

# Verification

## Verify NVE Interface

```bash
show interface nve1
```

Expected:

```text
nve1 is up
Encapsulation VXLAN
```

---

## Verify VNI State

```bash
show nve vni
```

Expected:

```text
nve1 10010 Up CP L2
```

Example:

```text
Interface VNI    State Mode Type
--------- ------ ----- ---- ----
nve1      10010  Up    CP   L2
```

---

## Verify NVE Peers

```bash
show nve peers
```

Expected:

Leaf1

```text
222.222.222.222
```

Leaf2

```text
111.111.111.111
```

Peer State:

```text
Up
```

Learn Type:

```text
CP
```

(Control Plane learned)

---

## Verify EVPN MAC Learning

```bash
show l2route evpn mac all
```

Expected on Leaf1:

```text
0050.0000.0700 Local
0050.0000.0800 BGP
```

Expected on Leaf2:

```text
0050.0000.0800 Local
0050.0000.0700 BGP
```

---

## Verify MAC Table

```bash
show mac address-table dynamic
```

Expected on Leaf1:

```text
0050.0000.0700 Eth1/10
0050.0000.0800 nve1(222.222.222.222)
```

Expected on Leaf2:

```text
0050.0000.0800 Eth1/10
0050.0000.0700 nve1(111.111.111.111)
```

---

## Verify Type-2 Routes

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
MAC routes learned from remote VTEP
```

Example:

```text
0050.0000.0700
0050.0000.0800
```

---

## Verify Type-3 Routes

```bash
show bgp l2vpn evpn route-type 3
```

Expected:

```text
Tunnel Type: Ingress Replication
```

and

```text
Tunnel Id:
111.111.111.111
222.222.222.222
```

---

# End-to-End Testing

## Host1

```bash
ip addr add 192.168.10.11/24 dev eth0
ip link set eth0 up
```

---

## Host2

```bash
ip addr add 192.168.10.22/24 dev eth0
ip link set eth0 up
```

---

## Ping Test

Host1

```bash
ping 192.168.10.22
```

Host2

```bash
ping 192.168.10.11
```

Expected:

```text
Success
```

---

# Key Learning Points

## Type-2 Route

Advertises:

```text
MAC Address
VNI
VTEP Information
```

Example:

```text
0050.0000.0800
```

---

## Type-3 Route (IMET)

Advertises:

```text
VTEP Membership
BUM Replication Information
```

Example:

```text
Tunnel Id: 222.222.222.222
```

---

## NVE

```text
NVE = Network Virtualization Edge
```

Logical VXLAN tunnel interface.

---

## VTEP

```text
VTEP = VXLAN Tunnel Endpoint
```

Leaf switch performing:

```text
Encapsulation
Decapsulation
```

using Loopback1 as source.

---

## Final Result

```text
OSPF Underlay              ✓
iBGP EVPN Overlay          ✓
Route Reflectors           ✓
Type-2 Routes              ✓
Type-3 Routes              ✓
NVE Peering                ✓
Remote MAC Learning        ✓
VXLAN Dataplane            ✓
Host-to-Host Connectivity  ✓
```
