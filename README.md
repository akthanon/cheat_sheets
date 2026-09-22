# 🛡️ PENTESTING CHEAT SHEETS COLLECTION

Personal collection of pentesting cheat sheets, organized by phase and domain. Each file is a self-contained document, ready to copy/paste commands and payloads during an authorized engagement.

**Last updated:** 2026
**Total cheat sheets:** 48
**Language:** English (payloads and code in English by convention)

---

## ⚠️ LEGAL AND ETHICAL NOTICE

This collection is **exclusively for educational use and authorized testing**.

- ✅ **ALLOWED:** Your own systems, isolated labs, CTFs, environments with written authorization.
- ❌ **FORBIDDEN:** Third-party systems, public networks, any activity without explicit consent.

Using these techniques without authorization is **illegal** in most jurisdictions. The author is not responsible for misuse.

---

## 📂 STRUCTURE

```
Pentesting/
├── WEB/         → 21 web vulnerability cheat sheets
├── RECON/       → 6 reconnaissance and enumeration cheat sheets
├── INFRA/       → 11 infrastructure, network and post-exploitation cheat sheets
├── CREDS/       → 2 cracking and brute-force cheat sheets
└── PAYLOADS/    → 8 shells, payloads and tooling cheat sheets
```

---

## 🌐 WEB/ — Web Vulnerabilities (21)

| File                                                               | Description                                                          |
| ------------------------------------------------------------------ | -------------------------------------------------------------------- |
| [`API Security Testing.md`](WEB/API%20Security%20Testing.md)       | OWASP API Top 10, BOLA, BFLA, mass assignment, JWT, GraphQL, tools   |
| [`CSRF.md`](WEB/CSRF.md)                                           | Cross-Site Request Forgery, GET/POST/JSON, token bypass              |
| [`Command Injection.md`](WEB/Command%20Injection.md)               | OS command injection, separators, time-based, OOB, bypasses          |
| [`DOM Based XSS.md`](WEB/DOM%20Based%20XSS.md)                     | Sources, sinks, payloads, postMessage, localStorage                  |
| [`File Upload.md`](WEB/File%20Upload.md)                           | Extension bypass, magic bytes, content-type, web shells              |
| [`GraphQL Injection.md`](WEB/GraphQL%20Injection.md)               | Introspection, batching, aliasing, IDOR, DoS, mutations              |
| [`HEADERS.md`](WEB/HEADERS.md)                                     | Offensive headers, JWT, GraphQL, NoSQLi, deserialization, WAF bypass |
| [`IDOR.md`](WEB/IDOR.md)                                           | Insecure Direct Object Reference, parameters, bypasses               |
| [`INSECURE DESERIALIZATION.md`](WEB/INSECURE%20DESERIALIZATION.md) | PHP, Java, Python, .NET, Ruby, Node.js, POP chains                   |
| [`JWT.md`](WEB/JWT.md)                                             | None algorithm, alg confusion, kid injection, jku/x5u, brute force   |
| [`LDAP XPath Injection.md`](WEB/LDAP%20XPath%20Injection.md)       | LDAP and XPath, auth bypass, data extraction                         |
| [`NoSQLi.md`](WEB/NoSQLi.md)                                       | MongoDB operators, `$ne`, `$regex`, `$where`, JavaScript injection   |
| [`Open Redirect.md`](WEB/Open%20Redirect.md)                       | Generic payloads, allowlist bypass, chaining with SSRF               |
| [`Path Traversal LFI RFI.md`](WEB/Path%20Traversal%20LFI%20RFI.md) | Directory traversal, LFI, RFI, PHP wrappers, log poisoning           |
| [`Race Conditions.md`](WEB/Race%20Conditions.md)                   | TOCTOU, limit overrun, HTTP/2 single-packet attack, Python script    |
| [`SQLi.md`](WEB/SQLi.md)                                           | SQL Injection, UNION, error-based, boolean, time-based, bypasses     |
| [`SSRF.md`](WEB/SSRF.md)                                           | Server-Side Request Forgery, cloud metadata, gopher, bypasses        |
| [`SSTI.md`](WEB/SSTI.md)                                           | Jinja2, Twig, Freemarker, Velocity, Pug, EJS, ERB, bypasses          |
| [`Wordpress.md`](WEB/Wordpress.md)                                 | Enumeration, wp-config, xmlrpc, REST API, WPScan, brute force        |
| [`XSS.md`](WEB/XSS.md)                                             | Reflected, stored, DOM, encoding, WAF bypass, polyglots              |
| [`XXE.md`](WEB/XXE.md)                                             | XML External Entity, file read, blind OOB, DoS, wrappers             |

