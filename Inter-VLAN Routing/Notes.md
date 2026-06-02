# VLAN Segmentation + Inter-VLAN Routing

**Tools:** Cisco Packet Tracer  
**Topics:** VLANs, trunking, router-on-a-stick, 802.1Q  
**Related cert:** CompTIA Network+ (N10-009)

---

## What this lab covers

Two VLANs are created across two switches. A single router handles traffic between them using a technique called **router-on-a-stick** — one physical cable, two logical networks.

The goal: a PC on VLAN 10 can ping a PC on VLAN 20, even though they're on completely separate networks.

---

## Network topology

```
        [  R1 ]
            |
            |  (trunk)
            |
        [  SW1 ]
        /        \
   (access)        \ (trunk)
                     \
   PC1    PC2      [  SW2  ]
  VLAN10  VLAN20   /        \
                 PC3         PC4
              VLAN10         VLAN20
```

---

## IP address plan

| Device | VLAN | IP Address       | Default Gateway |
|--------|------|------------------|-----------------|
| PC1    | 10   | 192.168.10.10/24 | 192.168.10.1    |
| PC2    | 20   | 192.168.20.10/24 | 192.168.20.1    |
| PC3    | 10   | 192.168.10.20/24 | 192.168.10.1    |
| PC4    | 20   | 192.168.20.20/24 | 192.168.20.1    |
| R1 g0/0.10 | 10 | 192.168.10.1/24 | -              |
| R1 g0/0.20 | 20 | 192.168.20.1/24 | -              |

---

## Step-by-step configuration

### 1. Create VLANs on SW1

VLANs are created first, then ports are assigned to them.

```
enable
configure terminal

vlan 10
 name VLAN10_Sales
exit

vlan 20
 name VLAN20_IT
exit
```

### 2. Assign access ports on SW1

Access ports connect to end devices (PCs). Each port belongs to one VLAN only.

```
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
exit

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
exit
```

### 3. Set trunk ports on SW1

Trunk ports carry traffic from all VLANs simultaneously using 802.1Q tagging. One trunk goes up to R1, one goes across to SW2.

```
interface fastEthernet 0/3
 switchport mode trunk
exit

interface fastEthernet 0/4
 switchport mode trunk
exit
```

### 4. Configure SW2

Same VLAN setup. Access ports for PCs, one trunk back to SW1.

```
enable
configure terminal

vlan 10
 name VLAN10_Sales
exit

vlan 20
 name VLAN20_IT
exit

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
exit

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
exit

interface fastEthernet 0/3
 switchport mode trunk
exit
```

### 5. Configure R1 (router-on-a-stick)

One physical interface (`g0/0`) is split into two logical sub-interfaces — one per VLAN. The `encapsulation dot1Q` line tells the router which VLAN tag to look for on incoming frames.

```
enable
configure terminal

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
```

### 6. Set PC IP addresses

On each PC: Desktop → IP Configuration.  
Set the IP, subnet mask (`255.255.255.0`), and default gateway as per the IP plan above.

---

## Verification

Run these on SW1 and confirm the output:

```
show vlan brief
```
Both VLANs (10 and 20) should appear with their assigned ports listed.

```
show interfaces trunk
```
Your trunk ports should appear here showing VLANs allowed and active.

Run this on R1:

```
show ip interface brief
```
Sub-interfaces `g0/0.10` and `g0/0.20` should both show `up/up` with their IPs assigned.

---

## Test — cross-VLAN ping

From PC1 (VLAN 10) open Desktop → Command Prompt:

```
ping 192.168.20.20
```

This pings PC4 on VLAN 20 across both switches. A successful reply proves the full chain is working — access ports, trunk links, and router-on-a-stick routing.

Expected output:
```
Reply from 192.168.20.20: bytes=32 time<1ms TTL=127
```

---

## Key concepts

**VLAN** — logically separates devices on the same physical switch into different networks. Devices in different VLANs cannot communicate without a router.

**Access port** — connects to an end device, belongs to one VLAN, traffic is untagged.

**Trunk port** — connects switches or routers, carries multiple VLANs simultaneously using 802.1Q tags.

**Router-on-a-stick** — a single router interface handles routing between VLANs using sub-interfaces. One cable, multiple logical connections.

**802.1Q** — the industry standard for VLAN tagging on trunk links.

---

## Files in this folder

| File | Description |
|------|-------------|
| `lab1-vlan.pkt` | Packet Tracer save file |
| `screenshots/` | Topology, show commands, ping output |
