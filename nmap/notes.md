# Nmap Reconnaissance on Metasploitable 2

**Tools:** Nmap 7.98 · Kali Linux (QEMU/KVM)  
**Target:** Metasploitable 2 · `192.168.122.60`  
**Part of:** Network+ / Early Security Portfolio

---

## What I found

This machine is a goldmine for an attacker. Running two Nmap scans against it revealed **30 open ports**, several of which have known backdoors baked straight into the software — not misconfigurations, actual backdoors. In a real environment, this machine would be compromised within minutes of being exposed to the internet.

**The standout discoveries:**

- **Port 1524 — a root shell sitting open.** No username, no password. Anyone who connects gets full control of the machine instantly. This is Metasploitable's most obvious intentional vulnerability.
- **Port 21 — vsftpd 2.3.4.** This specific version of the FTP server has a backdoor built into it (CVE-2011-2523). It was snuck into the source code in 2011 and triggers a root shell when you send a specific character in the username field.
- **Port 445 — Samba 3.0.20.** The version of Windows file sharing running here has a remote code execution vulnerability (CVE-2007-2447) that Metasploit can exploit in seconds.
- **Port 6667 — UnrealIRCd.** Another backdoored piece of software (CVE-2010-2075). The IRC server has a hidden command that executes anything you send it as root.
- **Plaintext protocols everywhere** — Telnet (23), rlogin (513), rsh (514). These send usernames and passwords across the network in plain text. Anyone sniffing the network sees everything.

The bigger picture: this exercise shows exactly why version numbers matter in security. Nmap didn't just tell us "FTP is running" — it told us *which version*, which is what lets you look up whether that version has known vulnerabilities.

---

## What I did and why

### Scan 1 — Full service and OS detection

```bash
sudo nmap -A -sV -O 192.168.122.60 -oN nmap-full.txt
```

The `-A` flag is the "tell me everything" flag — it runs service detection, OS fingerprinting, default scripts, and a traceroute all in one go. `-sV` probes each open port to figure out exactly what software and version is running. `-O` tries to guess the operating system by analysing how the machine responds to certain packets. Everything gets saved to `nmap-full.txt` so I have a record.

This took about 75 seconds and came back with 23 open ports with full version info.

### Scan 2 — All 65,535 ports

```bash
sudo nmap -p- 192.168.122.60 -oN nmap-allports.txt
```

By default Nmap only checks the 1,000 most common ports. The `-p-` flag tells it to check every single port. This caught an extra 7 ports running on high, unusual port numbers that the first scan missed. In a real assessment you always run both — services hiding on non-standard ports are often the most interesting ones.

This scan took 4 seconds because it's just checking if ports are open, not doing any version detection.

---

## Full port table

| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21 | FTP | vsftpd 2.3.4 | **Critical** — backdoor CVE-2011-2523, anonymous login allowed |
| 22 | SSH | OpenSSH 4.7p1 | Old version, weak 1024-bit DSA key |
| 23 | Telnet | Linux telnetd | Plaintext — credentials visible on the wire |
| 25 | SMTP | Postfix smtpd | SSLv2 supported — vulnerable to DROWN attack |
| 53 | DNS | ISC BIND 9.4.2 | Old version, potential zone transfer |
| 80 | HTTP | Apache 2.2.8 | Outdated — multiple known CVEs |
| 111 | RPCbind | RPC #100000 | Exposes NFS mount list |
| 139 | NetBIOS | Samba 3.X–4.X | SMB signing disabled — relay attacks possible |
| 445 | SMB | Samba 3.0.20-Debian | **Critical** — RCE via CVE-2007-2447 |
| 512 | Exec | rexec | Remote execution, no encryption |
| 513 | Login | rlogind | Plaintext remote login |
| 514 | Shell | rsh | Remote shell, no authentication |
| 1099 | Java-RMI | GNU Classpath grmiregistry | RCE potential via Java RMI |
| 1524 | Bindshell | Metasploitable root shell | **Critical** — unauthenticated root shell |
| 2049 | NFS | NFS 2–4 | Shares may be world-readable |
| 2121 | FTP | ProFTPD 1.3.1 | Known vulnerabilities in this version |
| 3306 | MySQL | 5.0.51a-3ubuntu5 | Exposed with no apparent auth |
| 3632 | Distccd | — | Compiler daemon — RCE abuse potential |
| 5432 | PostgreSQL | 8.3.0–8.3.7 | Expired self-signed cert from 2010 |
| 5900 | VNC | Protocol 3.3 | Weak authentication, no NLA |
| 6000 | X11 | — | Display server exposed to network |
| 6667 | IRC | UnrealIRCd 3.2.8.1 | **Critical** — backdoor CVE-2010-2075 |
| 6697 | IRC-U | — | Secondary IRC port |
| 8009 | AJP13 | Apache JServ v1.3 | Ghostcat vulnerability CVE-2020-1938 |
| 8180 | HTTP | Apache Tomcat 5.5 | Extremely outdated, default creds likely |
| 8787 | Msgsrvr | — | Unidentified service |
| 41493 | Unknown | — | High port, unidentified |
| 41754 | Unknown | — | High port, unidentified |
| 47208 | Unknown | — | High port, unidentified |
| 49881 | Unknown | — | High port, unidentified |

**OS detected:** Linux 2.6.9–2.6.33 · FQDN: `metasploitable.localdomain`

---

## Why this matters

In a SOC role, understanding what Nmap output means is day one stuff. Alerts often reference ports and service versions — knowing that port 1524 open on an internal machine means "someone has a root backdoor listening" versus "that's probably fine" is the difference between catching an incident and missing it.

For pentesting, this scan output is the foundation everything else builds on. Every lab that follows in this repo — Nikto, Gobuster, exploitation — starts from what Nmap found here.

---

## Files

| File | Description |
|------|-------------|
| `nmap-full.txt` | Raw output — full service and OS scan |
| `nmap-allports.txt` | Raw output — all 65,535 ports scan |
| `screenshots/` | Terminal screenshots of both scans | -sV -O` scan |
| `nmap-allports.txt` | Raw output from `-p-` all ports scan |
| `screenshots/` | Terminal screenshots of scans running and completing |
