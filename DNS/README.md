# DNS Deep Dive with dig & nslookup

## Key Findings

| | |
|---|---|
| **Tools Used** | `dig`, `nslookup` |
| **Environment** | Kali Linux on QEMU/KVM |
| **Target Domain** | google.com (and www.bbc.co.uk for CNAME) |
| **Date Completed** | 3 June 2026 |

### What we discovered

DNS (Domain Name System) is how the internet translates human-readable addresses like `google.com` into IP addresses that machines can actually connect to. In this lab I queried DNS directly — instead of letting it happen silently in the background — and got to see exactly what's going on under the hood.

Here's a quick summary of what came back from the queries:

- **Google's IPv4 addresses** — six different IPs (e.g. 142.251.30.101), showing Google load-balances traffic across many servers
- **Google's IPv6 addresses** — four addresses in the `2a00:1450:4009:c13::` range, confirming Google fully supports the newer IPv6 standard
- **Mail server** — all of Google's email routes through `smtp.google.com` with a priority of 10
- **Name servers** — Google runs four of its own DNS servers: ns1 through ns4.google.com
- **BBC CNAME alias** — `www.bbc.co.uk` is actually an alias pointing to `www.bbc.co.uk.pri.bbc.co.uk`, a common pattern for large organisations managing web traffic
- **Reverse lookup** — the IP `8.8.8.8` (Google's public DNS) correctly resolves back to `dns.google`
- **Full DNS trace** — watched the resolution process start from the root servers at the very top of the internet (a.root-servers.net through m.root-servers.net), pass through the `.com` TLD servers, and land on Google's own name servers for the final answer

### Why this matters

This is directly relevant to both **Network+** and **security work**. DNS is one of the most abused protocols in real-world attacks — attackers use it for data exfiltration and command-and-control (C2) communication. Being able to manually query DNS records means you can spot anomalies, map out an organisation's infrastructure during recon, and investigate suspicious traffic in logs. The `+trace` command in particular shows you how DNS delegation actually works, which is the kind of thing that separates someone who *uses* DNS from someone who *understands* it.

---

## What I Did

### Step 1 — A Record (IPv4 lookup)

```bash
dig google.com A
```

The most basic DNS query — asking "what's the IP address for google.com?" The answer came back with **six different IPs**, all in the 142.251.30.x range. Google returns multiple addresses and rotates between them to spread traffic across their servers (this is called DNS-based load balancing). Query time was 20ms.

---

### Step 2 — AAAA Record (IPv6 lookup)

```bash
dig google.com AAAA
```

Same idea, but asking for IPv6 addresses instead. Google returned **four IPv6 addresses** in the `2a00:1450:4009:c13::` range. Not every domain supports IPv6, but Google does. Query time here was 0ms — blazing fast, likely already cached from the previous query.

---

### Step 3 — MX Record (mail servers)

```bash
dig google.com MX
```

MX records tell you where emails for a domain get delivered. Google's mail all goes through **smtp.google.com** with a priority value of 10. Lower numbers = higher priority, so if there were multiple mail servers, the one with the lowest number gets tried first. From a security perspective, this is useful recon — it reveals the mail infrastructure of a target organisation.

---

### Step 4 — NS Record (name servers)

```bash
dig google.com NS
```

NS records tell you which servers are *authoritative* for a domain — i.e. the ones that hold the official, definitive DNS records for it. Google runs its own: **ns1.google.com, ns2.google.com, ns3.google.com, and ns4.google.com**. Four of them for redundancy. Query time was 24ms.

---

### Step 5 — CNAME Record (alias lookup)

```bash
dig www.bbc.co.uk CNAME
```

A CNAME is basically an alias — one domain name pointing to another. `www.bbc.co.uk` doesn't point directly to an IP; it's an alias for `www.bbc.co.uk.pri.bbc.co.uk`. This is a common pattern for large organisations that route their web traffic through internal infrastructure layers. Query time was 0ms.

---

### Step 6 — PTR Record (reverse lookup)

```bash
dig -x 8.8.8.8
```

This is DNS in reverse — instead of "what's the IP for this name?", you're asking "what's the name for this IP?". Querying `8.8.8.8` (Google's famous public DNS resolver) came back with **dns.google** — which is exactly what you'd expect. Reverse lookups are heavily used in log analysis and incident response to identify what a particular IP actually belongs to.

---

### Step 7 — dig +trace (full recursive resolution)

```bash
dig google.com +trace
```

This is the most important command in the lab. Instead of just giving you the final answer, `+trace` shows you every single step DNS takes to resolve a name from scratch. Here's what happened:

1. Started at the **root servers** — the very top of the DNS hierarchy (a.root-servers.net through m.root-servers.net). These are the 13 clusters of servers that know where everything on the internet begins.
2. The root servers pointed down to the **.com TLD servers** (h.gtld-servers.net, e.gtld-servers.net, etc.)
3. The TLD servers handed off to **Google's own name servers** (ns1–ns4.google.com)
4. Google's name servers gave the final answer — six A records with the actual IP addresses

You'll notice some "UDP setup failed: network unreachable" messages in the output — that's just the VM trying to reach IPv6 addresses for the root/TLD servers, which aren't available on the network setup here. The IPv4 path worked fine.

---

### Step 8 — nslookup

```bash
nslookup google.com
nslookup -type=MX google.com
nslookup -type=NS google.com
```

`nslookup` is the older, simpler DNS query tool that still shows up everywhere — on Windows machines, in interviews, in legacy documentation. It does the same job as `dig` but with less detail. Running all three commands confirmed the same results we got with dig:

- **Basic lookup** — returned Google's IPs (both IPv4 and IPv6 this time, since nslookup returns both by default)
- **MX lookup** — confirmed smtp.google.com as the mail exchanger with priority 10
- **NS lookup** — confirmed all four of Google's name servers (ns1–ns4.google.com)

---

## Tools Reference

| Tool | What it does |
|---|---|
| `dig` | Detailed DNS query tool — preferred for security work |
| `nslookup` | Simpler DNS tool — older but still widely used |

| Record Type | What it means |
|---|---|
| A | Maps a domain to an IPv4 address |
| AAAA | Maps a domain to an IPv6 address |
| MX | Points to the mail server(s) for a domain |
| NS | Lists the authoritative name servers for a domain |
| CNAME | An alias — one domain pointing to another |
| PTR | Reverse lookup — maps an IP back to a domain name |

---

## What's in this folder

```
Lab5-DNS/
├── README.md               — this file, explaining the lab and findings
└── screenshots/
    ├── A-IPv4.png              — dig A record query for google.com
    ├── AAAA-IPv6.png           — dig AAAA (IPv6) record query for google.com
    ├── MailServer.png          — dig MX record showing smtp.google.com
    ├── NameServer.png          — dig NS record showing ns1–ns4.google.com
    ├── CNAME.png               — dig CNAME showing BBC alias chain
    ├── PTR.png                 — reverse lookup of 8.8.8.8 → dns.google
    ├── _trace_prompt_start.png — dig +trace, top of output (root servers)
    ├── _trace_prompt_end.png   — dig +trace, bottom of output (final resolution)
    └── nslookup.png            — all three nslookup queries side by side
```
