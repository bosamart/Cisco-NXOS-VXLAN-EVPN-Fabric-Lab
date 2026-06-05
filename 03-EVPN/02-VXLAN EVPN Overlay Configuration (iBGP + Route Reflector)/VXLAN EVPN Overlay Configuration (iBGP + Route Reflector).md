# VXLAN EVPN Overlay Configuration (iBGP + Route Reflector)

## Design

```text
Underlay:
  OSPF Area 0

Overlay:
  iBGP EVPN

AS Number:
  65000

Spines:
  Route Reflectors

Leaves:
  RR Clients
```

---

# Spine1

```bash
router bgp 65000
  router-id 1.1.1.1

  address-family l2vpn evpn

  neighbor 11.11.11.11
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended

  neighbor 22.22.22.22
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended
```

---

# Spine2

```bash
router bgp 65000
  router-id 2.2.2.2

  address-family l2vpn evpn

  neighbor 11.11.11.11
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended

  neighbor 22.22.22.22
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended
```

---

# Leaf1

```bash
router bgp 65000
  router-id 11.11.11.11

  address-family l2vpn evpn

  neighbor 1.1.1.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 2.2.2.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

# Leaf2

```bash
router bgp 65000
  router-id 22.22.22.22

  address-family l2vpn evpn

  neighbor 1.1.1.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 2.2.2.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

# Verification

## BGP EVPN Sessions

```bash
show bgp l2vpn evpn summary
```

Expected:

```text
Leaf1 -> Spine1 Established
Leaf1 -> Spine2 Established

Leaf2 -> Spine1 Established
Leaf2 -> Spine2 Established
```

---

## Verify Route Reflection

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
Originator: 22.22.22.22
Cluster list: 1.1.1.1
```

or

```text
Originator: 11.11.11.11
Cluster list: 1.1.1.1
```

This confirms the route was reflected by the spine RR.

---

## Verify Type-2 Routes

```bash
show bgp l2vpn evpn route-type 2
```

Expected:

```text
Remote MAC routes present
```

Example:

```text
0050.0000.0800
0050.0000.0700
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

and remote VTEP loopbacks:

```text
111.111.111.111
222.222.222.222
```

---

## Verify NVE Peers

```bash
show nve peers
```

Expected:

```text
111.111.111.111
222.222.222.222
```

present and Up.

---

## Verify Remote MAC Programming

```bash
show l2route evpn mac all
```

Expected:

Leaf1

```text
0050.0000.0800
Next-Hop 222.222.222.222
```

Leaf2

```text
0050.0000.0700
Next-Hop 111.111.111.111
```

This confirms EVPN control plane correctly programmed the VXLAN dataplane.

---

# Key Learning Points

* OSPF provides underlay reachability.
* iBGP EVPN provides the overlay control plane.
* Spines act as Route Reflectors.
* Leaves never peer directly with each other.
* Type-2 routes advertise MAC addresses.
* Type-3 routes advertise VTEP reachability (IMET routes).
* NVE peers are discovered automatically through EVPN.
* VXLAN dataplane is built using information learned from EVPN routes.
