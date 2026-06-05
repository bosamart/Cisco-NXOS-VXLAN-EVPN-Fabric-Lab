# 4-EVPN-Control-Plane.md

## Objective

Build the VXLAN EVPN control plane using BGP EVPN Route Reflectors.

At the end of this phase:

* OSPF underlay is operational
* Loopback reachability exists between all devices
* EVPN BGP sessions are established
* No NVE interface yet
* No VXLAN tunnel yet
* No host connectivity yet

---

## Topology

```text
                 Spine1
               Lo0 1.1.1.1
                    |
                    |
      ------------------------------
      |                            |
      |                            |
    Leaf1                        Leaf2
 Lo0 11.11.11.11             Lo0 22.22.22.22
      |                            |
      |                            |
      ------------------------------
                    |
                    |
                 Spine2
               Lo0 2.2.2.2
```

---

## Design

### Underlay

* OSPF Area 0
* Advertise Loopback0
* Advertise all leaf-spine links

### Overlay

* EVPN over BGP
* Spine1 = Route Reflector
* Spine2 = Route Reflector
* Leafs = RR Clients

---

## Leaf1 Configuration

```bash
router bgp 65101
  router-id 11.11.11.11

  address-family l2vpn evpn

  neighbor 1.1.1.1
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 2.2.2.2
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

## Leaf2 Configuration

```bash
router bgp 65102
  router-id 22.22.22.22

  address-family l2vpn evpn

  neighbor 1.1.1.1
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended

  neighbor 2.2.2.2
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
```

---

## Spine1 Configuration

```bash
router bgp 65000
  router-id 1.1.1.1

  address-family l2vpn evpn

  neighbor 11.11.11.11
    remote-as 65101
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended

  neighbor 22.22.22.22
    remote-as 65102
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended
```

---

## Spine2 Configuration

```bash
router bgp 65000
  router-id 2.2.2.2

  address-family l2vpn evpn

  neighbor 11.11.11.11
    remote-as 65101
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended

  neighbor 22.22.22.22
    remote-as 65102
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      route-reflector-client
      send-community
      send-community extended
```

---

## Verification

### Verify EVPN Neighbors

```bash
show bgp l2vpn evpn summary
```

Expected:

```text
Leaf1
  1.1.1.1 Established
  2.2.2.2 Established

Leaf2
  1.1.1.1 Established
  2.2.2.2 Established

Spine1
  11.11.11.11 Established
  22.22.22.22 Established

Spine2
  11.11.11.11 Established
  22.22.22.22 Established
```

---

## Verification Commands

```bash
show bgp l2vpn evpn summary
show bgp l2vpn evpn
show bgp l2vpn evpn neighbors
```

---

## Current Status

```text
OSPF Underlay        COMPLETE
EVPN Control Plane   COMPLETE
NVE Interface        NOT CONFIGURED
VXLAN Tunnel         NOT CONFIGURED
VLAN-VNI Mapping     NOT CONFIGURED
Host Connectivity    NOT CONFIGURED
```

---

## Next Phase

4-NVE-and-VTEP.md

* Create Loopback1 (VTEP IP)
* Configure NVE1
* Configure VNI 10010
* Verify Type-3 IMET routes
* Verify NVE Peers

```
```
