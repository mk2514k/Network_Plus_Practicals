# Access Control Lists — Controlling Traffic on a Cisco Router

**Tools:** Cisco Packet Tracer  
**Devices:** 2911 Router · 2960 Switch · 2x PCs · Server  
**Part of:** Network+ / Early Security Portfolio

---

## What I built and why it matters

An ACL is basically a bouncer for your router. You write a list of rules — allow this, block that — and the router checks every packet against them in order. This lab configured a router to allow web traffic from anyone, restrict SSH to trusted networks only, and silently drop everything else.

This is one of the most common security controls in real networks. SOC analysts read ACL configs constantly. Pentesters look for gaps in them. Getting comfortable with how they work and how to verify them is genuinely useful from day one.

---

## Key findings

- **HTTP worked from anywhere** — User PC hit the web server on port 80 without issue. The permit rule matched and let it through.
- **SSH was blocked** — `telnet 10.0.0.10 22` from User PC timed out with no response. The ACL dropped it silently, which is exactly what a deny rule should do.
- **Hit counters confirmed everything** — after running both tests, `show ip access-lists` showed 10 matches on the HTTP permit rule and 12 matches on the deny-all. Every packet was being evaluated and counted.
- **Ping also got blocked** — once the ACL was applied, ICMP (ping) stopped working too. That's not a bug — ping isn't in the permit rules so the deny-all catches it. This is what "implicit deny" looks like in practice.

---

## The rules

```
ip access-list extended EDGE_POLICY
 permit tcp any any eq 80
 permit tcp any any eq 443
 deny ip any any
```

Applied inbound on `g0/0` — the LAN-facing interface. Traffic gets checked as it arrives from the PCs, before the router does anything else with it.

---

## Verification

| Test | Result |
|------|--------|
| HTTP to server from User PC | ✅ Page loaded |
| Telnet port 22 from User PC | ❌ Connection timed out |
| show ip access-lists hit counters | ✅ Rules actively matching traffic |

---

## Files

| File | Description |
|------|-------------|
| `ACL Lab.pkt` | Packet Tracer save file |
| `screenshots/` | Topology, interface brief, ping, ACL config & application, HTTP test, SSH denial, hit counters |
