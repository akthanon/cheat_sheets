# .NET MALWARE ANALYSIS & REVERSING COMPLETE CHEAT SHEET

## 1. WHAT IS .NET MALWARE ANALYSIS?

**.NET malware analysis** is the process of reverse-engineering, deobfuscating, and understanding compiled .NET assemblies (EXEs, DLLs) to identify behavior, extract IOCs, and find vulnerabilities. Common scenarios:

- **Analyzing suspicious .NET executables** (bots, stealers, RATs)
- **Deobfuscating protected binaries** (Confuser, Eazfuscator, .NET Reactor)
- **Extracting hardcoded credentials** (API keys, tokens, webhooks)
- **Auditing client-side licensing systems** (Supabase, Firebase, custom APIs)
- **Finding design vulnerabilities** (exposed backend, missing RLS, weak crypto)

**Key fact:** .NET assemblies contain IL (Intermediate Language) that is relatively easy to decompile. Obfuscation raises the bar but is rarely a real barrier.

---

## 2. INITIAL TRIAGE

### 2.1 File Identification

```bash
file suspicious.exe
stat suspicious.exe
```

### 2.2 Hash Calculation

```bash
md5sum suspicious.exe
sha1sum suspicious.exe
sha256sum suspicious.exe
sha512sum suspicious.exe
```

### 2.3 Detect .NET, Packer, Framework

```bash
# .NET check
strings -n 6 suspicious.exe | grep -i "mscorlib"
strings -n 6 suspicious.exe | grep -i "System.Runtime"

# Protector detection
strings suspicious.exe | grep -iE "Confuser|Eazfuscator|Reactor|SmartAssembly|Dotfuscator|Costura"

# Framework version
strings suspicious.exe | grep -iE "v[0-9]+\.[0-9]+\.[0-9]+"
```

### 2.4 VirusTotal / Online Analysis

```
- https://virustotal.com           → Detections, tags, behavior
- https://hybrid-analysis.com      → Sandbox report
- https://app.any.run              → Interactive sandbox
- https://www.joesandbox.com/      → Automated analysis
- https://unpac.me                 → Multi-packer unpacker
```

**Look for tags:** `obfuscated`, `detect-debug-environment`, `long-sleeps`, `anti-VM`.

---

## 3. ENVIRONMENT SETUP

### 3.1 Install Tools (Arch / Debian / Ubuntu)

```bash
# Arch
sudo pacman -S file binwalk ent pev hexdump xxd mono dotnet-sdk

# Debian / Ubuntu
sudo apt install file binwalk ent pev-common mono-complete dotnet-sdk-8.0
```

### 3.2 .NET Tools

```bash
# ILSpy CLI (decompiler)
dotnet tool install -g ilspycmd

# dnSpy (GUI decompiler / debugger) — download from GitHub
# https://github.com/dnSpy/dnSpy/releases

# de4dot (deobfuscator)
git clone https://github.com/de4dot/de4dot.git
cd de4dot && dotnet build -c Release

# Alternative via AUR
yay -S de4dot
```

### 3.3 Reversing Frameworks

| Tool | Purpose |
|------|---------|
| **dnSpy** | GUI decompiler + debugger (best for .NET) |
| **ILSpy** | Open-source decompiler (CLI + GUI) |
| **dotPeek** | JetBrains .NET decompiler |
| **JustDecompile** | Telerik decompiler |
| **Reflector** | Red Gate decompiler |
| **de4dot** | Deobfuscator for Confuser, Reactor, etc. |
| **ConfuserEx** | Protector (for testing) |
| **Detect It Easy (DIE)** | Packer / protector identifier |
| **PEStudio** | Static analysis (Windows) |
| **CFF Explorer** | PE header viewer |
| **HxD / 010 Editor** | Hex editors |

---

## 4. STATIC ANALYSIS

### 4.1 Extract Strings

```bash
strings -n 6 suspicious.exe > all_strings.txt
strings -n 10 suspicious.exe > long_strings.txt
strings -a -n 4 suspicious.exe > all_ascii.txt

# Unicode strings (common in .NET)
strings -e l suspicious.exe > unicode_strings.txt
```

### 4.2 Filter Suspicious Strings

