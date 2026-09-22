# NMAP PENETRATION TESTING COMPLETE CHEAT SHEET

## 1. WHAT IS NMAP?

**Nmap (Network Mapper)** is the industry-standard tool for network discovery and security auditing. It is used for:

- **Host discovery** (which hosts are alive)
- **Port scanning** (which ports are open)
- **Service/version detection** (what's running)
- **OS fingerprinting** (which OS)
- **Vulnerability detection** (via NSE scripts)
- **Network mapping** (topology and inventory)

**Key fact:** Nmap is the first tool you run in any pentest. It defines the attack surface.

---

## 2. BASIC SYNTAX

```bash
nmap [Scan Type] [Options] {target specification}
```

**Targets:**
```bash
nmap <IP>                       # Single IP
nmap 192.168.1.0/24             # CIDR range
nmap 192.168.1.1-50             # Range
nmap <host1> <host2>            # Multiple hosts
nmap -iL targets.txt            # From file
nmap 10.0-5.0-255.1-254         # Multiple octet ranges
```

---

## 3. HOST DISCOVERY

```bash
nmap -sn <range>                # Ping scan (no port scan)
nmap -Pn <IP>                   # Skip ping (assume host is up)
nmap -PS <IP>                   # TCP SYN ping
nmap -PA <IP>                   # TCP ACK ping
nmap -PU <IP>                   # UDP ping
nmap -PE <IP>                   # ICMP echo ping
nmap -PP <IP>                   # ICMP timestamp ping
nmap -PM <IP>                   # ICMP netmask ping
nmap -PR <IP>                   # ARP ping (local network only)
nmap -n <IP>                    # No DNS resolution
nmap -R <IP>                    # Always resolve DNS
```

---

## 4. PORT SCANNING

```bash
nmap -p 22 <IP>                 # Single port
nmap -p 22,80,443 <IP>          # Multiple ports
nmap -p 1-1000 <IP>             # Range
nmap -p- <IP>                   # All 65535 ports
nmap --top-ports 20 <IP>        # Top 20 most common
nmap --top-ports 100 <IP>       # Top 100
nmap -p U:53,T:80 <IP>          # Mixed UDP/TCP
nmap -p http* <IP>              # By service name
```

---

## 5. SCAN TYPES

```bash
nmap -sS <IP>                   # TCP SYN (stealth, fast, needs root)
nmap -sT <IP>                   # TCP Connect (no root needed)
nmap -sU <IP>                   # UDP scan
nmap -sA <IP>                   # TCP ACK (firewall mapping)
nmap -sN <IP>                   # TCP NULL
nmap -sF <IP>                   # TCP FIN
nmap -sX <IP>                   # TCP Xmas
nmap -sW <IP>                   # TCP Window
nmap -sM <IP>                   # TCP Maimon
nmap -sI <zombie> <IP>          # Idle scan
nmap -sY <IP>                   # SCTP INIT
nmap -sZ <IP>                   # SCTP COOKIE-ECHO
nmap -sO <IP>                   # IP protocol scan
nmap --scanflags SYNACK <IP>    # Custom TCP flags
```

---

## 6. SERVICE / VERSION DETECTION

```bash
nmap -sV <IP>                   # Service version detection
nmap -sV --version-intensity 9 <IP>  # Aggressive version detection
nmap -sV --version-light <IP>   # Light version detection
nmap -sV --version-all <IP>     # Try all probes
```

---

## 7. OS DETECTION

```bash
nmap -O <IP>                    # OS detection
nmap -O --osscan-guess <IP>     # Aggressive OS guessing
nmap -O --osscan-limit <IP>     # Limit OS detection
nmap -A <IP>                    # OS + version + traceroute + scripts
```

---

## 8. OUTPUT FORMATS

```bash
nmap -oN result.txt <IP>        # Normal output
nmap -oX result.xml <IP>        # XML output
nmap -oG result.gnmap <IP>      # Greppable output
nmap -oA result <IP>            # All formats at once
nmap -oN - <IP>                 # Output to stdout
nmap --append-output -oN result.txt <IP>  # Append
nmap -v <IP>                    # Verbose
nmap -vv <IP>                   # More verbose
nmap --reason <IP>              # Show reason for state
nmap --stats-every 5s <IP>      # Show stats every 5s
nmap --open <IP>                # Show only open ports
```

---

## 9. TIMING AND PERFORMANCE

```bash
nmap -T0 <IP>                   # Paranoid (very slow, IDS evasion)
nmap -T1 <IP>                   # Sneaky
nmap -T2 <IP>                   # Polite (less bandwidth)
nmap -T3 <IP>                   # Normal (default)
nmap -T4 <IP>                   # Aggressive (recommended)
nmap -T5 <IP>                   # Insane (fastest, may miss)
nmap --min-rate 1000 <IP>       # Minimum packets/sec
nmap --max-rate 1000 <IP>       # Maximum packets/sec
nmap --min-parallelism 100 <IP> # Min parallelism
nmap --max-retries 1 <IP>       # Max retries
nmap --host-timeout 30m <IP>    # Host timeout
nmap --scan-delay 1s <IP>       # Delay between probes
nmap --max-scan-delay 5s <IP>   # Max delay
```

---

## 10. FIREWALL / IDS EVASION

```bash
nmap -f <IP>                    # Fragment packets
nmap -ff <IP>                   # Fragment into 8-byte chunks
nmap --mtu 16 <IP>              # Custom MTU
nmap -D RND:10 <IP>             # 10 random decoys
nmap -D decoy1,decoy2,ME <IP>   # Specific decoys
nmap -S <spoofed_IP> <IP>       # IP spoofing
nmap -e eth0 <IP>               # Specify interface
nmap --source-port 53 <IP>      # Source port (DNS)
nmap -g 53 <IP>                 # Same as --source-port
nmap --data-length 25 <IP>      # Add random data
nmap --randomize-hosts <IP>     # Randomize target order
nmap --spoof-mac 0 <IP>         # Random MAC
nmap --spoof-mac Cisco <IP>     # Spoof vendor MAC
nmap --proxies http://proxy:8080 <IP>  # Use proxy
nmap --badsum <IP>              # Send bad checksums
nmap --send-eth <IP>            # Send raw Ethernet
nmap --send-ip <IP>             # Send raw IP
```

---

## 11. NSE SCRIPTS (NMAP SCRIPTING ENGINE)

```bash
nmap --script=default <IP>              # Default scripts
nmap --script=vuln <IP>                 # Vulnerabilities
nmap --script=safe <IP>                 # Safe scripts
nmap --script=intrusive <IP>            # Intrusive scripts
nmap --script=malware <IP>              # Malware detection
nmap --script=discovery <IP>            # Discovery
nmap --script=auth <IP>                 # Authentication
nmap --script=broadcast <IP>            # Broadcast
nmap --script=exploit <IP>              # Exploits
nmap --script=fuzzer <IP>               # Fuzzers
nmap --script=version <IP>              # Version detection
nmap --script=http* <IP>                # All HTTP scripts
nmap --script=http-title <IP>           # HTTP title
nmap --script=http-headers <IP>         # HTTP headers
nmap --script=http-enum <IP>            # HTTP enumeration
nmap --script=http-vuln* <IP>           # HTTP vulnerabilities
nmap --script=smb* <IP>                 # All SMB scripts
nmap --script=smb-vuln* <IP>            # SMB vulnerabilities
nmap --script=smb-enum-shares <IP>      # SMB shares
nmap --script=smb-enum-users <IP>       # SMB users
nmap --script=smb-os-discovery <IP>     # SMB OS discovery
nmap --script=ssl* <IP>                 # SSL scripts
nmap --script=ssl-heartbleed <IP>       # Heartbleed detection
nmap --script=ssl-enum-ciphers <IP>     # SSL ciphers
nmap --script=dns-* <IP>                # DNS scripts
nmap --script=dns-zone-transfer <IP>    # DNS zone transfer
nmap --script=ftp-* <IP>                # FTP scripts
nmap --script=ftp-anon <IP>             # Anonymous FTP
nmap --script=ssh-* <IP>                # SSH scripts
nmap --script=mysql-* <IP>              # MySQL scripts
nmap --script=mongodb-* <IP>            # MongoDB scripts
nmap --script=ldap-* <IP>               # LDAP scripts
nmap --script=rdp-* <IP>                # RDP scripts
nmap --script=vnc-* <IP>                # VNC scripts
```

**NSE script with arguments:**
```bash
nmap --script=http-brute --script-args userdb=users.txt,passdb=passwords.txt <IP>
nmap --script=ssl-enum-ciphers --script-args tls.servername=example.com <IP>
nmap --script smb-brute --script-args smbuser=admin,smbpass=admin <IP>
```

**Update NSE scripts:**
```bash
sudo nmap --script-updatedb
```

---

## 12. COMMON SCAN COMBINATIONS

### 12.1 Quick scan
```bash
nmap -T4 -F <IP>
nmap -T4 --top-ports 100 <IP>
```

### 12.2 Full port scan + version + scripts
```bash
nmap -p- -sV -sC -T4 -A <IP>
```

### 12.3 Stealth scan
```bash
nmap -sS -T2 -f -D RND:10 --randomize-hosts <IP>
```

### 12.4 UDP scan (top 100)
```bash
nmap -sU --top-ports 100 -T4 <IP>
```

### 12.5 Vulnerability scan
```bash
nmap --script=vuln -sV <IP>
```

### 12.6 Fast network sweep
```bash
nmap -sn 192.168.1.0/24
```

### 12.7 Aggressive all-in-one
```bash
nmap -A -T4 -v -p- <IP>
```

### 12.8 Save all formats
```bash
nmap -A -T4 -p- -oA scan_<IP> <IP>
```

---

## 13. SCRIPTING / AUTOMATION

### 13.1 Scan multiple hosts from file
```bash
nmap -iL targets.txt -oA results
```

### 13.2 Scan excluding hosts
```bash
nmap 192.168.1.0/24 --exclude 192.168.1.1,192.168.1.10
nmap 192.168.1.0/24 --excludefile exclude.txt
```

### 13.3 Scan with dynamic targets
```bash
nmap -sn 192.168.1.0/24 -oG - | awk '/Up$/{print $2}' > live_hosts.txt
nmap -iL live_hosts.txt -p- -oA full_scan
```

### 13.4 Bash loop scanning
```bash
#!/bin/bash
for ip in $(cat ips.txt); do
    echo "[*] Scanning $ip"
    nmap -T4 -A -p- -oA scan_$ip $ip
done
```

---

## 14. TROUBLESHOOTING

```bash
nmap -Pn <IP>                   # If host seems down but isn't
sudo nmap -sS <IP>              # If SYN scan requires root
nmap --unprivileged <IP>        # Avoid privileged scans
nmap -n <IP>                    # Skip DNS (speed)
nmap -sV --version-trace <IP>   # Debug version detection
```

---

## 15. TOOLS COMPLEMENTARY TO NMAP

| Tool | Usage |
|------|-------|
| **masscan** | Very fast port scanner |
| **rustscan** | Fast port scanner, feeds into nmap |
| **naabu** | Fast port scanner by ProjectDiscovery |
| **Zenmap** | GUI for nmap |
| **nmap-vulners** | NSE script for CVE detection |
| **nmap-nse-vuln** | Additional vulnerability scripts |

**RustScan + Nmap combo:**
```bash
rustscan -a <IP> --ulimit 5000 -- -sV -sC -A
```

**Masscan:**
```bash
sudo masscan -p1-65535 <IP> --rate=1000
```

---

## 16. TIPS FOR TESTING

1. Start with `-sn` for host discovery.
2. Use `-p-` for full port scan (some services hide on high ports).
3. Follow with `-sV -sC` for versions and default scripts.
4. Use `-A` when you want everything (`-O -sV -sC --traceroute`).
5. Save output in all formats with `-oA` for easy parsing.
6. Use `--script=vuln` for vulnerability detection.
7. Combine with `--script=http-title` for quick web enumeration.
8. Reduce noise with `-T2` or add decoys with `-D RND:10` when stealth matters.
9. Skip ping (`-Pn`) when ICMP is blocked.
10. Use `-n` to speed up scans (no DNS).
11. Always scan authorized targets only.

---

## 17. QUICK REFERENCE – MOST USED COMMANDS

| Purpose | Command |
|---------|---------|
| Host discovery | `nmap -sn <range>` |
| Fast scan | `nmap -T4 -F <IP>` |
| Full ports | `nmap -p- <IP>` |
| Version + scripts | `nmap -sV -sC <IP>` |
| Aggressive | `nmap -A -T4 <IP>` |
| UDP top 100 | `nmap -sU --top-ports 100 <IP>` |
| Vuln scan | `nmap --script=vuln <IP>` |
| Stealth | `nmap -sS -T2 -f <IP>` |
| Save all | `nmap -oA output <IP>` |
| Fast + full | `nmap -T4 -p- -A -oA output <IP>` |

---

## ALL IN ONE 

```
nmap <IP>
nmap 192.168.1.0/24
nmap 192.168.1.1-50
nmap <host1> <host2>
nmap -iL targets.txt
nmap 10.0-5.0-255.1-254
nmap -sn <range>
nmap -Pn <IP>
nmap -PS <IP>
nmap -PA <IP>
nmap -PU <IP>
nmap -PE <IP>
nmap -PP <IP>
nmap -PM <IP>
nmap -PR <IP>
nmap -n <IP>
nmap -R <IP>
nmap -p 22 <IP>
nmap -p 22,80,443 <IP>
nmap -p 1-1000 <IP>
nmap -p- <IP>
nmap --top-ports 20 <IP>
nmap --top-ports 100 <IP>
nmap -p U:53,T:80 <IP>
nmap -p http* <IP>
nmap -sS <IP>
nmap -sT <IP>
nmap -sU <IP>
nmap -sA <IP>
nmap -sN <IP>
nmap -sF <IP>
nmap -sX <IP>
nmap -sW <IP>
nmap -sM <IP>
nmap -sI <zombie> <IP>
nmap -sY <IP>
nmap -sZ <IP>
nmap -sO <IP>
nmap --scanflags SYNACK <IP>
nmap -sV <IP>
nmap -sV --version-intensity 9 <IP>
nmap -sV --version-light <IP>
nmap -sV --version-all <IP>
nmap -O <IP>
nmap -O --osscan-guess <IP>
nmap -O --osscan-limit <IP>
nmap -A <IP>
nmap -oN result.txt <IP>
nmap -oX result.xml <IP>
nmap -oG result.gnmap <IP>
nmap -oA result <IP>
nmap -oN - <IP>
nmap --append-output -oN result.txt <IP>
nmap -v <IP>
nmap -vv <IP>
nmap --reason <IP>
nmap --stats-every 5s <IP>
nmap --open <IP>
nmap -T0 <IP>
nmap -T1 <IP>
nmap -T2 <IP>
nmap -T3 <IP>
nmap -T4 <IP>
nmap -T5 <IP>
nmap --min-rate 1000 <IP>
nmap --max-rate 1000 <IP>
nmap --min-parallelism 100 <IP>
nmap --max-retries 1 <IP>
nmap --host-timeout 30m <IP>
nmap --scan-delay 1s <IP>
nmap --max-scan-delay 5s <IP>
nmap -f <IP>
nmap -ff <IP>
nmap --mtu 16 <IP>
nmap -D RND:10 <IP>
nmap -D decoy1,decoy2,ME <IP>
nmap -S <spoofed_IP> <IP>
nmap -e eth0 <IP>
nmap --source-port 53 <IP>
nmap -g 53 <IP>
nmap --data-length 25 <IP>
nmap --randomize-hosts <IP>
nmap --spoof-mac 0 <IP>
nmap --spoof-mac Cisco <IP>
nmap --proxies http://proxy:8080 <IP>
nmap --badsum <IP>
nmap --send-eth <IP>
nmap --send-ip <IP>
nmap --script=default <IP>
nmap --script=vuln <IP>
nmap --script=safe <IP>
nmap --script=intrusive <IP>
nmap --script=malware <IP>
nmap --script=discovery <IP>
nmap --script=auth <IP>
nmap --script=broadcast <IP>
nmap --script=exploit <IP>
nmap --script=fuzzer <IP>
nmap --script=version <IP>
nmap --script=http* <IP>
nmap --script=http-title <IP>
nmap --script=http-headers <IP>
nmap --script=http-enum <IP>
nmap --script=http-vuln* <IP>
nmap --script=smb* <IP>
nmap --script=smb-vuln* <IP>
nmap --script=smb-enum-shares <IP>
nmap --script=smb-enum-users <IP>
nmap --script=smb-os-discovery <IP>
nmap --script=ssl* <IP>
nmap --script=ssl-heartbleed <IP>
nmap --script=ssl-enum-ciphers <IP>
nmap --script=dns-* <IP>
nmap --script=dns-zone-transfer <IP>
nmap --script=ftp-* <IP>
nmap --script=ftp-anon <IP>
nmap --script=ssh-* <IP>
nmap --script=mysql-* <IP>
nmap --script=mongodb-* <IP>
nmap --script=ldap-* <IP>
nmap --script=rdp-* <IP>
nmap --script=vnc-* <IP>
nmap --script=http-brute --script-args userdb=users.txt,passdb=passwords.txt <IP>
nmap --script=ssl-enum-ciphers --script-args tls.servername=example.com <IP>
nmap --script smb-brute --script-args smbuser=admin,smbpass=admin <IP>
sudo nmap --script-updatedb
nmap -T4 -F <IP>
nmap -T4 --top-ports 100 <IP>
nmap -p- -sV -sC -T4 -A <IP>
nmap -sS -T2 -f -D RND:10 --randomize-hosts <IP>
nmap -sU --top-ports 100 -T4 <IP>
nmap --script=vuln -sV <IP>
nmap -sn 192.168.1.0/24
nmap -A -T4 -v -p- <IP>
nmap -A -T4 -p- -oA scan_<IP> <IP>
nmap -iL targets.txt -oA results
nmap 192.168.1.0/24 --exclude 192.168.1.1,192.168.1.10
nmap 192.168.1.0/24 --excludefile exclude.txt
nmap -sn 192.168.1.0/24 -oG - | awk '/Up$/{print $2}' > live_hosts.txt
nmap -iL live_hosts.txt -p- -oA full_scan
nmap -Pn <IP>
sudo nmap -sS <IP>
nmap --unprivileged <IP>
nmap -n <IP>
nmap -sV --version-trace <IP>
rustscan -a <IP> --ulimit 5000 -- -sV -sC -A
sudo masscan -p1-65535 <IP> --rate=1000
```
