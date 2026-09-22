# LINUX NETWORK RECON & SNIFFING CHEAT SHEET

## 1. WHAT IS THIS?

Commands for **network discovery, monitoring, and packet capture** from a Linux (or Windows) host. This is what you run before you attack:

- **Local network state** (interfaces, routes, connections)
- **Neighbor discovery** (ARP, IPv6)
- **WiFi management** (nmcli, nmtui, AP mode)
- **Packet capture** (tcpdump, tshark)
- **DNS / HTTPS inspection** (SNI extraction)

**Key fact:** Sniffing is silent and passive. Recon before attack.

---

## 2. LOCAL NETWORK STATE (LINUX)

```bash
ip a                                    # interfaces + IPs
ip r                                    # routing table
ifconfig                                # legacy interfaces
route -n                                # routing table
htop                                    # live system resources
lsof -i -P -n                           # open connections + processes
ss -tuln                                # listening TCP/UDP sockets
ss -tlnp                                # listening TCP + process
ss -tunp state established              # established connections
ss -tnp state established | grep -v 127.0.0.1
netstat -tunap                          # all sockets + processes
netstat -a                              # all connections
netstat -aon > ports.txt                # dump to file
```

## 3. LOCAL NETWORK STATE (WINDOWS)

```cmd
ipconfig /all
ipconfig /displaydns
route print
arp -a
arp -n
netstat -a
netstat -aon > ports.txt
netstat -ano -b
netstat -onb
netstat -nao | findstr ESTABLISHED
tracert www.example.com
nslookup www.example.com
ping www.example.com
curl http://ip-api.com/json/<IP>
curl -s https://api.ipify.org
```

### PowerShell: live TCP connections with process details

```powershell
Get-NetTCPConnection -State Established |
    ForEach-Object {
        $procId = $_.OwningProcess
        $proc = Get-WmiObject Win32_Process -Filter "ProcessId=$procId"
        [PSCustomObject]@{
            LocalAddress  = $_.LocalAddress
            LocalPort     = $_.LocalPort
            RemoteAddress = $_.RemoteAddress
            RemotePort    = $_.RemotePort
            ProcessId     = $procId
            ProcessName   = $proc.Name
            Path          = $proc.ExecutablePath
        }
    } | Format-Table -AutoSize
```

---

## 4. IPv6 NEIGHBOR SCAN

```bash
ip -6 neigh | awk '{print $1}' | grep ':' | xargs -I{} sudo nmap -6 -T5 {}
ip -6 neigh | awk '/REACHABLE/ {print $1}' | xargs -I{} sudo nmap -6 -T5 {}
ip -6 neigh | awk '!/FAILED/ {print $1}' | xargs -I{} sudo nmap -6 -T4 --top-ports 100 {}
```

---

## 5. WIFI MANAGEMENT (NMCLI / NMTUI)

```bash
nmtui                                              # Text UI
nmcli dev wifi list                                # list networks
nmcli dev wifi connect "SSID" password "PASS"      # connect
nmcli device set wlan0 managed yes                 # enable management
nmcli connection show                              # list connections
nmcli connection up MiAP                           # bring up saved connection

# Create hotspot / AP
nmcli device wifi hotspot ifname wlan0 ssid MyAP password chocolate
sudo sysctl -w net.ipv4.ip_forward=1               # enable forwarding
```

---

## 6. TCPDUMP — PACKET CAPTURE

### 6.1 Interfaces & Basic Capture

```bash
tcpdump -D                                    # list interfaces
sudo tcpdump -i wlan0                         # capture on interface
sudo tcpdump -i wlan0 -w capture.pcap         # write to pcap
tcpdump -r capture.pcap                       # read pcap
sudo tcpdump -i any                           # all interfaces
```

### 6.2 Filters

```bash
tcpdump -i wlan0 src 192.168.1.10
tcpdump -i wlan0 dst 192.168.1.10
tcpdump -i wlan0 port 80
tcpdump -i wlan0 src port 443
tcpdump -i wlan0 icmp
tcpdump -i wlan0 arp
tcpdump -i wlan0 net 192.168.1.0/24
tcpdump -i wlan0 host 185.199.108.153
```

### 6.3 HTTPS / DNS

```bash
sudo tcpdump -i wlan0 port 53 -nn -v
sudo tcpdump -i wlan0 -nn -s 0 -v port 443 | grep "Server Name"
sudo tcpdump -i wlan0 port 80 -A
```

### 6.4 Attack Detection

```bash
# Detect SYN scans
tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'

# Common ports traffic
tcpdump -i wlan0 'dst port 22 or dst port 23 or dst port 80 or dst port 443'

# Only packets with data (no empty ACKs)
tcpdump -i wlan0 'tcp[((tcp[12] & 0xf0) >> 2):4] != 0'
```

### 6.5 Output Options

```bash
tcpdump -A -i wlan0              # ASCII output
tcpdump -i wlan0 -c 20           # capture 20 packets then stop
tcpdump -X -i wlan0              # hex + ASCII
```

### 6.6 Post-Analysis

```bash
wireshark capture.pcap
tshark -r capture.pcap
```

---

## 7. TSHARK — DNS / HTTPS SNI SNIFFING

```bash
# DNS queries (domains visited)
sudo tshark -i any -Y "dns.qry.name" -T fields -e dns.qry.name

# HTTPS SNI (TLS 1.2 / 1.3 - Server Name Indication)
sudo tshark -i any -Y "ssl.handshake.extensions_server_name" -T fields -e ssl.handshake.extensions_server_name
sudo tshark -i any -Y "tls.handshake.extensions_server_name" -T fields -e tls.handshake.extensions_server_name

# Save all HTTP hosts seen
sudo tshark -i any -Y "http.host" -T fields -e http.host
```

---

## 8. HOST FILE / DNS OVERRIDE

```bash
sudo nano /etc/hosts
```

```ini
127.0.1.1       myhostname
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```

```bash
sudo systemctl stop apache2
sudo systemctl stop nginx
getcap -r / 2>/dev/null
getcap /path/to/binary
getfacl /path/to/binary
```

---

## 9. SECURE COPY / REMOTE FILES

```bash
scp -r user@host:/remote/path /local/path
scp .\N64\* user@192.168.0.230:/home/user/games/
echo "kali:1234" | sudo chpasswd
ssh-keygen -R 192.168.100.125
```