```bash
# URLs
grep -E 'https?://[a-zA-Z0-9./?=_%:-]+' all_strings.txt

# IPs
grep -E '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' all_strings.txt

# Domains
grep -E '\b[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b' all_strings.txt | sort -u

# Credentials / secrets keywords
grep -iE '(password|passwd|pwd|login|auth|token|secret|key|apikey|api_key|webhook|bearer)' all_strings.txt

# JWT tokens
grep -E 'eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' all_strings.txt

# Base64 blobs
grep -E '^[A-Za-z0-9+/]{20,}={0,2}$' all_strings.txt

# Hex blobs (hashes, keys)
grep -E '[0-9a-fA-F]{32,}' all_strings.txt

# File names / config
grep -iE '\.(dll|exe|dat|ini|log|config|key|pem|cert|lic|json|xml)$' all_strings.txt
```

### 4.3 Entropy Analysis

```bash
ent suspicious.exe
ent -t suspicious.exe
head -c 1024 suspicious.exe | ent -t
tail -c 1024 suspicious.exe | ent -t
```

**Interpretation:**
- **< 6.0** → likely plaintext / code
- **6.0 – 7.0** → compressed or mixed
- **> 7.0** → encrypted, packed, or shellcode

### 4.4 Packer Detection

```bash
binwalk suspicious.exe
diec suspicious.exe                    # Detect It Easy CLI
strings suspicious.exe | grep -iE "Confuser|Costura|Eazfuscator|Reactor"
```

---

## 5. DEOBFUSCATION

### 5.1 Identify Protector

```bash
# Look for signatures
strings suspicious.exe | grep -iE "Confuser|Eazfuscator|Reactor|SmartAssembly|Dotfuscator|.netshrink|Babel"
```

### 5.2 Deobfuscate with de4dot

```bash
# Basic deobfuscation
de4dot suspicious.exe -o cleaned.exe

# With specific protector
de4dot -p cr suspicious.exe -o cleaned.exe    # Confuser
de4dot -p dr suspicious.exe -o cleaned.exe    # .NET Reactor
de4dot -p sa suspicious.exe -o cleaned.exe    # SmartAssembly
de4dot -p ef suspicious.exe -o cleaned.exe    # Eazfuscator

# Preserve metadata, rename all
de4dot --preserve-tokens --rename-all cleaned.exe
```

### 5.3 Manual Unpacking (when de4dot fails)

1. Open in **dnSpy** or **ILSpy**.
2. Look for a **decryptor method** (usually `Module.cctor` or `Assembly.Load`).
3. Set a breakpoint after decryption.
4. Dump the decrypted assembly from memory using **MegaDumper** or **ExtremeDumper**.

---

## 6. DECOMPILATION

### 6.1 With ILSpy (CLI)

```bash
# Decompile to project
ilspycmd suspicious.exe -p -o ./decompiled_code/

# Decompile to single file
ilspycmd suspicious.exe -o ./decompiled_code/
```

### 6.2 With dnSpy (GUI)

