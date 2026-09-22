# ADVANCED WEB RECONNAISSANCE COMPLETE CHEAT SHEET

## 1. WHAT IS ADVANCED WEB RECON?

Advanced web reconnaissance goes beyond simple port scanning and directory brute-forcing. It focuses on **content discovery**, **technology fingerprinting**, and **JavaScript analysis** to map the full attack surface of modern web applications.

**Tools covered:**
- **Katana** — JavaScript-aware web crawler
- **WhatWeb** — Technology fingerprinting
- **httpx** — HTTP probing with tech detection
- **naabu** — Fast port scanner
- **dnsx** — DNS enumeration
- **tlsx** — TLS analysis
- **waybackurls** — Historical URL discovery

**Key fact:** Modern web apps hide endpoints in JavaScript files. Traditional crawlers miss them — Katana doesn't.

---

## 2. KATANA — JAVASCRIPT-AWARE WEB CRAWLER

Katana is a next-generation crawling framework that parses JavaScript files to discover hidden endpoints, API routes, and dynamic URLs.

### 2.1 Basic Crawl

```bash
katana -u https://target.com
katana -u https://target.com -jc              # Enable JS crawling
katana -u https://target.com -jc -o endpoints.txt
katana -list urls.txt -jc -o all_endpoints.txt
```

### 2.2 Crawling Options

```bash
katana -u https://target.com -d 5              # Depth 5
katana -u https://target.com -ct 2m            # Crawl for 2 minutes
katana -u https://target.com -kf robotstxt,sitemapxml
katana -u https://target.com -s breadth-first
katana -u https://target.com -aff              # Auto form fill
katana -u https://target.com -fx               # Extract form elements
```

### 2.3 JavaScript Parsing

```bash
katana -u https://target.com -jc               # JS endpoint parsing
katana -u https://target.com -jsl              # jsluice parsing (memory intensive)
katana -u https://target.com -jc -xhr          # Extract XHR requests
```

### 2.4 Headless Crawling

```bash
katana -u https://target.com -headless -no-sandbox
katana -u https://target.com -headless -sc     # Use system Chrome
katana -u https://target.com -headless -cdd /path/to/chrome/data
katana -headless -u https://target.com -cwu ws://127.0.0.1:9222
```

### 2.5 Scope Control

```bash
katana -u https://target.com -cs '.*\.target\.com'    # In-scope regex
katana -u https://target.com -cos '.*\.external\.com' # Out-of-scope regex
katana -u https://target.com -fs fqdn                 # Scope by FQDN
katana -u https://target.com -ns                      # Disable scope
katana -u https://target.com -do                      # Display out-of-scope
```

### 2.6 Output Options

```bash
katana -u https://target.com -f qurl -silent          # Only URLs
katana -u https://target.com -jsonl -o output.jsonl   # JSONL format
katana -u https://target.com -o endpoints.txt -silent
```

### 2.7 Proxy and Headers

```bash
katana -u https://target.com -proxy http://127.0.0.1:8080
katana -u https://target.com -H "Authorization: Bearer token"
katana -u https://target.com -H "Cookie: session=abc123"
```

---

## 3. WHATWEB — TECHNOLOGY FINGERPRINTING

WhatWeb identifies CMS, web servers, JavaScript libraries, analytics, and embedded devices.

### 3.1 Basic Scan

```bash
whatweb https://target.com
whatweb -v https://target.com                  # Verbose
whatweb -a 3 https://target.com                # Aggressive
whatweb -a 4 https://target.com                # Heavy
```

### 3.2 Multiple Targets

```bash
whatweb target1.com target2.com
whatweb -i urls.txt                            # From file
whatweb --input-file urls.txt
whatweb 192.168.1.0/24                         # CIDR range
```

### 3.3 Plugin Selection

```bash
whatweb -p plugins/phpbb.rb https://target.com
whatweb -p wordpress,joomla https://target.com
```

### 3.4 Output

```bash
whatweb --log-json=results.json https://target.com
whatweb --log-brief=out.txt https://target.com
whatweb -i urls.txt --log-json=results.json
```

### 3.5 Stealth Options

