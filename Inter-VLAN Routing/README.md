# VLAN Segmentation and Inter-VLAN Routing

**Tools Used:** Cisco Packet Tracer
**Topics Covered:** VLANs, Trunking, Router-on-a-Stick, 802.1Q Tagging
**Related Certification:** CompTIA Network+ (N10-009)

---

## Overview

In this lab, I created two separate VLANs across two switches and configured a router to allow communication between them using a router-on-a-stick setup.

The VLANs represent different departments on a network, with devices in VLAN 10 and VLAN 20 placed into separate broadcast domains. By default, devices in different VLANs cannot communicate with each other, so I used router sub-interfaces and 802.1Q tagging to enable routing between the networks.

The goal was to allow a device in VLAN 20 to successfully communicate with a device in VLAN 10 while maintaining VLAN separation.

---

## What I Learned

During this lab I practised:

* Creating and naming VLANs
* Assigning switch ports to VLANs
* Configuring trunk links between switches
* Understanding 802.1Q VLAN tagging
* Configuring router-on-a-stick routing
* Creating router sub-interfaces
* Testing communication between VLANs
* Verifying VLAN and trunk configurations

---

## Network Topology

```text
              [    R1   ]
                   |
                   | (Trunk)
                   |

              [   SW1   ]
              /          \
       (Access)            \ (Trunk)
     /       \               \ 
   PC0     PC1          [   SW2   ]
  VLAN10   VLAN20       /         \
                       PC2          PC3
                     VLAN10        VLAN20
```

---

## IP Addressing Plan

| Device     | VLAN | IP Address       | Default Gateway |
| ---------- | ---- | ---------------- | --------------- |
| PC0        | 10   | 192.168.10.10/24 | 192.168.10.1    |
| PC1        | 20   | 192.168.20.10/24 | 192.168.20.1    |
| PC2        | 10   | 192.168.10.20/24 | 192.168.10.1    |
| PC3        | 20   | 192.168.20.20/24 | 192.168.20.1    |
| R0 G0/0.10 | 10   | 192.168.10.1/24  | N/A             |
| R1 G0/0.20 | 20   | 192.168.20.1/24  | N/A             |

---

## VLAN Configuration

I started by creating VLAN 10 and VLAN 20 on both switches and assigning the access ports connected to the PCs.

The access ports were configured so that:

* PC0 and PC2 belonged to VLAN 10
* PC1 and PC3 belonged to VLAN 20

This separated the devices into different logical networks even though they were connected to the same switching infrastructure.

---

## Trunk Configuration

To allow VLAN traffic to travel between switches and reach the router, I configured trunk links using 802.1Q tagging.

The trunk connections were:

* SW0 to SW1
* SW0 to R0

These links carry traffic for multiple VLANs across a single cable.

---

## Router-on-a-Stick Configuration

To allow communication between VLANs, I configured router sub-interfaces on R1.

Each VLAN was assigned its own logical interface:

| Sub-Interface | VLAN | Gateway Address |
| ------------- | ---- | --------------- |
| G0/0.10       | 10   | 192.168.10.1    |
| G0/0.20       | 20   | 192.168.20.1    |

Using `encapsulation dot1Q`, the router can identify VLAN tags and route traffic between the networks.

This was my first experience using router-on-a-stick and it helped me understand how a single physical router interface can service multiple VLANs.

---

## Verification

After completing the configuration, I verified the setup using the following commands.

### Verify VLANs

```cisco
show vlan brief
```

Expected result:

* VLAN 10 appears with the correct access ports
* VLAN 20 appears with the correct access ports

### Verify Trunk Links

```cisco
show interfaces trunk
```

Expected result:

* Trunk ports appear as active
* VLANs 10 and 20 are allowed across the trunk

### Verify Router Interfaces

```cisco
show ip interface brief
```

Expected result:

* G0/0.10 shows `up/up`
* G0/0.20 shows `up/up`

---

## Connectivity Test

From PC3, I tested connectivity to PC0:

```cmd
ping 192.168.10.1
```

Successful replies confirmed that:

* VLANs were configured correctly
* Trunk links were working
* Router-on-a-stick routing was functioning
* Devices could communicate across VLAN boundaries

Expected output:

```text
Reply from 192.168.10.1: bytes=32 time<1ms TTL=127
```

---

## Key Takeaways

This lab helped me understand why VLANs are used to separate networks and how routers enable communication between them.

Before completing the lab, I understood the basic idea of VLANs, but configuring them across multiple switches and seeing traffic successfully routed between VLANs made the concept much easier to understand.

It also gave me practical experience with trunking, 802.1Q tagging and router-on-a-stick routing, which are common concepts in enterprise networking environments.

---

## Files Included

| File            | Description                                      |
| --------------- | ------------------------------------------------ |
| `lab1-vlan.pkt` | Cisco Packet Tracer lab file                     |
| `screenshots/`  | Topology, verification commands and ping results |
