# HSRP + Port Security Lab (Cisco Packet Tracer)

A Packet Tracer lab combining **Hot Standby Router Protocol (HSRP)** for gateway redundancy with **switchport port security** to lock down end-device access. Built and verified end-to-end.

---

## What This Lab Does

Two routers share a single virtual IP that the PCs use as their default gateway. If the active router goes down, the standby takes over automatically — and the PCs don't notice. Port security on the switches makes sure only the known PC MAC addresses can use those ports.

---

## Topology

```
PC1 — Switch1 — Router1 (Active) — Switch2 — PC2
                     |
               Router2 (Standby)
```

- **Router1** is HSRP Active on both subnets (higher priority: 150 on Gig0/0)
- **Router2** is HSRP Standby, ready to take over
- Both routers share **virtual IPs** — the PCs only ever talk to the VIP, not a real router interface

---

## IP Addressing

| Device  | Interface | IP Address    |
|---------|-----------|---------------|
| PC1     | NIC       | 192.168.1.1   |
| PC2     | NIC       | 192.168.2.1   |
| Router1 | Gig0/0    | 192.168.1.2   |
| Router1 | Gig0/1    | 192.168.2.2   |
| Router2 | Gig0/0    | 192.168.1.3   |
| Router2 | Gig0/1    | 192.168.2.3   |
| VIP (Group 1) | —   | 192.168.1.5   |
| VIP (Group 2) | —   | 192.168.2.5   |

PC1's default gateway → `192.168.1.5` | PC2's default gateway → `192.168.2.5`

---

## HSRP Config

Both routers run HSRP on both interfaces. Priority decides who's active — higher wins. `preempt` means if the higher-priority router comes back online, it reclaims the active role.

**Router1**
```
interface GigabitEthernet0/0
 standby 1 ip 192.168.1.5
 standby 1 priority 150
 standby 1 preempt

interface GigabitEthernet0/1
 standby 2 ip 192.168.2.5
 standby 2 priority 100
 standby 2 preempt
```

**Router2**
```
interface GigabitEthernet0/0
 standby 1 ip 192.168.1.5
 standby 1 priority 100
 standby 1 preempt

interface GigabitEthernet0/1
 standby 2 ip 192.168.2.5
 standby 2 priority 120
 standby 2 preempt
```

> Note: Router2 has higher priority (120) on Gig0/1 / Group 2, making it active for that subnet — load is split across both routers by design.

---

## Port Security Config

Applied to the switch ports facing PC1 and PC2 (`fa0/24` on each switch). Sticky MAC learning means the switch remembers the first MAC it sees and locks the port to it. If anything else plugs in, the port shuts down.

```
interface FastEthernet0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

Verify with:
```
show port-security interface fa0/24
show port-security address
```

---

## Verification Steps

1. **Before HSRP** — ping between PCs fails (no default gateway set yet)
2. **After HSRP + VIP assigned as gateway** — pings succeed
3. **Failover test** — `ping -t 192.168.2.5` from PC1, then physically power off Router1 in Packet Tracer. Pings continue with at most one dropped packet as Router2 transitions from Standby → Active
4. **ARP check** — `arp -a` on PC1 shows the VIP resolving to Router2's MAC after failover
5. **Port security** — ping triggers MAC learning; `show port-security` confirms the sticky entry and zero violations

---

## Key Concepts

- **HSRP** is a Cisco FHRP (First Hop Redundancy Protocol) — it lets two routers share one virtual gateway IP so hosts never need to change their config
- **Preempt** ensures the higher-priority router retakes the active role after recovering from a failure
- **Sticky MAC** port security is a middle ground between fully static (you manually enter MACs) and fully open — it learns automatically but then locks in permanently

---

## Files

| File | Description |
|------|-------------|
| screenshots/ | all screenshots taken to show the lab at different stages |
| HSRP+Port Sec.pk | lab |
