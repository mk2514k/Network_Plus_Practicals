# Site-to-Site IPSec VPN — Cisco Packet Tracer Lab

## What is a VPN and why does it matter?

When two offices need to communicate over the internet, there's a problem — the internet is public. Anyone sitting between your two locations could theoretically intercept your traffic. A VPN (Virtual Private Network) solves this by creating an encrypted tunnel between the two endpoints, so even if someone captures the packets in transit, they can't read them.

A site-to-site VPN connects two entire networks together, rather than a single device dialling in. Think of it like a private road between two buildings — traffic still travels through public space, but it's enclosed and protected the whole way.

IPSec is the protocol suite that makes this happen. It handles authentication (proving both ends are who they say they are) and encryption (scrambling the data). It operates in two phases:

- **Phase 1 (ISAKMP)** — The two routers negotiate a secure management channel. They agree on encryption algorithms, hashing, authentication method, and a Diffie-Hellman group to safely exchange keys. This is the handshake.
- **Phase 2 (IPSec/ESP)** — Using the secure channel from Phase 1, they negotiate how the actual data traffic will be encrypted. This is the tunnel you send your real data through.

---

## Topology

```
PC1 — Router1 (10.0.0.1) -----[internet]----- Router2 (10.0.0.2) — PC2
       192.168.1.0/24                               192.168.2.0/24
```

Two Cisco 2911 routers act as VPN gateways. PC1 sits in the 192.168.1.0/24 network, PC2 in the 192.168.2.0/24 network. The routers are connected to each other over a simulated WAN link (the 10.0.0.0/30 segment). The goal is for PC1 to reach PC2 securely through an encrypted tunnel.

---

## Prerequisites — Enabling the Security License

Before any crypto commands will work on a Cisco 2911 in Packet Tracer, you need to activate the security technology package. Without this, commands like `crypto isakmp policy` simply won't be recognised — the router will just return `% Unrecognized command`.

Run this on **both routers** and reload:

```
Router(config)# license boot module c2900 technology-package securityK9
```

Accept the EULA when prompted, then:

```
Router# write
Router# reload
```

> **Screenshot:** Router1 CLI showing the license activation, EULA acceptance, and confirmation that the securityk9 license has been accepted.

This was a key stumbling block — the lab won't progress without it, and the error message gives you no hint as to why the crypto commands aren't working.

---

## Step 1 — Basic Interface Configuration

Before the VPN can even be attempted, both routers need their interfaces configured and their routing sorted so they can reach each other across the WAN.

**Router1:**
```
Router1(config)# interface GigabitEthernet0/0
Router1(config-if)# ip address 192.168.1.1 255.255.255.0
Router1(config-if)# no shutdown

Router1(config)# interface GigabitEthernet0/1
Router1(config-if)# ip address 10.0.0.1 255.255.255.252
Router1(config-if)# no shutdown

Router1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

**Router2:**
```
Router2(config)# interface GigabitEthernet0/0
Router2(config-if)# ip address 192.168.2.1 255.255.255.0
Router2(config-if)# no shutdown

Router2(config)# interface GigabitEthernet0/1
Router2(config-if)# ip address 10.0.0.2 255.255.255.252
Router2(config-if)# no shutdown

Router2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

The static routes tell each router where to send traffic destined for the remote LAN. Without these, packets wouldn't even know where to go before IPSec gets involved.

> **Screenshot:** Both router CLIs side by side showing interface configuration and confirmation that interfaces came up.

---

## Step 2 — Configure Phase 1 (ISAKMP Policy)

Phase 1 is about the routers authenticating each other and agreeing on how to protect the control channel. You define a policy with a priority number (lower = higher priority) and set the encryption, hash, authentication method, DH group, and lifetime.

Run this on **both routers**:

```
crypto isakmp policy 10
 encryption aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
```

- `aes` — AES encryption for the control channel
- `sha` — SHA hashing for integrity
- `pre-share` — we're using a pre-shared key (a password both sides know) rather than certificates
- `group 2` — Diffie-Hellman group 2 for key exchange
- `lifetime 86400` — how long before the Phase 1 SA expires (24 hours)

Then configure the pre-shared key. Both routers need to know the same key, and each one points it at the other's WAN IP:

**Router1:**
```
crypto isakmp key cisco123 address 10.0.0.2
```

**Router2:**
```
crypto isakmp key cisco123 address 10.0.0.1
```

This key is what proves to each router that the other end is legitimate. In production you'd use something far stronger than `cisco123`.

---

