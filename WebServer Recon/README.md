# Web Server Reconnaissance — Nikto, Gobuster & Netcat

**Tools:** Nikto v2.6.0 · Gobuster v3.8.2 · Netcat · Kali Linux  
**Target:** Metasploitable 2 · `192.168.122.60:80`  
**Part of:** Network+ / Early Security Portfolio

---

## What I found

The web server on this machine is a disaster — and that's putting it kindly. Running three different tools against port 80 painted a very clear picture of a server that was never hardened, never updated, and was essentially left wide open.

**The standout discoveries:**

- **phpMyAdmin is publicly accessible.** This is the web interface for managing the MySQL database. It's sitting at `/phpMyAdmin` with no IP restriction, no extra authentication layer, nothing. In a real environment this alone would be a critical finding — an attacker with default credentials (or a brute force tool) now has a GUI into your entire database.

- **phpinfo.php is exposed.** This single file dumps everything about the server — PHP version, loaded modules, file paths, environment variables, server configuration. It's essentially handing an attacker a complete map of the system. It should never exist on a production server.

- **The server broadcasts exactly what it's running.** The Netcat banner grab confirmed `Apache/2.2.8` and `PHP/5.2.4` in plain text in every response header. Apache 2.2.8 was released in 2008. PHP 5.2.4 reached end of life in 2011. Both have well-documented CVEs. Knowing the exact version turns a generic "web server" into a specific target.

- **WebDAV is enabled (`/dav`).** WebDAV is a protocol that allows files to be uploaded directly to a web server. On a misconfigured server like this, it's a potential file upload vector — meaning an attacker could potentially upload a web shell and execute code on the server.

- **Zero security headers.** Nikto flagged that the server is missing every modern browser security header — no Content-Security-Policy, no X-Content-Type-Options, no Strict-Transport-Security. These headers exist to prevent a whole class of attacks (XSS, clickjacking, MIME sniffing). Their absence isn't the end of the world on its own, but combined with everything else it paints a picture of a server that's never been security-reviewed.

- **HTTP TRACE is enabled.** This is a method that should never be on in production — it can be abused in Cross-Site Tracing (XST) attacks to steal cookies and credentials.

- **The HTML source revealed even more apps.** The Netcat response showed links to TWiki, phpMyAdmin, Mutillidae, DVWA, and WebDAV — all deliberately vulnerable web applications sitting on the same server.

---

## What I did and why

### Nikto — automated web vulnerability scan

```bash
nikto -h 192.168.122.60 -p 80 -o nikto-output.txt -timeout 10
```

Nikto runs through thousands of checks against a web server — known vulnerable files, misconfigurations, outdated software, exposed directories, missing security headers. Think of it as an automated checklist of everything a web server shouldn't be doing. The `-timeout 10` flag stops it hanging on slow responses from the target.

It confirmed the Apache and PHP versions, found phpMyAdmin and phpinfo.php exposed, flagged WebDAV, HTTP TRACE, and every missing security header.

### Gobuster — directory brute force

```bash
gobuster dir -u http://192.168.122.60 -w /usr/share/wordlists/dirb/common.txt -o gobuster-output.txt
```

Gobuster works differently to Nikto — instead of checking for known vulnerabilities, it just tries thousands of common directory and file names and sees what the server responds with. It found directories that Nikto didn't specifically flag, and confirmed which ones are accessible vs blocked. It scanned all 4,613 entries in the wordlist in under a minute.

### Netcat — manual HTTP GET / banner grab

```bash
nc 192.168.122.60 80
GET / HTTP/1.0
```

This is the most fundamental of the three. Instead of using a browser or a tool, you're talking to the web server directly in raw HTTP — exactly the way a browser does, but manually. The server's response headers reveal the software stack immediately: `Server: Apache/2.2.8 (Ubuntu) DAV/2` and `X-Powered-By: PHP/5.2.4`. The HTML body also revealed all the vulnerable apps linked on the homepage.

This matters because it shows you don't need a specialised tool to fingerprint a web server — basic knowledge of how HTTP works is enough.

---

## Gobuster findings

| Path | Status | What it means |
|------|--------|---------------|
| `/dav` | 301 | WebDAV enabled — potential file upload vector |
| `/phpMyAdmin` | 301 | Database admin panel publicly accessible |
| `/phpinfo` | 200 | Full PHP configuration exposed |
| `/phpinfo.php` | 200 | Same — accessible via both paths |
| `/index.php` | 200 | Main page |
| `/test` | 301 | Test directory accessible |
| `/twiki` | 301 | TWiki installation found |
| `/.htpasswd` | 403 | Exists but blocked — credentials file present |
| `/.htaccess` | 403 | Exists but blocked |
| `/cgi-bin/` | 403 | Exists but blocked |
| `/server-status` | 403 | Apache status page exists but blocked |

**Status codes:** 200 = fully accessible. 301 = redirects to the directory (accessible). 403 = exists but access refused.

---

## Nikto key findings

| Finding | Risk | Reference |
|---------|------|-----------|
| Apache 2.2.8 — severely outdated | High | Multiple CVEs, EOL since 2017 |
| PHP 5.2.4 — severely outdated | High | EOL since 2011 |
| phpinfo.php exposed | High | CWE-552 — full system info disclosure |
| phpMyAdmin accessible | High | Database admin panel, no access control |
| HTTP TRACE enabled | Medium | XST attack vector |
| WebDAV enabled | Medium | Potential file upload vector |
| /doc/ directory browsable | Medium | CVE-1999-0678 |
| PHP Easter Eggs enabled | Low | Version disclosure via query strings |
| Missing security headers (5) | Low | CSP, HSTS, X-Content-Type, Referrer, Permissions |
| Directory indexing on /icons/ | Low | Information disclosure |
| mod_negotiation enabled | Low | Aids filename brute forcing |

---

## Banner grab result

```
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Length: 891
Connection: close
Content-Type: text/html
```

This single response tells you the OS (Ubuntu), web server and exact version (Apache 2.2.8), scripting language and exact version (PHP 5.2.4), and that WebDAV is active — all without logging in or doing anything intrusive. Just one HTTP request.

---

## Why this matters

In a SOC role, these are exactly the kinds of findings that come up in vulnerability scan reports. Knowing what phpMyAdmin being exposed means, why an outdated PHP version is dangerous, or why HTTP TRACE should be disabled — that's the difference between someone who can read a report and someone who understands it.

For pentesting, this recon output directly feeds the next phase. phpMyAdmin with default credentials, WebDAV file upload, DVWA and Mutillidae for web app exploitation — every one of these is a potential foothold.

---

## Files

| File | Description |
|------|-------------|
| `nikto-output.txt` | Raw Nikto scan output |
| `gobuster-output.txt` | Raw Gobuster directory enumeration output |
| `screenshots/` | Gobuster complete, Nikto running, Netcat banner grab |
