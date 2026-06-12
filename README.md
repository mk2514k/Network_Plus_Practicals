## Network Security Labs

Hands-on networking and security labs covering routing protocols, network segmentation, 
access control, VPN configuration, and active reconnaissance against live targets.

---

### Labs

| Lab | What it demonstrates |
|-----|----------------------|
| Wireshark — DNS & HTTP capture | Protocol analysis, identifying cleartext credential traffic |
| Inter-VLAN Routing (802.1Q trunking) | Network segmentation, VLAN isolation, Layer 3 switch routing |
| Site-to-site IPsec VPN | IKE phase 1/2 negotiation, ESP encapsulation, tunnel vs transport mode |
| HSRP + Port Security | Gateway redundancy, MAC flooding mitigation, sticky MAC enforcement |
| ACL configuration | Traffic filtering, least-privilege network access, security zoning |
| OSPF + Static Routing + PAT | Dynamic routing, NAT overload, egress filtering |
| Nmap recon against Metasploitable 2 | Active reconnaissance, service enumeration, OS fingerprinting |
| Web server recon (Nikto, Netcat, Gobuster) | Web enumeration, banner grabbing, directory discovery |
| Subnetting / VLSM | IPv4 allocation, subnet design for segmented network architectures |
| DNS configuration & troubleshooting | Authoritative vs recursive resolution, zone file structure, query analysis |

---

### Key Commands

**Inter-VLAN routing (Router-on-a-Stick):**
```text
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

**IKEv1 Phase 1 — ISAKMP policy:**
```text
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
```

**Port Security — sticky MAC with violation restrict:**
```text
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 2
Switch(config-if)# switchport port-security violation restrict
Switch(config-if)# switchport port-security mac-address sticky
```

**OSPF dynamic routing:**
```text
Router(config)# router ospf 1
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
```

**Nmap — stealth scan with OS detection and version enumeration:**
```bash
nmap -sS -O -sV -R 192.168.1.0/24
```

**TShark — isolate cleartext HTTP GET requests from a host:**
```bash
tshark -r capture.pcap -Y "http.request.method == GET && ip.src == 192.168.10.5"
```

**Nikto — web server assessment against a simulated target:**
```bash
nikto -h http://192.168.1.105 -Tuning 1,2,3,b
```

---

### Environment

- Cisco Packet Tracer (routing/switching labs — `.pkt` files included)
- GNS3 / Kali Linux / Metasploitable 2
- Tools: Wireshark, Nmap, Nikto, Gobuster, Netcat

---

> All scanning and enumeration conducted against isolated lab environments and authorised targets only.
