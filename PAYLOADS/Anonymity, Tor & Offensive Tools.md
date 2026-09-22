# ANONYMITY, TOR & OFFENSIVE TOOLS CHEAT SHEET

## 1. WHAT IS THIS?

Tools for **anonymity, tunneling, phishing, and specific attacks**:

- **Tor** — anonymity network, hidden services (.onion)
- **Proxychains** — route any app through Tor or SOCKS
- **SET** — Social Engineering Toolkit (phishing)
- **DoS tools** — hping3
- **Camera tools** — Cameradar
- **Pip Python config**

---

## 2. TOR — INSTALL AND CONFIGURE

```bash
sudo apt install tor
sudo systemctl start tor
sudo systemctl enable tor
sudo systemctl status tor
```

### 2.1 SOCKS Proxy (Default)

Tor listens on `127.0.0.1:9050` by default.

```bash
curl --socks5 127.0.0.1:9050 https://check.torproject.org/
curl --socks5-hostname 127.0.0.1:9050 http://example.onion
```

### 2.2 Environment Variables (temporary shell-wide)

```bash
export HTTP_PROXY=socks5://127.0.0.1:9050
export HTTPS_PROXY=socks5://127.0.0.1:9050
export ALL_PROXY=socks5://127.0.0.1:9050
```

---

## 3. TOR HIDDEN SERVICES (.ONION)

### 3.1 Configure /etc/tor/torrc

```ini
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080
```

### 3.2 Restart and Get .onion Address

```bash
sudo systemctl restart tor
python3 -m http.server 8080
sudo cat /var/lib/tor/hidden_service/hostname
```

---

## 4. PROXYCHAINS

### 4.1 Install

```bash
sudo apt install proxychains4
```

### 4.2 Configure /etc/proxychains4.conf

```ini
strict_chain
proxy_dns
tcp_read_time_out 15000
tcp_connect_time_out 8000

[ProxyList]
socks5 127.0.0.1 9050
```

### 4.3 Usage

```bash
proxychains firefox
proxychains curl http://example.onion
proxychains ssh user@host
proxychains nmap -sT -Pn target.onion
proxychains git clone http://example.onion/repo.git
```

---

## 5. TOR TUNNEL SCRIPT (AUTOMATED)

```bash
#!/bin/bash
# tunnel-tor.sh

RED='\033[0;31m'; GREEN='\033[0;32m'; BLUE='\033[0;34m'; NC='\033[0m'

echo -e "${BLUE}=== Configuring Tor SOCKS tunnel ===${NC}"

check_tor_installation() {
    if ! command -v tor &> /dev/null; then
        echo -e "${RED}Tor not installed. Installing...${NC}"
        sudo apt update && sudo apt install tor -y
    fi
}

start_tor_service() {
    sudo systemctl start tor
    sudo systemctl enable tor
}

setup_socks_tunnel() {
    if netstat -tuln | grep -q ':9050'; then
        echo -e "${GREEN}SOCKS proxy active on 127.0.0.1:9050${NC}"
    else
        echo -e "${RED}Tor not listening on 9050${NC}"
        exit 1
    fi
}

test_tor_connection() {
    if curl --socks5 127.0.0.1:9050 --connect-timeout 10 -s https://check.torproject.org/ | grep -q "Congratulations"; then
        echo -e "${GREEN}Tor connection OK${NC}"
    else
        echo -e "${RED}Tor connection failed${NC}"
    fi
}

show_usage() {
    echo -e "\n${GREEN}Usage examples:${NC}"
    echo -e "  curl:  curl --socks5 127.0.0.1:9050 http://example.onion"
    echo -e "  wget:  wget -e use_proxy=yes -e http_proxy=127.0.0.1:9050 URL"
    echo -e "  ssh:   ssh -o ProxyCommand='nc -x 127.0.0.1:9050 %h %p' user@host"
    echo -e "  env:   export ALL_PROXY=socks5://127.0.0.1:9050"
}

check_tor_installation
start_tor_service
setup_socks_tunnel
test_tor_connection
show_usage
```

---

## 6. TOR STATUS SCRIPT

```bash
#!/bin/bash
# tor-status.sh

echo "=== Tor service status ==="
sudo systemctl status tor --no-pager -l

echo -e "\n=== SOCKS port ==="
netstat -tuln | grep 9050

echo -e "\n=== Anonymity test ==="
IP_REAL=$(curl -s https://api.ipify.org)
IP_TOR=$(curl --socks5 127.0.0.1:9050 -s https://api.ipify.org)

echo "Real IP: $IP_REAL"
echo "Tor IP:  $IP_TOR"

if [ "$IP_REAL" != "$IP_TOR" ]; then
    echo "Anonymity active - different IPs"
else
    echo "Error: same IP"
fi
```

---

## 7. DOS ATTACKS (AUTHORIZED ONLY)

> ⚠️ **Warning:** DoS attacks can take down services. Use only on authorized targets, isolated labs, or your own infrastructure.

```bash
sudo apt install hping3

# ICMP flood with spoofed sources
sudo hping3 --icmp --rand-source --flood -d 1400 IP

# SYN flood on port 80
sudo hping3 -S --flood -p 80 [TARGET_IP]

# UDP flood on port 53
sudo hping3 --flood -2 -p 53 [TARGET_IP]

# ICMP flood on specific interface
sudo hping3 -I eth0 -1 --flood -V [TARGET_IP]
```

---

## 8. CAMERADAR — IP CAMERA SCANNER

```bash
sudo apt install docker.io
sudo systemctl start docker
git clone https://github.com/Ullaakut/cameradar
sudo docker run ullaakut/cameradar -t IP -p PORT
```

---

## 9. SET — SOCIAL ENGINEERING TOOLKIT

📁 **Carpeta destino sugerida:** `Pentesting/WEB/`

```bash
sudo setoolkit
```

**Menu:**
```
1) Social-Engineering Attacks
2) Penetration Testing (Fast-Track)
3) Third Party Modules
4) Update the Social-Engineer Toolkit
5) Exit
```

**Common attacks:**
```
1) Social-Engineering Attacks > 2) Website Attack Vectors
   - Clonar página web (Credential Harvester)
   - Java Applet / Metasploit browser exploit

1) Social-Engineering Attacks > 3) Infectious Media Generator
   - Malicious USB
```

**Config file:**
```
/etc/setoolkit/set.config
```

---

## 10. PIP PYTHON — ROM PER SYSTEM PACKAGES

### Option 1: Global config
```bash
python3 -m pip config set global.break-system-packages true
```

### Option 2: /etc/pip.conf
```bash
sudo nano /etc/pip.conf
```

```ini
[global]
break-system-packages = true
```

### Option 3: Virtual environment (SAFE)
```bash
sudo apt install python3-venv
python3 -m venv venv
source venv/bin/activate
deactivate
```

### Compile Python to EXE
```bash
pyinstaller --onefile my_script.py
```

---

## 11. TIPS

1. Always test Tor with `check.torproject.org` before trusting it.
2. Use `proxychains` for apps that don't support SOCKS natively.
3. Never run SET against unauthorized targets — it is a phishing tool.
4. DoS attacks are illegal outside authorized engagements.
5. Cameradar requires Docker; make sure Docker is running.
6. Prefer virtual environments over `break-system-packages` for stability.
7. Save `tor-status.sh` and run before long anonymous sessions.
8. Hidden services take ~30 seconds to publish after restarting Tor.