## Step 3 — Configure Phase 2 (IPSec Transform Set and Crypto Map)

Phase 2 defines how the actual data traffic is encrypted. You create a transform set (the encryption/integrity algorithms for the data plane), then a crypto map that ties everything together — who to connect to, what traffic to encrypt, and which transform set to use.

**Both routers:**
```
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
```

This says: encrypt the payload with AES (`esp-aes`) and authenticate it with SHA-HMAC (`esp-sha-hmac`). ESP (Encapsulating Security Payload) is the IPSec protocol that does the actual encryption.

Now the crypto map. This is the policy that gets applied to an interface:

**Router1:**
```
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set VPN-SET
 match address VPN-ACL
```

**Router2:**
```
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.1
 set transform-set VPN-SET
 match address VPN-ACL
```

The `match address VPN-ACL` line tells the router which traffic should be encrypted. We define that next.

---

## Step 4 — Define Interesting Traffic (ACL)

IPSec doesn't encrypt everything — only "interesting" traffic you define. This is done with an extended ACL.

**Router1:**
```
ip access-list extended VPN-ACL
 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
```

**Router2:**
```
ip access-list extended VPN-ACL
 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
```

This says: any traffic from the local LAN going to the remote LAN should be caught by IPSec and encrypted. The ACLs are mirrors of each other — Router1 catches traffic going from LAN1 to LAN2, Router2 catches the return traffic.

---

## Step 5 — Apply the Crypto Map to the WAN Interface

The crypto map doesn't do anything until it's applied to the outbound interface — the one facing the other router:

**Both routers:**
```
interface GigabitEthernet0/1
 crypto map VPN-MAP
```

This is what activates the VPN. Any packet leaving GigabitEthernet0/1 that matches the VPN-ACL will now be encrypted before it goes out.

---

## Step 6 — Verify the Tunnel

First, trigger the tunnel by pinging from PC1 to PC2:

```
C:\> ping 192.168.2.10
```

The first couple of pings may time out — this is normal. The tunnel needs to be negotiated on the first packet, which takes a moment. Subsequent pings should succeed.

> **Screenshot:** PC1 command prompt showing the ping results — first attempt with some timeouts, second attempt with 100% success as the tunnel is now up.

Then verify on the routers:

```
Router1# show crypto isakmp sa
```

You're looking for a state of `QM_IDLE`, which means Phase 1 is established and the routers have authenticated each other.

```
Router1# show crypto ipsec sa
```

Look at the packet counters — `#pkts encrypt` and `#pkts decrypt` should be incrementing, confirming actual data is flowing through the tunnel. The SAs should show `Status: ACTIVE`.

> **Screenshot:** Router1 CLI showing `show crypto isakmp sa` with QM_IDLE state and `show crypto ipsec sa` with ACTIVE inbound and outbound SAs and incrementing packet counters.

---

## What the Output Tells Us

Looking at the verified output from this lab:

- **ISAKMP SA:** `dst 10.0.0.2, src 10.0.0.1, state QM_IDLE` — Phase 1 is up. QM_IDLE means Quick Mode (Phase 2) has completed and the tunnel is idle but ready.
- **IPSec SA:** Inbound SPI `0x8C655086`, outbound SPI `0x274F8169` — these are the Security Parameter Indexes, unique identifiers for each SA. Both show `Status: ACTIVE`.
- **Packet counters:** `#pkts encaps: 6, #pkts encrypt: 6` on the outbound side confirms real traffic was encrypted and sent through the tunnel.
- **Transform:** `esp-aes esp-sha-hmac` — confirming AES encryption and SHA integrity are in use.
- **Crypto endpoints:** `local crypto endpt.: 10.0.0.1, remote crypto endpt.: 10.0.0.2` — the tunnel endpoints are exactly as configured.

---

## Key Takeaways

- IPSec operates in two phases — Phase 1 builds a secure management channel, Phase 2 builds the actual data tunnel
- The security technology license must be enabled on Cisco 2911 routers before any crypto commands will be recognised
- Interesting traffic is defined by an ACL — only matching traffic gets encrypted
- The crypto map ties the peer address, transform set, and ACL together and must be applied to the WAN interface
- First pings may time out as the tunnel negotiates — this is expected behaviour, not a failure

---

## Files

| File      | Description      |
|-----------|------------------|
| `site-to-site IPsec VPN lab` | Packet Tracer save file |
| `screenshots/`               | Topology, interface brief, ping, man config sec feature, tunnel status & packet count, ping via tunnel |
