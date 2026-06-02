# Static Routing, OSPF and PAT (Simulated Internet Access)

**Tools Used:** Cisco Packet Tracer  
**Topics Covered:** Static Routes, OSPF, PAT (NAT Overload), Routing Tables  
**Related Certification:** CompTIA Network+ (N10-009)

---

## Overview

In this lab, I connected two routers together over a WAN link and used OSPF to automatically share routes between them. I also configured PAT (Port Address Translation) on R1 so that a device on a private LAN could access a simulated internet address.

To simulate internet connectivity, I created a loopback interface on R2 with the IP address `8.8.8.8`.

The goal was to get a PC on the `192.168.1.0/24` network to successfully ping `8.8.8.8` while learning how routing and NAT work together.

---

## What I Learned

During this lab I practised:

- Configuring router interfaces
- Setting up OSPF between two routers
- Understanding OSPF wildcard masks
- Creating and advertising a default route
- Configuring PAT (NAT overload)
- Checking routing tables and NAT translations
- Troubleshooting connectivity using show commands

---

## Network Topology

```text
[ PC1 ] — [ SW1 ] — [ R1 ] —— WAN (10.0.0.0/30) —— [ R2 ]
              LAN                OSPF Area 0              |
          192.168.1.0/24      PAT on g0/1            Loopback0
                                                      8.8.8.8/32
```

---

## IP Addressing Plan

| Device | Interface | IP Address | Purpose |
|----------|-----------|------------|----------|
| PC1 | NIC | 192.168.1.10/24 | Client device |
| R1 | G0/0 | 192.168.1.1/24 | Default gateway |
| R1 | G0/1 | 10.0.0.1/30 | WAN connection |
| R2 | G0/0 | 10.0.0.2/30 | WAN connection |
| R2 | Loopback0 | 8.8.8.8/32 | Simulated internet host |

---

## Configuring OSPF

This was my first time working with OSPF in a small network. Instead of manually creating routes on each router, OSPF allows routers to automatically share information about the networks they know about.

After configuring OSPF on both routers, I verified that they formed a neighbour relationship using:

```cisco
show ip ospf neighbor
```

When the neighbour state changed to `FULL`, I knew the routers were successfully exchanging routing information.

---

## Configuring PAT

I then configured PAT on R1.

My understanding is that PAT allows multiple devices on a private network to share a single public IP address. In this lab there was only one PC, but the configuration would still work if more devices were added later.

The `overload` keyword enables PAT by using port numbers to keep track of different connections.

---

## Verification

To confirm everything was working correctly, I checked:

### OSPF Neighbours

```cisco
show ip ospf neighbor
```

### Routing Table

```cisco
show ip route
```

### NAT Translations

```cisco
show ip nat translations
```

The NAT table was empty until traffic was generated from the PC.

---

## Connectivity Test

From PC1 I ran:

```cmd
ping 8.8.8.8
```

The ping was successful, which confirmed:

- OSPF was exchanging routes correctly
- The default route was working
- PAT was translating traffic as expected
- End-to-end connectivity had been established

---

## Key Takeaways

This lab helped me understand how routing and NAT work together in a small network.

Before completing it, I understood the theory behind OSPF and PAT, but configuring them and verifying the results made the concepts much clearer. Seeing the routing table update automatically through OSPF and watching NAT translations appear after sending traffic helped connect the theory to what actually happens on a network.

---

## Files Included

| File | Description |
|--------|------------|
| `lab2-ospf-pat.pkt` | Cisco Packet Tracer lab file |
| `screenshots/` | Screenshots of the topology, verification commands and successful ping test |
