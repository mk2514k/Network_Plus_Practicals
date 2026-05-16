## Objective
Complete 10 subnetting problems without a calculator. For each address, find the network address, broadcast address, first and last usable host, subnet mask, and total number of hosts. Verify answers using ipcalc. Repeat any incorrect ones.

## Why This Matters
Subnetting is one of the most tested skills in networking. In the real world, network engineers subnet constantly — whether it's carving up office networks, setting up VLANs, or designing cloud infrastructure. Being able to do this quickly in your head is what separates someone who understands networking from someone who just read about it.

---

## Method
For any address with a prefix (e.g. /26), the approach is:

1. Find the block size — subtract the last octet of the mask from 256. That's how many addresses are in each subnet.
2. Find the network address — round the host octet down to the nearest multiple of the block size.
3. Broadcast — network address + block size - 1.
4. First host — network address + 1.
5. Last host — broadcast - 1.
6. Total hosts — 2^(32 - prefix) - 2.

---

## Drill Problems and Solutions

### 1. 192.168.10.55 /26
- Block size: 64 — subnet boundaries at 0, 64, 128...
- Network: 192.168.10.0
- Broadcast: 192.168.10.63
- First host: 192.168.10.1
- Last host: 192.168.10.62
- Mask: 255.255.255.192
- Usable hosts: 62

### 2. 10.0.0.130 /25
- Block size: 128 — boundaries at 0, 128...
- Network: 10.0.0.128
- Broadcast: 10.0.0.255
- First host: 10.0.0.129
- Last host: 10.0.0.254
- Mask: 255.255.255.128
- Usable hosts: 126

### 3. 172.16.45.200 /28
- Block size: 16 — boundaries at 192, 208...
- Network: 172.16.45.192
- Broadcast: 172.16.45.207
- First host: 172.16.45.193
- Last host: 172.16.45.206
- Mask: 255.255.255.240
- Usable hosts: 14

### 4. 192.168.1.77 /27
- Block size: 32 — boundaries at 64, 96...
- Network: 192.168.1.64
- Broadcast: 192.168.1.95
- First host: 192.168.1.65
- Last host: 192.168.1.94
- Mask: 255.255.255.224
- Usable hosts: 30

### 5. 10.10.10.10 /30
- Block size: 4 — boundaries at 8, 12...
- Network: 10.10.10.8
- Broadcast: 10.10.10.11
- First host: 10.10.10.9
- Last host: 10.10.10.10
- Mask: 255.255.255.252
- Usable hosts: 2
- Note: /30 is the standard for point-to-point links between routers

### 6. 172.31.255.250 /24
- Block size: 256 — clean boundary at .0
- Network: 172.31.255.0
- Broadcast: 172.31.255.255
- First host: 172.31.255.1
- Last host: 172.31.255.254
- Mask: 255.255.255.0
- Usable hosts: 254

### 7. 192.168.100.199 /29
- Block size: 8 — boundaries at 192, 200...
- Network: 192.168.100.192
- Broadcast: 192.168.100.199
- First host: 192.168.100.193
- Last host: 192.168.100.198
- Mask: 255.255.255.248
- Usable hosts: 6

### 8. 10.0.5.67 /22
- Prefix crosses into third octet — block size 4 in third octet
- Boundaries at 10.0.4.0, 10.0.8.0...
- Network: 10.0.4.0
- Broadcast: 10.0.7.255
- First host: 10.0.4.1
- Last host: 10.0.7.254
- Mask: 255.255.252.0
- Usable hosts: 1022

### 9. 192.168.0.201 /23
- Block size 2 in third octet — boundaries at 192.168.0.0, 192.168.2.0...
- Network: 192.168.0.0
- Broadcast: 192.168.1.255
- First host: 192.168.0.1
- Last host: 192.168.1.254
- Mask: 255.255.254.0
- Usable hosts: 510

### 10. 172.16.8.15 /20
- Block size 16 in third octet — boundaries at 172.16.0.0, 172.16.16.0...
- Network: 172.16.0.0
- Broadcast: 172.16.15.255
- First host: 172.16.0.1
- Last host: 172.16.15.254
- Mask: 255.255.240.0
- Usable hosts: 4094

---

## Verification
All answers verified using ipcalc:
ipcalc 192.168.10.55/26
ipcalc 10.0.0.130/25
ipcalc 172.16.45.200/28
ipcalc 192.168.1.77/27
ipcalc 10.10.10.10/30
ipcalc 172.31.255.250/24
ipcalc 192.168.100.199/29
ipcalc 10.0.5.67/22
ipcalc 192.168.0.201/23
ipcalc 172.16.8.15/20

---

## Supernetting Practice
Supernetting is the opposite of subnetting — combining smaller networks into one larger one.

Example: Combine 192.168.0.0/24 and 192.168.1.0/24
- Both share the first 23 bits
- Supernet: 192.168.0.0/23
- This covers 192.168.0.0 through 192.168.1.255 (510 usable hosts)

Rule: To supernet, the networks must be contiguous and the combined block must land on a natural boundary for the new prefix.

---

## Key Takeaways
- Subnetting is pattern recognition — once you know the block sizes for each prefix, it becomes mechanical
- /30 = point-to-point links, /29 = small segments, /24 = standard LAN, /22 and above = large or aggregated networks
- Supernetting is used in routing to summarise multiple routes into one, reducing routing table size
