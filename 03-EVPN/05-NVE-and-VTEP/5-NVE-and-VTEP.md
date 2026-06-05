# 5-NVE-and-VTEP-Ready-Config.md

## Prerequisites

Verify:

```bash
show bgp l2vpn evpn summary
```

All EVPN neighbors must be Established before continuing.

---

# Leaf1 Complete Configuration

## Loopback1 (VTEP)

```bash
conf t

interface loopback1
  description VXLAN-VTEP
  ip address 111.111.111.111/32
  ip router ospf UNDERLAY area 0.0.0.0

end
```

---

## VLAN and VNI Mapping

```bash
conf t

vlan 10
  name USERS
  vn-segment 10010

end
```

---

## NVE Interface

```bash
conf t

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1

  member vni 10010
    ingress-replication protocol bgp

end
```

---

## EVPN VNI Binding

```bash
conf t

evpn
  vni 10010 l2
    rd auto
    route-target import 10010:10010
    route-target export 10010:10010

end
```

---

# Leaf2 Complete Configuration

## Loopback1 (VTEP)

```bash
conf t

interface loopback1
  description VXLAN-VTEP
  ip address 222.222.222.222/32
  ip router ospf UNDERLAY area 0.0.0.0

end
```

---

## VLAN and VNI Mapping

```bash
conf t

vlan 10
  name USERS
  vn-segment 10010

end
```

---

## NVE Interface

```bash
conf t

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1

  member vni 10010
    ingress-replication protocol bgp

end
```

---

## EVPN VNI Binding

```bash
conf t

evpn
  vni 10010 l2
    rd auto
    route-target import 10010:10010
    route-target export 10010:10010

end
```

---

# Verification Step 1

Verify VTEP Reachability

## Leaf1

```bash
show ip route 222.222.222.222
ping 222.222.222.222 source 111.111.111.111
```

Expected:

```text
Success
```

---

## Leaf2

```bash
show ip route 111.111.111.111
ping 111.111.111.111 source 222.222.222.222
```

Expected:

```text
Success
```

---

# Verification Step 2

Verify Type-3 Routes

```bash
show bgp l2vpn evpn route-type 3
```

Expected:

Leaf1 learns:

```text
222.222.222.222
```

Leaf2 learns:

```text
111.111.111.111
```

---

# Verification Step 3

Verify VNI

```bash
show nve vni
```

Expected:

```text
Interface  VNI    State  Type
nve1       10010  Up     L2
```

---

# Verification Step 4

Verify NVE Peers

## Leaf1

```bash
show nve peers
```

Expected:

```text
222.222.222.222 Up
```

---

## Leaf2

```bash
show nve peers
```

Expected:

```text
111.111.111.111 Up
```

---

# Success Criteria

```text
EVPN Sessions          UP
Loopback1 Reachable    YES
Type-3 Routes          Present
NVE Interface          UP
VNI 10010              UP
NVE Peers              UP
```

---

# Next File

6-L2VXLAN-End-to-End.md

```
```
