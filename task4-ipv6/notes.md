## Objective
Understand IPv6 address types and notation rules. Cover EUI-64 interface ID generation, link-local addresses, the loopback address, and compression rules. Complete 3 full-to-compressed address conversions.

## Why This Matters
IPv4 addresses ran out. IPv6 is how the internet scales going forward — it's already widely deployed in ISPs, mobile networks, and cloud infrastructure. Understanding IPv6 addressing shows awareness of where the industry is heading, not just where it's been.

---

## Address Types

### Loopback — ::1
The equivalent of IPv4's 127.0.0.1. Used by a device to refer to itself.
Full form: 0000:0000:0000:0000:0000:0000:0000:0001

### Link-Local — fe80::/10
Automatically assigned to every IPv6-enabled interface, no configuration needed. Only valid on the local network segment — not routable. Used for neighbour discovery and router communication on the same link. Always starts with fe80.

### EUI-64 (Interface ID Generation)
EUI-64 is how a device automatically generates the last 64 bits of its IPv6 address from its MAC address.

Steps:
1. Take the MAC address — e.g. 00:1A:2B:3C:4D:5E
2. Split it in half: 00:1A:2B | 3C:4D:5E
3. Insert FF:FE in the middle: 00:1A:2B:FF:FE:3C:4D:5E
4. Flip the 7th bit of the first byte — 00 in binary is 00000000, flip bit 7 to get 00000010 = 02
5. Result: 02:1A:2B:FF:FE:3C:4D:5E
6. Written as IPv6 groups: 021A:2BFF:FE3C:4D5E

Combined with a fe80::/64 prefix, the full link-local address becomes:
fe80::021A:2BFF:FE3C:4D5E

---

## Compression Rules

IPv6 addresses are 128 bits written as 8 groups of 4 hex digits. Two rules let you shorten them:

Rule 1 — Drop leading zeros within each group
- 0042 becomes 42
- 0000 becomes 0

Rule 2 — Replace the longest consecutive run of all-zero groups with ::
- Can only be used once per address
- 0000:0000:0000 becomes ::

---

## Compression Exercises

### 1. 2001:0DB8:0000:0000:0000:0000:0000:0001

Step by step:
- Drop leading zeros: 2001:DB8:0:0:0:0:0:1
- Replace longest zero run with :: — result: 2001:DB8::1

Compressed: 2001:db8::1

---

### 2. FE80:0000:0000:0000:021A:2BFF:FE3C:4D5E

Step by step:
- Drop leading zeros: FE80:0:0:0:21A:2BFF:FE3C:4D5E
- Replace zero run with :: — result: FE80::21A:2BFF:FE3C:4D5E

Compressed: fe80::21a:2bff:fe3c:4d5e

Note: This is what a real EUI-64 generated link-local address looks like.

---

### 3. 2001:0DB8:ABCD:0000:0000:1234:0000:5678

Step by step:
- Drop leading zeros: 2001:DB8:ABCD:0:0:1234:0:5678
- Two zero runs present — replace the first and longest: 2001:DB8:ABCD::1234:0:5678

Compressed: 2001:db8:abcd::1234:0:5678

Note: The single 0 at position 7 cannot also become :: — the double colon can only be used once. So it stays as 0.

---

## Key Takeaways
- IPv6 addresses are long but the compression rules make them manageable in practice
- EUI-64 means devices can self-assign addresses from their MAC — useful for stateless autoconfiguration
- Link-local addresses are always there in the background, keeping the local network functioning
- The :: shortcut can only appear once — if unsure which zero run to collapse, pick the longest one