---

## 🔍 RECON/ — Reconnaissance & Enumeration (6)

| File | Description |
|------|-------------|
| [`ADVANCED WEB RECON.md`](RECON/ADVANCED%20WEB%20RECON.md) | Katana, WhatWeb, httpx, naabu, dnsx, tlsx, waybackurls, pipelines |
| [`Linux Network Recon & Sniffing.md`](RECON/Linux%20Network%20Recon%20%26%20Sniffing.md) | ip, ss, netstat, tcpdump, tshark, WiFi, IPv6, DNS/SNI sniffing |
| [`NMAP.md`](RECON/NMAP.md) | Scans, NSE scripts, evasion, timing, output, automation |
| [`OSINT.md`](RECON/OSINT.md) | theHarvester, SpiderFoot, Sherlock, Holehe, Shodan, exiftool, whois, Maltego |
| [`Subdomain Takeover.md`](RECON/Subdomain%20Takeover.md) | Detection, fingerprints, subjack, SubOver, dnsReaper, mitigation |
| [`Web Recon Tools.md`](RECON/Web%20Recon%20Tools.md) | ffuf, Gobuster, CeWL, Nuclei, Masscan, pipelines |

---

## 🏗️ INFRA/ — Infrastructure & Network (11)

| File | Description |
|------|-------------|
| [`Active Directory.md`](INFRA/Active%20Directory.md) | Enumeration, Kerberoasting, AS-REP, BloodHound, multi-DC scripts |
| [`BlueTooth.md`](INFRA/BlueTooth.md) | hciconfig, hcitool, bluetoothctl, sdptool, BLE, btlejack, KNOB, BIAS |
| [`Cloud Pentesting.md`](INFRA/Cloud%20Pentesting.md) | AWS, Azure, GCP, IMDS, S3, IAM, containers (Docker/K8s), CI/CD |
| [`Firewall, IDS & IPS.md`](INFRA/Firewall,%20IDS%20%26%20IPS.md) | UFW, iptables, fail2ban, Suricata, Snort |
| [`Linux System & Desktop Tuning.md`](INFRA/Linux%20System%20%26%20Desktop%20Tuning.md) | bashrc, fastfetch, systemd, TLP, powertop, KDE reset, fstab |
| [`MITM Tools.md`](INFRA/MITM%20Tools.md) | Bettercap, Ettercap, mitmproxy, ARP/DNS spoofing, filters |
| [`POST-EXPLOITATION.md`](INFRA/POST-EXPLOITATION.md) | Sliver C2, Ligolo-ng, Chisel, persistence, exfiltration, covering tracks |
| [`Pivoting & Privilege Escalation.md`](INFRA/Pivoting%20%26%20Privilege%20Escalation.md) | SSH tunneling, linpeas, SUID, capabilities, pivoting from Linux |
| [`SSH Pivoting, Servers & Infrastructure.md`](INFRA/SSH%20Pivoting,%20Servers%20%26%20Infrastructure.md) | SSH tunnels, ngrok, cloudflared, vsftpd, Hydra SSH |
| [`WiFi Attacks.md`](INFRA/WiFi%20Attacks.md) | Aircrack-ng, Wifite, Wifipumpkin3, handshake, PMKID, evil twin |
| [`Windows AD Tooling.md`](INFRA/Windows%20AD%20Tooling.md) | smbclient, evil-winrm, Responder, Impacket, winPEAS, MSSQL |

---

## 🔑 CREDS/ — Credentials (2)

| File | Description |
|------|-------------|
| [`Credential Attacks.md`](CREDS/Credential%20Attacks.md) | Hydra (SSH, FTP, HTTP, RDP, MySQL, VNC), Pipal (dump analysis) |
| [`Hashcat & John.md`](CREDS/Hashcat%20%26%20John.md) | Modes, attacks, masks, rules, wordlists, GPU/CPU optimization |

---

## 🎯 PAYLOADS/ — Shells, Payloads & Tooling (8)