1. Open `suspicious.exe` in dnSpy.
2. Navigate the assembly tree.
3. Right-click → **Edit Method** (view IL or C#).
4. Set breakpoints and debug live.

### 6.3 Structure You Should Expect

```
decompiled_code/
├── Form1.cs
├── Program.cs
├── Settings.cs
├── AssemblyInfo.cs
├── Class1.cs
├── Resources.resx
└── Properties/
```

**Focus on:**
- `Main()` entry point
- `AssemblyInfo.cs` (metadata, version, company)
- Classes that handle network, crypto, file I/O
- Constants (hardcoded keys, URLs)

---

## 7. EXTRACTING HARDCODED SECRETS

### 7.1 Common Locations

| Pattern | Example |
|---------|---------|
| `private const string` | Hardcoded API keys |
| `WebClient.DownloadString` | URLs |
| `HttpClient.PostAsync` | API endpoints |
| `new Webhook` / `Discord` | Discord webhooks |
| `Convert.FromBase64String` | Encoded secrets |
| `Aes.Create()` / `RSA.Create()` | Crypto keys |
| `Registry.GetValue` | Persistence / config |

### 7.2 Search Techniques

```bash
# Search in decompiled code
grep -rE "api_key|apikey|bearer|token|secret|webhook" ./decompiled_code/

# Search for JWT
grep -rE "eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+" ./decompiled_code/

# Search for URLs
grep -rE "https?://" ./decompiled_code/

# Search for connection strings
grep -riE "connectionstring|Data Source|Server=|User Id=" ./decompiled_code/
```

### 7.3 Decode and Validate

```bash
# Decode JWT payload
echo "JWT_PAYLOAD" | cut -d. -f2 | base64 -d 2>/dev/null | jq .

# Decode Base64
echo "BASE64_STRING" | base64 -d

# Decode hex
echo "DEADBEEF" | xxd -r -p
```

---

## 8. ANALYZING CLIENT-SIDE BACKENDS

### 8.1 Common Backends (BaaS)

| Backend | URL Pattern |
|---------|-------------|
| **Supabase** | `https://<project>.supabase.co` |
| **Firebase** | `https://<project>.firebaseio.com` |
| **Appwrite** | `https://<project>.appwrite.io` |
| **PocketBase** | `https://<host>/api/` |
| **Discord Webhook** | `https://discord.com/api/webhooks/<id>/<token>` |

### 8.2 Supabase Testing

```bash
# Set variables
API_KEY="<ANON_KEY>"
SUPABASE_URL="<SUPABASE_URL>"

# List available tables (via REST)
curl -H "apikey: $API_KEY" -H "Authorization: Bearer $API_KEY" \
     "$SUPABASE_URL/rest/v1/"

# SELECT
curl -H "apikey: $API_KEY" -H "Authorization: Bearer $API_KEY" \
     "$SUPABASE_URL/rest/v1/<table>?select=*"

# INSERT (test permission)
curl -X POST -H "apikey: $API_KEY" -H "Content-Type: application/json" \
     -d '{"test":"value"}' \
     "$SUPABASE_URL/rest/v1/<table>"

# UPDATE (test permission)
curl -X PATCH -H "apikey: $API_KEY" -H "Content-Type: application/json" \
     -d '{"field":"value"}' \
     "$SUPABASE_URL/rest/v1/<table>?id=eq.<id>"

# DELETE (test permission)
curl -X DELETE -H "apikey: $API_KEY" \
     "$SUPABASE_URL/rest/v1/<table>?id=eq.<id>"
```

**Look for:**
- Missing **RLS (Row Level Security)** → `UPDATE` / `DELETE` allowed with anon key
- Exposed **service_role** key (worse than anon)
- Public **storage buckets**
- Public **auth endpoints**

### 8.3 Firebase Testing

```bash
# Check if database is public
curl "https://<project>.firebaseio.com/.json"

# Read specific node
curl "https://<project>.firebaseio.com/users.json"

# Test write
curl -X PUT -d '{"test":"value"}' "https://<project>.firebaseio.com/test.json"
```

### 8.4 Discord Webhook Testing

```bash
# Validate webhook (does it exist?)
curl "https://discord.com/api/webhooks/<ID>/<TOKEN>"

# Send test message (authorized testing only)
curl -X POST -H "Content-Type: application/json" \
     -d '{"content":"Test message"}' \
     "https://discord.com/api/webhooks/<ID>/<TOKEN>"

# Delete webhook (if it belongs to you)
curl -X DELETE "https://discord.com/api/webhooks/<ID>/<TOKEN>"
```

---

## 9. BEHAVIORAL ANALYSIS

### 9.1 Key .NET APIs to Watch

| API | Purpose |
|-----|---------|
| `System.Net.WebClient` | HTTP requests |
| `System.Net.Http.HttpClient` | HTTP requests (modern) |
| `System.Net.Sockets.TcpClient` | Raw TCP |
| `System.Diagnostics.Process` | Process execution |
| `Microsoft.Win32.Registry` | Persistence |
| `System.IO.File` | File operations |
| `System.Drawing.Graphics.CopyFromScreen` | Screenshots |
| `keybd_event` / `mouse_event` (P/Invoke) | Input simulation |
| `System.Management` | WMI queries |
| `System.Security.Cryptography` | Crypto operations |

### 9.2 Anti-Analysis Techniques

```bash
# Look for
strings suspicious.exe | grep -iE "IsDebuggerPresent|CheckRemoteDebuggerPresent|VirtualBox|VMware|Sandboxie|Sleep|TickCount|Stopwatch"

# Common sandbox evasion
strings suspicious.exe | grep -iE "GetSystemMetrics|GetTickCount|Sleep\([0-9]+\)|Environment.TickCount"
```

### 9.3 MITRE ATT&CK Mapping

| Tactic | IDs |
|--------|-----|
| **Execution** | T1047, T1059, T1106, T1129, T1574 |
| **Privilege Escalation** | T1055 |
| **Defense Evasion** | T1027, T1055, T1070, T1140, T1497, T1562, T1574, T1620 |
| **Credential Access** | T1056 |
| **Discovery** | T1010, T1033, T1082, T1083, T1087, T1497 |
| **Collection** | T1056, T1113 |
| **Command & Control** | T1071 |

---

## 10. DYNAMIC ANALYSIS

### 10.1 dnSpy Debugging

1. Open target in dnSpy.
2. Set breakpoints on suspicious methods (network, crypto, file I/O).
3. **Debug → Start** (or attach to running process).
4. Step through, inspect locals, watch for decrypted strings.

### 10.2 Frida for .NET

```bash
# Hook methods at runtime
frida -U -f suspicious.exe -l hook.js --no-pause

# Example hook.js
Interceptor.attach(Module.findExportByName(null, "AmsiScanBuffer"), {
  onEnter: function (args) {
    console.log("AMSI called");
  }
});
```

### 10.3 Sandbox Execution

```bash
# Windows Sandbox / VMware / VirtualBox (isolated)
# Tools: Process Monitor, Process Hacker, Wireshark, FakeNet-NG

# Linux: Wine + strace + ltrace
WINEDEBUG=+all wine suspicious.exe 2> wine.log
strace -f -o strace.log wine suspicious.exe
```

### 10.4 Network Capture

```bash
sudo tcpdump -i any -w capture.pcap
# Or Wireshark GUI
wireshark
```

**Look for:**
- HTTP/HTTPS requests (use Fiddler / mitmproxy to decrypt)
- DNS queries
- WebSocket connections
- Persistence callbacks

---

## 11. AUTOMATION SCRIPT — STATIC ANALYSIS

```bash
#!/bin/bash
# static_analysis.sh — Generic .NET binary analyzer

TARGET="$1"
OUT="analysis_$(date +%Y%m%d_%H%M%S)"

if [ -z "$TARGET" ] || [ ! -f "$TARGET" ]; then
    echo "Usage: $0 <target.exe>"
    exit 1
fi

mkdir -p "$OUT"/{strings,hex,resources}

echo "[*] Hashing..."
{
    echo "MD5:    $(md5sum "$TARGET" | cut -d' ' -f1)"
    echo "SHA1:   $(sha1sum "$TARGET" | cut -d' ' -f1)"
    echo "SHA256: $(sha256sum "$TARGET" | cut -d' ' -f1)"
    echo "Size:   $(stat -c%s "$TARGET") bytes"
    echo "Type:   $(file -b "$TARGET")"
} | tee "$OUT/hashes.txt"

echo "[*] Extracting strings..."
strings -n 6 "$TARGET" > "$OUT/strings/all_strings.txt"
strings -n 10 "$TARGET" > "$OUT/strings/long_strings.txt"
strings -e l "$TARGET" > "$OUT/strings/unicode_strings.txt"

echo "[*] Filtering interesting strings..."
{
    echo "=== URLs ==="
    grep -E 'https?://[a-zA-Z0-9./?=_%:-]+' "$OUT/strings/all_strings.txt"
    echo "=== IPs ==="
    grep -E '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' "$OUT/strings/all_strings.txt"
    echo "=== JWT ==="
    grep -E 'eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' "$OUT/strings/all_strings.txt"
    echo "=== Keywords ==="
    grep -iE '(password|token|secret|apikey|api_key|webhook|bearer|auth)' "$OUT/strings/all_strings.txt"
} > "$OUT/interesting.txt"

echo "[*] Entropy..."
ent "$TARGET" > "$OUT/entropy.txt" 2>/dev/null

echo "[*] Packer detection..."
{
    strings "$TARGET" | grep -iE "Confuser|Costura|Eazfuscator|Reactor|Dotfuscator"
} > "$OUT/packer.txt"

echo "[*] First bytes (hexdump)..."
hexdump -C "$TARGET" | head -20 > "$OUT/hex/head.txt"

echo "[+] Analysis complete. Results in $OUT/"
```

---

## 12. EXTRACTING LICENSING VULNERABILITIES

### 12.1 Common Weaknesses

| Weakness | Impact |
|----------|--------|
| API key hardcoded in client | Full DB access |
| Missing RLS | UPDATE / DELETE by anyone |
| Client talks directly to backend | No server-side validation |
| Weak obfuscation (Confuser 1.x) | Trivial to deobfuscate |
| No rate limiting | Brute force |
| No logging | No audit trail |
| Master keys in code | Free access to software |
| HWID stored client-side only | Trivial to spoof |

### 12.2 Testing Checklist

```
[ ] Extract all API keys, tokens, URLs
[ ] Identify backend (Supabase, Firebase, custom)
[ ] Test SELECT with anon key
[ ] Test INSERT with anon key
[ ] Test UPDATE with anon key
[ ] Test DELETE with anon key
[ ] Test storage buckets for public access
[ ] Test auth endpoints
[ ] Check for missing RLS
[ ] Check for exposed service_role key
[ ] Check for master keys / bypass logic
[ ] Test HWID spoofing
[ ] Test time-based bypass (change system date)
[ ] Test offline mode
```

---

## 13. REPORTING FINDINGS

### 13.1 Report Structure

```
1. Executive Summary
2. Target Information (hashes, type, framework)
3. Behavioral Analysis (what it does)
4. IOCs (URLs, IPs, domains, hashes)
5. Vulnerabilities (severity, impact, PoC)
6. Recommendations (fixes)
7. Evidence (screenshots, logs, dumps)
```

### 13.2 Severity Ratings

| Severity | Criteria |
|----------|----------|
| **CRITICAL** | Full DB access, RCE, credential leak |
| **HIGH** | Data modification, auth bypass |
| **MEDIUM** | Info disclosure, weak crypto |
| **LOW** | Misconfigurations, best-practice violations |

---

## 14. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Never hardcode secrets in clients** | Use server-side proxies. |
| **2. Use service_role only on backend** | `anon` keys are public by definition. |
| **3. Enable RLS on all tables** | Supabase / Postgres rows require explicit policies. |
| **4. Rotate keys regularly** | Assume compromise and rotate. |
| **5. Avoid client-side license checks** | Validate on server. |
| **6. Use strong obfuscation wisely** | It delays, not prevents, reversing. |
| **7. Add rate limiting** | Prevent brute force. |
| **8. Log all sensitive operations** | Enable audit trails. |
| **9. Implement HWID binding server-side** | Never trust client. |
| **10. Monitor webhook abuse** | Detect leaked webhooks. |

---

## 15. TIPS FOR TESTING

1. Always start with `file` and `strings` — most secrets leak there.
2. Use `de4dot` before anything else if a protector is detected.
3. Prefer **dnSpy** over ILSpy for interactive reversing.
4. Search decompiled code for `const string`, `WebClient`, `HttpClient`.
5. Decode every JWT you find — payload reveals backend structure.
6. Test Supabase / Firebase permissions with `curl`.
7. Look for `MASTER`, `BYPASS`, `ADMIN`, `LICENSE` keywords in strings.
8. Sandbox the binary and watch network traffic with Wireshark.
9. Never upload proprietary binaries to online sandboxes without permission.
10. Document everything — reversing is only useful if shared responsibly.

---

## 16. ETHICS AND SAFETY

- Analyze **only** binaries you own or have permission to test.
- Do **not** interact with production backends without authorization.
- Report vulnerabilities through **responsible disclosure**.
- Do **not** publish live API keys, tokens, or user data.
- Use isolated VMs for dynamic analysis.
- Follow local laws regarding reverse engineering.

---

## FAST COMMANDS

```
file suspicious.exe
stat suspicious.exe
md5sum suspicious.exe
sha1sum suspicious.exe
sha256sum suspicious.exe
sha512sum suspicious.exe
strings -n 6 suspicious.exe | grep -i "mscorlib"
strings -n 6 suspicious.exe | grep -i "System.Runtime"
strings suspicious.exe | grep -iE "Confuser|Eazfuscator|Reactor|SmartAssembly|Dotfuscator|Costura"
strings suspicious.exe | grep -iE "v[0-9]+\.[0-9]+\.[0-9]+"
sudo pacman -S file binwalk ent pev hexdump xxd mono dotnet-sdk
sudo apt install file binwalk ent pev-common mono-complete dotnet-sdk-8.0
dotnet tool install -g ilspycmd
git clone https://github.com/de4dot/de4dot.git
cd de4dot && dotnet build -c Release
yay -S de4dot
strings -n 6 suspicious.exe > all_strings.txt
strings -n 10 suspicious.exe > long_strings.txt
strings -a -n 4 suspicious.exe > all_ascii.txt
strings -e l suspicious.exe > unicode_strings.txt
grep -E 'https?://[a-zA-Z0-9./?=_%:-]+' all_strings.txt
grep -E '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' all_strings.txt
grep -E '\b[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b' all_strings.txt | sort -u
grep -iE '(password|passwd|pwd|login|auth|token|secret|key|apikey|api_key|webhook|bearer)' all_strings.txt
grep -E 'eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+' all_strings.txt
grep -E '^[A-Za-z0-9+/]{20,}={0,2}$' all_strings.txt
grep -E '[0-9a-fA-F]{32,}' all_strings.txt
grep -iE '\.(dll|exe|dat|ini|log|config|key|pem|cert|lic|json|xml)$' all_strings.txt
ent suspicious.exe
ent -t suspicious.exe
head -c 1024 suspicious.exe | ent -t
tail -c 1024 suspicious.exe | ent -t
binwalk suspicious.exe
diec suspicious.exe
de4dot suspicious.exe -o cleaned.exe
de4dot -p cr suspicious.exe -o cleaned.exe
de4dot -p dr suspicious.exe -o cleaned.exe
de4dot -p sa suspicious.exe -o cleaned.exe
de4dot -p ef suspicious.exe -o cleaned.exe
de4dot --preserve-tokens --rename-all cleaned.exe
ilspycmd suspicious.exe -p -o ./decompiled_code/
ilspycmd suspicious.exe -o ./decompiled_code/
grep -rE "api_key|apikey|bearer|token|secret|webhook" ./decompiled_code/
grep -rE "eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+" ./decompiled_code/
grep -rE "https?://" ./decompiled_code/
grep -riE "connectionstring|Data Source|Server=|User Id=" ./decompiled_code/
echo "JWT_PAYLOAD" | cut -d. -f2 | base64 -d 2>/dev/null | jq .
echo "BASE64_STRING" | base64 -d
echo "DEADBEEF" | xxd -r -p
API_KEY="<ANON_KEY>"
SUPABASE_URL="<SUPABASE_URL>"
curl -H "apikey: $API_KEY" -H "Authorization: Bearer $API_KEY" "$SUPABASE_URL/rest/v1/"
curl -H "apikey: $API_KEY" -H "Authorization: Bearer $API_KEY" "$SUPABASE_URL/rest/v1/<table>?select=*"
curl -X POST -H "apikey: $API_KEY" -H "Content-Type: application/json" -d '{"test":"value"}' "$SUPABASE_URL/rest/v1/<table>"
curl -X PATCH -H "apikey: $API_KEY" -H "Content-Type: application/json" -d '{"field":"value"}' "$SUPABASE_URL/rest/v1/<table>?id=eq.<id>"
curl -X DELETE -H "apikey: $API_KEY" "$SUPABASE_URL/rest/v1/<table>?id=eq.<id>"
curl "https://<project>.firebaseio.com/.json"
curl "https://<project>.firebaseio.com/users.json"
curl -X PUT -d '{"test":"value"}' "https://<project>.firebaseio.com/test.json"
curl "https://discord.com/api/webhooks/<ID>/<TOKEN>"
curl -X POST -H "Content-Type: application/json" -d '{"content":"Test message"}' "https://discord.com/api/webhooks/<ID>/<TOKEN>"
curl -X DELETE "https://discord.com/api/webhooks/<ID>/<TOKEN>"
strings suspicious.exe | grep -iE "IsDebuggerPresent|CheckRemoteDebuggerPresent|VirtualBox|VMware|Sandboxie|Sleep|TickCount|Stopwatch"
strings suspicious.exe | grep -iE "GetSystemMetrics|GetTickCount|Sleep\([0-9]+\)|Environment.TickCount"
frida -U -f suspicious.exe -l hook.js --no-pause
WINEDEBUG=+all wine suspicious.exe 2> wine.log
strace -f -o strace.log wine suspicious.exe
sudo tcpdump -i any -w capture.pcap
wireshark
```
