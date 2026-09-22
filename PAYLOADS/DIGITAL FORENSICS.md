# DIGITAL FORENSICS COMPLETE CHEAT SHEET

## 1. WHAT IS DIGITAL FORENSICS?

**Digital Forensics** is the process of preserving, acquiring, analyzing, and reporting digital evidence from systems, networks, and storage media. It is used in:

- **Incident Response (IR)** — understand what happened after a breach
- **Malware analysis** — post-infection forensics
- **Legal investigations** — evidence for court
- **CTF / challenges** — forensic puzzle solving
- **Compliance audits** — verify data integrity
- **Threat hunting** — proactive hunting for IOCs

**Key fact:** Forensics requires **chain of custody** — every step must be documented and hashed to preserve evidence integrity.

---

## 2. FORENSIC METHODOLOGY

```
1. Identification       → Locate evidence sources
2. Preservation         → Write blockers, hashing, imaging
3. Acquisition          → Disk / memory / network capture
4. Examination          → Recover and extract data
5. Analysis             → Correlate, timeline, attribute
6. Reporting            → Document findings, chain of custody
```

**Golden rules:**
- **Never** work on the original — always use a write blocker.
- **Hash everything** — MD5, SHA1, SHA256 before and after.
- **Document every command** — including timestamps.
- **Preserve metadata** — don't modify timestamps.
- **Use forensically sound tools** — validated, tested.

---

## 3. ACQUISITION

### 3.1 Disk Imaging

**dd (Linux):**
```bash
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M conv=noerror,sync

# With hashing
sudo dd if=/dev/sda bs=4M | tee /evidence/disk.img | sha256sum > disk.sha256
```

**dcfldd (better than dd):**
```bash
sudo dcfldd if=/dev/sda of=/evidence/disk.img bs=4M hash=sha256 hashwindow=1G hashlog=disk.log
```

**dc3dd (forensic-focused):**
```bash
sudo dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256 log=disk.log
```

**Guymager (GUI):**
```bash
sudo guymager
```

**FTK Imager (Windows, free):**
```
- File → Create Disk Image
- Choose source (physical drive, logical drive, image file)
- Set destination (E01 or RAW)
- Fragment size, compression, case info
- Verify image after creation
```

**EWF / E01 format:**
```bash
ewfacquire /dev/sda
ewfverify evidence.E01
ewfinfo evidence.E01
```

### 3.2 Memory Acquisition

**LiME (Linux Memory Extractor):**
```bash
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src
make
sudo insmod lime.ko "path=/evidence/memory.lime format=lime"
sudo insmod lime.ko "path=/evidence/memory.raw format=raw"
```

**AVML (Microsoft, cross-platform):**
```bash
avml /evidence/memory.lime
```

**WinPmem / DumpIt (Windows):**
```cmd
winpmem_mini_x64.exe memory.raw
DumpIt.exe /output memory.raw
```

**Volatility's `imagecopy` (via winpmem):**
```
vol -f memory.raw windows.info
```

### 3.3 Hashing

```bash
# Generate hash
sha256sum evidence.img > evidence.img.sha256
md5sum evidence.img > evidence.img.md5

# Verify hash
sha256sum -c evidence.img.sha256

# Combined
hashdeep -c sha256 -r /evidence > evidence_hashes.txt
md5deep -r /evidence
```

### 3.4 Write Blockers

**Hardware:** Tableau, WiebeTech, CRU.

**Software (Linux):**
```bash
# Set disk read-only
blockdev --setro /dev/sda

# Check
blockdev --getro /dev/sda
```

---

## 4. DISK FORENSICS

### 4.1 Image Analysis

**Autopsy / Sleuth Kit:**
```bash
sudo apt install autopsy sleuthkit

# CLI with TSK
img_stat evidence.img
mmls evidence.img                            # partition layout
fsstat -o 2048 evidence.img                  # filesystem stats
fls -r -o 2048 evidence.img                  # list files recursively
fls -r -o 2048 -m / evidence.img > body.txt  # timeline (bodyfile)
icat -o 2048 evidence.img 1234 > recovered.txt
blkls -o 2048 evidence.img > unallocated.img
```

**Autopsy (GUI):**
```bash
autopsy
# Then access http://localhost:9999/autopsy (older version)
# Or the newer Autopsy 4.x desktop app
```

