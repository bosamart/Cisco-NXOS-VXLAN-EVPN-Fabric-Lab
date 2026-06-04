# 03-Verification.md

## Verify Interface Status

```bash
show ip interface brief
show interface description
```

Expected:

```text
All links Up/Up
```

---

# Verify BGP Adjacencies

```bash
show bgp ipv4 unicast summary
```

Expected:

## Leaf1

```text
10.0.11.1 Established
10.0.12.1 Established
```

## Leaf2

```text
10.0.21.1 Established
10.0.22.1 Established
```

---

# Verify Router-ID Reachability

## Leaf1

```bash
show ip route 1.1.1.1
show ip route 2.2.2.2
show ip route 22.22.22.22
```

---

## Leaf2

```bash
show ip route 1.1.1.1
show ip route 2.2.2.2
show ip route 11.11.11.11
```

---

# Verify VTEP Reachability

## Leaf1

```bash
show ip route 222.222.222.222
```

Expected:

```text
Route learned via BGP
```

---

## Leaf2

```bash
show ip route 111.111.111.111
```

Expected:

```text
Route learned via BGP
```

---

# Verify BGP Learned Routes

```bash
show ip route bgp
```

Expected:

```text
1.1.1.1/32
2.2.2.2/32
11.11.11.11/32
22.22.22.22/32
111.111.111.111/32
222.222.222.222/32
```

---

# Underlay Success Criteria

```text
✓ All interfaces Up

✓ All BGP neighbors Established

✓ Loopback0 Reachability Verified

✓ Loopback1 Reachability Verified

✓ Underlay Ready for EVPN
```