| File | Description |
|------|-------------|
| [`Anonymity, Tor & Offensive Tools.md`](PAYLOADS/Anonymity,%20Tor%20%26%20Offensive%20Tools.md) | Tor, proxychains, hidden services, SET, hping3, cameradar, pip config |
| [`BufferOverflow.md`](PAYLOADS/BufferOverflow.md) | GDB, pwndbg, pwntools, Ghidra, ROP, ret2libc, QEMU, Windows debuggers |
| [`Cross-Compilation & Python RE.md`](PAYLOADS/Cross-Compilation%20%26%20Python%20RE.md) | Wine, box64, MinGW, PyInstaller extraction, pycdc, uncompyle6 |
| [`Dotnet.md`](PAYLOADS/Dotnet.md) | .NET malware analysis, de4dot, ILSpy, dnSpy, secret extraction, Supabase testing |
| [`Metasploit & Payload Tools.md`](PAYLOADS/Metasploit%20%26%20Payload%20Tools.md) | msfconsole, msfvenom, Meterpreter, handler, encoders, formats |
| [`Netcat.md`](PAYLOADS/Netcat.md) | nc, ncat, socat, reverse/bind shells, alternatives to `-e`, TTY upgrade |
| [`OpenSSL.md`](PAYLOADS/OpenSSL.md) | Keys, certificates, CSR, PEM/DER/PFX, TLS testing, encryption, basic CA |
| [`Specific Exploits & Privesc.md`](PAYLOADS/Specific%20Exploits%20%26%20Privesc.md) | JNDI/Log4Shell, MongoDB privesc, vi/vim escape, base64 tricks |

---

## 🎓 RECOMMENDED WORKFLOW

```
1. RECON/         → Passive and active enumeration (OSINT, Nmap, subdomain takeover)
2. RECON/         → Advanced web recon (Katana, WhatWeb, httpx, Nuclei)
3. WEB/           → Web vulnerability testing (XSS, SQLi, SSRF, etc.)
4. API Security   → API-specific testing
5. INFRA/         → Internal movement, AD, WiFi, MITM
6. CREDS/         → Cracking captured hashes
7. PAYLOADS/      → Shells, payloads, listeners, post-exploitation
8. INFRA/POST-EX  → C2, pivoting, persistence, exfiltration
```

---

## 📌 PENDING / FUTURE TOPICS

Ideas to expand the collection:

- **OAuth / SAML / SSO Attacks** (modern auth, redirect_uri, token theft)
- **HTTP Request Smuggling** (CL.TE, TE.CL, TE.TE)
- **Prototype Pollution** (Node.js, potential RCE)
- **WebSocket Pentesting** (CSWSH, message injection)
- **Mobile Pentesting (Android)** (APK, Frida, SSL pinning bypass)
- **Format String & Heap Exploitation** (extends BO)
- **Active Directory Advanced** (ADCS, ESC1-ESC8, delegation)
- **Red Team Infra** (redirectors, malleable C2 profiles, phishing infrastructure)
- **Advanced Reverse Engineering** (IDA Pro, Binary Ninja, Ghidra scripting)

---

## 🛠️ CROSS-CUTTING TOOLS

The following tools appear across multiple cheat sheets:

**Recon & Enumeration:**
`nmap`, `masscan`, `naabu`, `httpx`, `katana`, `whatweb`, `gobuster`, `ffuf`, `nuclei`, `subfinder`, `amass`, `dnsx`, `tlsx`, `waybackurls`

**Web:**
`Burp Suite`, `sqlmap`, `jwt_tool`, `Commix`, `LFISuite`, `Tplmap`, `SSTImap`, `graphql-cop`

**Infra & AD:**
`impacket`, `crackmapexec`, `bloodhound`, `responder`, `ntlmrelayx`, `evil-winrm`, `ldapdomaindump`

**Cracking:**
`hashcat`, `john`, `hydra`, `pipal`, `cewl`

**Payloads & C2:**
`msfvenom`, `msfconsole`, `sliver`, `ligolo-ng`, `chisel`, `socat`, `netcat`, `ncat`

**Cloud:**
`pacu`, `scoutsuite`, `prowler`, `cloudfox`, `roadtools`, `trufflehog`

**Post-Exploitation:**
`linpeas`, `winpeas`, `pspy`, `mimikatz`, `sliver`, `ligolo-ng`

---

## 📚 REFERENCES

- **OWASP Top 10** — https://owasp.org/Top10/
- **OWASP API Top 10** — https://owasp.org/API-Security/
- **PortSwigger Web Security Academy** — https://portswigger.net/web-security
- **PayloadsAllTheThings** — https://github.com/swisskyrepo/PayloadsAllTheThings
- **HackTricks** — https://book.hacktricks.xyz/
- **PEASS-ng** — https://github.com/peass-ng/PEASS-ng
- **GTFOBins** — https://gtfobins.github.io/
- **LOLBAS** — https://lolbas-project.github.io/
- **Can I Take Over XYZ** — https://github.com/EdOverflow/can-i-take-over-xyz

---