**FTK Imager:**
```
- Add Evidence Item → Image File
- Browse filesystem, view hex, recover deleted files
- Export files, generate hash reports
```

### 4.2 Partition & Filesystem

```bash
# Partition tables
mmls evidence.img
fdisk -l evidence.img
parted evidence.img unit s print

# Extract partition
dd if=evidence.img of=partition.img bs=512 skip=2048 count=... 
# or use ddrescue
ddrescue -i 1048576 evidence.img partition.img
```

### 4.3 File Recovery

```bash
# Scalpel (signature-based carving)
sudo apt install scalpel
scalpel evidence.img -o output/

# Foremost (file carving)
sudo apt install foremost
foremost -t all -i evidence.img -o output/
foremost -t jpg,pdf,doc -i evidence.img -o output/

# Photorec (TestDisk suite)
sudo apt install testdisk
photorec evidence.img

# Bulk Extractor (fast extraction)
sudo apt install bulk-extractor
bulk_extractor -o output/ evidence.img
```

### 4.4 Filesystem-Specific

**NTFS (Windows):**
```bash
# Analyze with TSK
fsstat -f ntfs evidence.img
fls -f ntfs -r -o 2048 evidence.img
icat -f ntfs -o 2048 evidence.img <inode> > file

# Extract MFT
ntfsinfo -m -o 2048 evidence.img
analyzeMFT.py -f MFT -o mft.csv
```

**Ext4 (Linux):**
```bash
fsstat -f ext4 evidence.img
fls -f ext4 -r evidence.img
```

**APFS (macOS):**
```bash
# apfs-fuse for mounting
sudo apt install apfs-fuse
sudo apfs-fuse evidence.img /mnt/apfs
```

### 4.5 Timeline Analysis

```bash
# Generate body file with TSK
fls -r -m / -o 2048 evidence.img > body.txt

# Convert to timeline
mactime -b body.txt -d > timeline.csv

# With date filter
mactime -b body.txt -d 2024-01-01..2024-12-31 > timeline.csv

# Log2timeline (Plaso) — the gold standard
sudo apt install plaso
log2timeline.py timeline.plaso evidence.img
psort.py -o l2tcsv timeline.plaso > timeline.csv
psort.py -o timeline.html timeline.plaso > timeline.html

# Filter
psort.py timeline.plaso "date > '2024-06-01'"
```

---

## 5. MEMORY FORENSICS

### 5.1 Volatility 2 (classic)

```bash
# Identify profile
volatility -f memory.raw imageinfo
volatility -f memory.raw --profile=Win10x64 kdbgscan

# Processes
volatility -f memory.raw --profile=Win10x64 pslist
volatility -f memory.raw --profile=Win10x64 psscan
volatility -f memory.raw --profile=Win10x64 pstree
volatility -f memory.raw --profile=Win10x64 cmdline
volatility -f memory.raw --profile=Win10x64 dlllist

# Network
volatility -f memory.raw --profile=Win10x64 netscan
volatility -f memory.raw --profile=Win10x64 connections

# Files
volatility -f memory.raw --profile=Win10x64 filescan
volatility -f memory.raw --profile=Win10x64 dumpfiles -Q 0x...
volatility -f memory.raw --profile=Win10x64 dumpfiles -D output/ -n

# Registry
volatility -f memory.raw --profile=Win10x64 hivelist
volatility -f memory.raw --profile=Win10x64 printkey -K "Software\Microsoft\Windows\CurrentVersion\Run"

# Malware detection
volatility -f memory.raw --profile=Win10x64 malfind
volatility -f memory.raw --profile=Win10x64 hollowfind
volatility -f memory.raw --profile=Win10x64 apihooks
volatility -f memory.raw --profile=Win10x64 ssdt

# Credentials
volatility -f memory.raw --profile=Win10x64 hashdump
volatility -f memory.raw --profile=Win10x64 lsadump

# Timeline
volatility -f memory.raw --profile=Win10x64 timeliner
```

### 5.2 Volatility 3 (modern)

