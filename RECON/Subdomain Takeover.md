# SUBDOMAIN TAKEOVER COMPLETE CHEAT SHEET

## 1. WHAT IS SUBDOMAIN TAKEOVER?

**Subdomain Takeover** occurs when a subdomain (e.g., `blog.example.com`) points via DNS (CNAME) to a third-party service (e.g., GitHub Pages, Heroku, S3) that is **no longer claimed**. An attacker can register the resource and serve content under the victim's subdomain.

**Impact:**
- Phishing on trusted domain
- Cookie theft (if cookies scoped to `.example.com`)
- Bypass of CSP allowlists
- Session hijacking
- Reputation damage

**Key fact:** It's a **dangling DNS record** problem, not a DNS bug. OWASP classifies it as a common misconfiguration.

---

## 2. HOW IT HAPPENS

```
1. Company creates blog.example.com → CNAME → myblog.github.io
2. Company stops using GitHub Pages → deletes the repo
3. DNS CNAME is NOT removed → still points to myblog.github.io
4. Attacker creates GitHub Pages with that name → takes over
```

**Common vulnerable services:**
- GitHub Pages (`*.github.io`)
- Heroku (`*.herokuapp.com`)
- AWS S3 (`*.s3.amazonaws.com`)
- Azure (`*.azurewebsites.net`, `*.cloudapp.azure.com`)
- Shopify, Fastly, Netlify, Vercel, Zendesk, Tumblr
- Surge.sh, Bitbucket, Pantheon, WordPress.com

---

## 3. DETECTION WORKFLOW

### 3.1 Enumerate Subdomains

```bash
subfinder -d target.com -o subs.txt
amass enum -d target.com -o subs2.txt
assetfinder target.com > subs3.txt
cat subs*.txt | sort -u > all_subs.txt
```

### 3.2 Resolve CNAMEs

```bash
# With dnsx
dnsx -l all_subs.txt -cname -resp -o cnames.txt

# With dig
for sub in $(cat all_subs.txt); do
    echo "=== $sub ==="
    dig +short CNAME $sub
done
```

### 3.3 Check for Vulnerable Patterns

```bash
# Bulk CNAME check
cat all_subs.txt | while read sub; do
    cname=$(dig +short CNAME $sub)
    if [ -n "$cname" ]; then
        echo "$sub -> $cname"
    fi
done
```

Look for CNAMEs pointing to:
```
*.github.io
*.herokuapp.com
*.s3.amazonaws.com
*.azurewebsites.net
*.cloudapp.azure.com
*.fastly.net
*.shopify.com
*.netlify.app
*.vercel.app
*.surge.sh
*.ghost.io
*.wordpress.com
*.tumblr.com
*.zendesk.com
*.bitbucket.io
*.pantheonsite.io
```

### 3.4 Verify Takeover-ability

Each service has an error page indicating the resource doesn't exist:

| Service | Error Signature |
|---------|-----------------|
| GitHub Pages | "There isn't a GitHub Pages site here." |
| Heroku | "No such app" |
| S3 | "NoSuchBucket" |
| Azure | "404 Web Site not found" |
| Shopify | "Sorry, this shop is currently unavailable." |
| Fastly | "Fastly error: unknown domain" |
| Netlify | "Not Found - Request ID" |
| Zendesk | "Help Center Closed" |
| Tumblr | "There's nothing here." |
| WordPress | "Do you want to register" |

```bash
# Test each subdomain for the error signature
for sub in $(cat cnames.txt); do
    echo "=== $sub ==="
    curl -sI "http://$sub" | head -5
done
```

---

## 4. TOOLS

### 4.1 Subjack

```bash
go install github.com/haccer/subjack@latest
subjack -w all_subs.txt -t 100 -timeout 30 -o results.txt -ssl
```

### 4.2 SubOver

```bash
go install github.com/Ice3man543/SubOver@latest
SubOver -l all_subs.txt
```

### 4.3 Nuclei — Subdomain Takeover Templates

```bash
nuclei -l all_subs.txt -t takeovers/ -o takeovers.txt
```

### 4.4 Can-I-Take-Over-XYZ (Online Resource)

Repository of fingerprints:
```
https://github.com/EdOverflow/can-i-take-over-xyz
```

### 4.5 dnsReaper

```bash
pip install dnsreaper
dnsreaper --filename all_subs.txt
```

---

## 5. EXPLOITATION

Once confirmed vulnerable:

1. **Register the resource** on the third-party service with the exact name.
2. **Upload a proof-of-concept page**:
   ```html
   <html><body><h1>Subdomain Takeover PoC by [YourName]</h1></body></html>
   ```
3. **Verify** by visiting the subdomain — it should serve your content.
4. **Take a screenshot** for the report.
5. **Report responsibly** — do NOT use for phishing.

---

## 6. MITIGATION (DEFENSE)

| Rule | Explanation |
|------|-------------|
| **1. Audit DNS regularly** | Remove unused CNAMEs. |
| **2. Monitor subdomains** | Use services like SecurityTrails, DNSTwist. |
| **3. Claim resources before deleting** | Don't delete a GitHub Pages repo without removing DNS. |
| **4. Use wildcard DNS carefully** | Wildcards can hide dangling records. |
| **5. Automate takeover checks** | Run nuclei/dnsreaper in CI/CD. |
| **6. Registrar lock** | Prevent unauthorized DNS changes. |
| **7. Enable DNSSEC** | Adds integrity to DNS responses. |

---

## 7. TIPS

1. Start with `subfinder + dnsx + nuclei takeovers` — fastest pipeline.
2. **Not all dangling CNAMEs are exploitable** — verify the error page.
3. Check both `http://` and `https://` for the error signature.
4. Some services require a specific region or account type.
5. **Never** use a takeover for phishing — it's illegal.
6. Report via **responsible disclosure** — many programs pay for this.
7. Look for subdomains of subdomains (e.g., `dev.blog.example.com`).
8. Historical DNS records (SecurityTrails, VirusTotal) reveal old CNAMEs.
9. Certificates (crt.sh) can reveal subdomains missed by wordlists.
10. Takeover bugs are common in **acquisitions** — old assets don't get cleaned up.

---

## FAST CHEAT SHEET

```
subfinder -d target.com -o subs.txt
amass enum -d target.com -o subs2.txt
assetfinder target.com > subs3.txt
cat subs*.txt | sort -u > all_subs.txt
dnsx -l all_subs.txt -cname -resp -o cnames.txt
for sub in $(cat all_subs.txt); do echo "=== $sub ==="; dig +short CNAME $sub; done
cat all_subs.txt | while read sub; do cname=$(dig +short CNAME $sub); if [ -n "$cname" ]; then echo "$sub -> $cname"; fi; done
for sub in $(cat cnames.txt); do echo "=== $sub ==="; curl -sI "http://$sub" | head -5; done
go install github.com/haccer/subjack@latest
subjack -w all_subs.txt -t 100 -timeout 30 -o results.txt -ssl
go install github.com/Ice3man543/SubOver@latest
SubOver -l all_subs.txt
nuclei -l all_subs.txt -t takeovers/ -o takeovers.txt
pip install dnsreaper
dnsreaper --filename all_subs.txt
```

---
