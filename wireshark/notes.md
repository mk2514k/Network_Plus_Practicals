# Wireshark: Live Packet Capture

## Objective
Capture a live DNS query/response and HTTP request/response in Wireshark.
Open the packet tree and identify headers at each OSI layer for both.

## Setup
- Interface used: `wlp0s20f3` (Wi-Fi)
- Wireshark installed via: `sudo dnf install wireshark wireshark-qt`
- Added user to wireshark group to capture without root:
  `sudo usermod -aG wireshark $USER`
- Used `newgrp wireshark` to apply group change without full logout

---

## Capture 1 — DNS

**Filter applied:** `dns`

**Traffic generated:**
```bash
nslookup google.com
```

**Packets captured:**
- Query: `Standard query A google.com`
- Response: `Standard query response A google.com A <IP>`

### OSI Layer Breakdown (DNS)

| OSI Layer | Header/Protocol | Key Fields |
|-----------|----------------|------------|
| L1 — Physical | Frame | Arrival time, frame length, interface |
| L2 — Data Link | Ethernet II | Source MAC, destination MAC |
| L3 — Network | Internet Protocol (IP) | Source IP, destination IP |
| L4 — Transport | UDP | Source port (ephemeral), destination port 53 |
| L7 — Application | Domain Name System | Query name (google.com), record type (A) |

> DNS uses **UDP port 53** for standard queries.

---

## Capture 2 — HTTP

**Filter applied:** `http`

**Traffic generated:**
```bash
curl http://example.com
```

**Packets captured:**
- Request: `GET / HTTP/1.1`
- Response: `HTTP/1.1 200 OK`

### OSI Layer Breakdown (HTTP)

| OSI Layer | Header/Protocol | Key Fields |
|-----------|----------------|------------|
| L1 — Physical | Frame | Arrival time, frame length, interface |
| L2 — Data Link | Ethernet II | Source MAC, destination MAC |
| L3 — Network | Internet Protocol (IP) | Source IP, destination IP |
| L4 — Transport | TCP | Source port (ephemeral), destination port 80 |
| L7 — Application | Hypertext Transfer Protocol | Method (GET), Host header, status code (200) |

> HTTP uses **TCP port 80**. Unlike DNS/UDP, TCP requires a 3-way handshake — you may have seen SYN/SYN-ACK/ACK packets just before the GET request.

---

## Key Takeaways
- Every packet contains headers from multiple OSI layers stacked on top of each other
- DNS = UDP, HTTP = TCP — protocol choice at L4 matters for reliability vs speed
- Wireshark's packet tree maps directly to the OSI model — great for visualising encapsulation
- Screenshots saved in `/screenshots/`