```bash
pip install volatility3

# Info
vol -f memory.dump windows.info
vol -f memory.dump linux.info
vol -f memory.dump mac.info

# Processes
vol -f memory.dump windows.pslist
vol -f memory.dump windows.psscan
vol -f memory.dump windows.pstree
vol -f memory.dump windows.cmdline
vol -f memory.dump windows.dlllist
vol -f memory.dump windows.handles
vol -f memory.dump windows.getsids

# Network
vol -f memory.dump windows.netscan
vol -f memory.dump windows.netstat

# Files
vol -f memory.dump windows.filescan
vol -f memory.dump windows.dumpfiles --virtaddr 0x...
vol -f memory.dump windows.dumpfiles --pid 1234

# Registry
vol -f memory.dump windows.registry.hivelist
vol -f memory.dump windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
vol -f memory.dump windows.registry.userassist

# Malware
vol -f memory.dump windows.malfind
vol -f memory.dump windows.hollowfind
vol -f memory.dump windows.vadyarascan --yara-rules rule.yar
vol -f memory.dump windows.vadinfo --pid 1234

# Credentials
vol -f memory.dump windows.hashdump
vol -f memory.dump windows.lsadump
vol -f memory.dump windows.cachedump

# Timeline
vol -f memory.dump timeliner

# Linux
vol -f memory.dump linux.pslist
vol -f memory.dump linux.bash
vol -f memory.dump linux.netstat
```

### 5.3 Rekall (alternative)

```bash
pip install rekall-agent
rekall -f memory.raw pslist
rekall -f memory.raw netscan
rekall -f memory.raw malfind
```

### 5.4 MemProcFS (mount memory as filesystem)

```bash
git clone https://github.com/ufrisk/MemProcFS
./memprocfs -device memory.raw -mount /mnt/mem
ls /mnt/mem/
```

---

## 6. NETWORK FORENSICS

### 6.1 Packet Capture Analysis

**Wireshark:**
```bash
wireshark capture.pcap

# Filters
ip.addr == 1.2.3.4
tcp.port == 443
http.request
dns.qry.name contains "evil"
tls.handshake.extensions_server_name
http.request.method == POST
ftp.request.command == "USER"
```

**tshark (CLI):**
```bash
tshark -r capture.pcap
tshark -r capture.pcap -Y "http.request"
tshark -r capture.pcap -Y "dns.qry.name contains evil"
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport
tshark -r capture.pcap -Y "http" -T fields -e http.host -e http.request.uri
tshark -r capture.pcap -z follow,tcp,ascii,0
tshark -r capture.pcap -q -z io,stat,1
```

**tcpdump:**
```bash
sudo tcpdump -i any -w capture.pcap
sudo tcpdump -r capture.pcap -A
sudo tcpdump -r capture.pcap 'tcp port 80'
```

**NetworkMiner:**
```bash
# GUI, extracts files, credentials, hosts from pcap
networkminer capture.pcap
```

### 6.2 Flow Analysis

```bash
# argus
argus -r capture.pcap -w argus.out
ra -r argus.out -n

# nfdump / nfsen
nfdump -r nfcapd.202401011200

# Zeek (formerly Bro)
zeek -r capture.pcap
cat conn.log
cat http.log
cat dns.log
cat files.log
```

### 6.3 Credential Extraction

```bash
# From pcap
tcpdump -r capture.pcap -A | grep -i "user\|pass"
dsniff -r capture.pcap
ettercap -T -q -r capture.pcap -w output.pcap
```

### 6.4 Log Analysis

```bash
# Web server logs
grep "POST" access.log
grep "200" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Syslog
grep "Failed password" /var/log/auth.log
journalctl -u ssh --since "1 hour ago"
journalctl --since "2024-06-01" --until "2024-06-02"

# Windows Event Logs (via evtx)
sudo apt install python3-evtx
evtx_dump.py Security.evtx > security.xml
```

---

## 7. ARTIFACT ANALYSIS (WINDOWS)

### 7.1 Registry

```bash
# RegRipper
rip.pl -r NTUSER.DAT -f ntuser
rip.pl -r SYSTEM -f system
rip.pl -r SOFTWARE -f software
rip.pl -r SAM -f sam
rip.pl -r SECURITY -f security

# Specific artifacts
rip.pl -r NTUSER.DAT -p userassist
rip.pl -r NTUSER.DAT -p run
rip.pl -r NTUSER.DAT -p recentdocs
rip.pl -r SYSTEM -p usb
rip.pl -r SYSTEM -p services
rip.pl -r SYSTEM -p timezone
rip.pl -r SAM -p samparse

# Registry Explorer (GUI, Windows)
# Windows Registry Recovery (MiTeC)
# Regshot (before/after comparison)
```