```bash
whatweb -U "Mozilla/5.0" https://target.com   # Custom UA
whatweb -H "X-Forwarded-For: 127.0.0.1" https://target.com
whatweb --proxy 127.0.0.1:8080 https://target.com
whatweb -t 10 https://target.com               # Threads
```

### 3.6 Common Flags

```
-a LEVEL      Aggression level (1=stealthy, 3=aggressive, 4=heavy)
-p PLUGINS    Run only specified plugins
-t THREADS    Concurrent threads (default 25)
-i FILE       Input file with targets
--log-json    JSON output
--log-brief   Brief output
-U            User-Agent
-H            Custom header
--proxy       Proxy
```

---

## 4. HTTPX — HTTP PROBING & TECH DETECTION

httpx probes hosts for HTTP/HTTPS and detects technologies.

### 4.1 Basic Probing

```bash
httpx -u https://target.com
httpx -l hosts.txt
cat hosts.txt | httpx
```

### 4.2 Tech Detection

```bash
httpx -u https://target.com -tech-detect
httpx -u https://target.com -td
httpx -u https://target.com -title
httpx -u https://target.com -status-code
httpx -u https://target.com -server
```

### 4.3 Screenshots

```bash
httpx -u https://target.com -screenshot
httpx -l hosts.txt -screenshot -o screenshots/
```

### 4.4 Filtering

```bash
httpx -l hosts.txt -mc 200,301,302      # Match status codes
httpx -l hosts.txt -fc 404              # Filter status codes
httpx -l hosts.txt -fs 1234             # Filter by size
httpx -l hosts.txt -fl 50               # Filter by lines
```

### 4.5 Pipelines

```bash
subfinder -d target.com -silent | httpx -silent | nuclei -silent
cat domains.txt | httpx -silent -td -title -o results.txt
```

---

## 5. NAABU — FAST PORT SCANNER

```bash
naabu -host target.com
naabu -host target.com -p 80,443,8080
naabu -host target.com -top-ports 100
naabu -list hosts.txt -p - -o ports.txt
naabu -host target.com -rate 1000
```

---

## 6. DNSX — DNS ENUMERATION

```bash
dnsx -d target.com -a -resp
dnsx -d target.com -cname -resp
dnsx -d target.com -mx -resp
dnsx -d target.com -txt -resp
dnsx -l subdomains.txt -a -resp
```

---

## 7. TLSX — TLS ANALYSIS

```bash
tlsx -u target.com:443
tlsx -u target.com:443 -tls-version
tlsx -u target.com:443 -cipher
tlsx -u target.com:443 -jarm
tlsx -u target.com:443 -serial
```

---

## 8. WAYBACKURLS — HISTORICAL URL DISCOVERY

```bash
waybackurls target.com
waybackurls target.com | grep "\.js$"
waybackurls target.com | grep "api"
waybackurls target.com | sort -u > all_urls.txt
```

---

## 9. COMBINED RECON PIPELINES

### 9.1 Full Web Recon Pipeline

```bash
subfinder -d target.com -silent | httpx -silent | katana -jc -silent | nuclei -silent
```

### 9.2 JavaScript Endpoint Discovery

```bash
katana -u https://target.com -jc -f qurl -silent | grep "\.js$" | httpx -silent | nuclei -t exposures/
```

### 9.3 Technology-Specific Pipeline

```bash
whatweb -i live_hosts.txt --log-json=tech.json
cat tech.json | jq '.[] | select(.plugins.WordPress)' | jq -r '.target' > wordpress_hosts.txt
nuclei -l wordpress_hosts.txt -t wordpress/
```

---

## 10. TIPS

1. **Katana first** — it finds endpoints that ffuf/gobuster miss.
2. **WhatWeb before Nuclei** — knowing the stack lets you target templates.
3. **httpx as the middle layer** — filter live hosts before heavy tools.
4. Use `-jc` on Katana always — 80% of modern endpoints are in JS.
5. Combine `waybackurls` with `httpx` for historical endpoint validation.
6. **naabu → httpx → nuclei** is the fastest pipeline for large scopes.
7. Always save output — recon is iterative, not one-shot.
8. Use `-silent` for pipeline-friendly output.
9. **Never** crawl without authorization — crawling is active scanning.
10. Combine `dnsx` + `tlsx` to map the full TLS/DNS surface.