### 7.2 Prefetch (Program Execution)

```bash
# Windows Prefetch files: C:\Windows\Prefetch\*.pf
PECmd.exe -d C:\Windows\Prefetch --csv output/
PECmd.exe -f FILE.pf
# WinPrefetchView (GUI)
```

### 7.3 ShimCache / AmCache

```bash
# AppCompatCache
AppCompatCacheParser.exe -f SYSTEM --csv output/

# AmCache
AmcacheParser.exe -f Amcache.hve --csv output/
```

### 7.4 LNK Files / Jump Lists

```bash
# LNK parser
LECmd.exe -d C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent --csv output/
LECmd.exe -f file.lnk

# Jump Lists
JLECmd.exe -d C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent --csv output/
```

### 7.5 Browser Artifacts

```bash
# BrowsingHistoryView (Windows GUI)
# DB Browser for SQLite

# Chrome history
sqlite3 "History" "SELECT url, title, last_visit_time FROM urls ORDER BY last_visit_time DESC LIMIT 50;"

# Firefox history
sqlite3 "places.sqlite" "SELECT url, title, last_visit_date FROM moz_places ORDER BY last_visit_date DESC LIMIT 50;"

# Chromium cache
chrome_cache_parser.py

# Hindsight (Chrome)
hindsight.py -i "Chrome Profile" -o output/
```

### 7.6 USB Forensics

```bash
# USBSTOR keys
rip.pl -r SYSTEM -p usb
usbdeviceforensics.py

# SetupAPI logs
grep -a "USB" C:\Windows\INF\setupapi.dev.log
```

### 7.7 Event Logs

```bash
# evtx_dump
evtx_dump.py Security.evtx > security.xml

# Chainsaw (fast)
chainsaw hunt evtx/ --rules sigma/ -o results.json

# Hayabusa
hayabusa.exe csv-timeline -d evtx/ -o timeline.csv

# Key Event IDs
# 4624  → Successful logon
# 4625  → Failed logon
# 4634  → Logoff
# 4648  → Explicit credentials
# 4672  → Special privileges
# 4688  → Process creation
# 4720  → User created
# 4726  → User deleted
# 4728  → Added to global group
# 4732  → Added to local group
# 1102  → Audit log cleared
# 7045  → Service installed
# 4104  → PowerShell script block
```

### 7.8 Shellbags, MFT, SRUM

```bash
# Shellbags
SBECmd.exe -d "C:\Users\<user>\NTUSER.DAT" --csv output/

# MFT
MFTECmd.exe -f MFT --csv output/

# SRUM
SrumECmd.exe -f SRUDB.dat -r SOFTWARE --csv output/
```

---

## 8. ARTIFACT ANALYSIS (LINUX)

### 8.1 System Artifacts

```bash
# Login history
last
lastlog
lastb
who
w
cat /var/log/wtmp
cat /var/log/btmp

# Auth logs
cat /var/log/auth.log
cat /var/log/secure
journalctl -u ssh

# Shell history
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.python_history

# Cron jobs
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
cat /var/spool/cron/crontabs/*

# Services
systemctl list-units --type=service
ls /etc/systemd/system/
ls /lib/systemd/system/

# User accounts
cat /etc/passwd
cat /etc/shadow
cat /etc/group
```

### 8.2 File System Artifacts

```bash
# Recently modified files
find / -mtime -7 -type f 2>/dev/null
find / -mmin -60 -type f 2>/dev/null

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Writable by all
find / -perm -o+w -type f 2>/dev/null

# Hidden files
find / -name ".*" -type f 2>/dev/null

# Deleted files still open
lsof +L1
```

### 8.3 Package & Persistence

```bash
# Installed packages
dpkg -l
rpm -qa
pacman -Q

# Recently installed
grep " install " /var/log/dpkg.log
grep "Installed:" /var/log/yum.log

# Startup
ls -la /etc/init.d/
cat /etc/rc.local
ls -la /etc/profile.d/

# SSH keys
cat ~/.ssh/authorized_keys
cat /etc/ssh/sshd_config
```

---

## 9. ARTIFACT ANALYSIS (macOS)

```bash
# Unified logs
log show --last 1h --predicate 'eventMessage CONTAINS "sudo"'

# FSEvents
# Requires fseventsparser
fseventsparser.py -s fsevents -o output/

# Quarantine events
sqlite3 ~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2

# Plist analysis
plutil -p file.plist
defaults read com.apple.something

# Keychain
security dump-keychain -d ~/Library/Keychains/login.keychain-db

# Browsers, Safari history
sqlite3 ~/Library/Safari/History.db
```

---

## 10. MOBILE FORENSICS

### 10.1 Android

```bash
# ADB
adb devices
adb shell
adb pull /sdcard/ /evidence/sdcard/
adb backup -apk -shared -all -f backup.ab
# Convert backup
java -jar abe.jar unpack backup.ab backup.tar

# Root required for full imaging
adb shell su -c "dd if=/dev/block/mmcblk0 of=/sdcard/full.img"
adb pull /sdcard/full.img

# Tools
# - Autopsy (supports Android images)
# - MOBILedit
# - Cellebrite (commercial)
# - Andriller
# - ALEAPP (Android Logs Events And Protobuf Parser)
```

### 10.2 iOS

```bash
# iTunes/Finder backup
# Location:
#   Windows: %APPDATA%\Apple Computer\MobileSync\Backup\
#   macOS:   ~/Library/Application Support/MobileSync/Backup/

# iTunes backup parser
pip install iphone_backup_decrypt
# or
# iPhone Backup Extractor
# iBackup Viewer
# Elcomsoft iOS Forensic Toolkit (commercial)
# Cellebrite UFED (commercial)

# Full filesystem requires checkm8 (jailbreak) or commercial tools
```

---

## 11. CLOUD FORENSICS

```bash
# AWS
aws cloudtrail lookup-events --max-results 50
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser
aws s3 ls s3://bucket/

# Azure
az monitor activity-log list
az monitor activity-log list --start-time 2024-06-01

# GCP
gcloud logging read "resource.type=gce_instance"
gcpdiag

# Tools
# - CloudTrail → JSON events, analyze with jq
# - Athena for S3-based CloudTrail logs
# - GuardDuty findings
# - Azure Sentinel
```

---

## 12. TOOLS SUMMARY

| Category | Tools |
|----------|-------|
| **Imaging** | dd, dcfldd, dc3dd, FTK Imager, Guymager, ewfacquire |
| **Memory** | LiME, AVML, WinPmem, DumpIt, Volatility 3, MemProcFS |
| **Disk** | Autopsy, Sleuth Kit, FTK, EnCase, X-Ways |
| **Carving** | Scalpel, Foremost, Photorec, Bulk Extractor |
| **Timeline** | log2timeline/Plaso, mactime, Autopsy timeline |
| **Network** | Wireshark, tshark, Zeek, NetworkMiner, tcpdump |
| **Windows Artifacts** | RegRipper, PECmd, LECmd, JLECmd, SBECmd, MFTECmd, AmcacheParser, EvtxECmd, Chainsaw, Hayabusa |
| **Mobile** | ADB, ALEAPP, iLEAPP, Cellebrite, Magnet AXIOM |
| **Linux** | TSK, chkrootkit, rkhunter, auditd, journalctl |
| **macOS** | fseventsparser, mac_apt, APOLLO |
| **Cloud** | CloudTrail, Athena, GuardDuty, Azure Sentinel |

---

## 13. TIPS

1. **Write blocker always** — software or hardware.
2. **Hash before and after** — evidence integrity.
3. **Document every command** — with timestamps.
4. **Work on copies** — never on originals.
5. **Log2timeline is your friend** — build a super-timeline.
6. **Volatility 3 > Volatility 2** for modern systems.
7. **Carve unallocated space** — deleted files live there.
8. **Prefetch + ShimCache + AmCache** = execution history.
9. **Event IDs 4688 + Sysmon** for process lineage.
10. **Combine memory + disk** — memory has runtime state, disk has persistence.
11. **Use YARA** to find known malware in memory/disk.
12. **Zeek for network**, Wireshark for deep packet.
13. **Chain of custody** — timestamp and sign every step.
14. **Report clearly** — findings must be reproducible.
