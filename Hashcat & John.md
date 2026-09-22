# HASHCAT & JOHN THE RIPPER COMPLETE CHEAT SHEET

## 1. INSTALLATION AND SETUP

### Arch Linux (NVIDIA GPU)
```bash
sudo pacman -S nvidia nvidia-utils opencl-nvidia opencl-headers ocl-icd clinfo
sudo pacman -S hashcat john
nvidia-smi
clinfo
hashcat -I
```

### Ubuntu/Debian
```bash
sudo apt install nvidia-driver-550 nvidia-cuda-toolkit
sudo apt install opencl-headers clinfo ocl-icd-libopencl1 ocl-icd-dev
sudo apt install hashcat john
```

---

## 2. HASH RECONNAISSANCE

```bash
hash-identifier
hashid "hash_here"
hashcat -m 0 hash.txt --example-hashes
```

---

## 3. HASHCAT BASIC COMMANDS

### General Structure
```bash
hashcat -m <mode> -a <attack> hash.txt dictionary.txt [options]
```

### Attack Modes (-a)
```bash
-a 0  # Dictionary
-a 1  # Combination
-a 3  # Mask (brute-force)
-a 6  # Dictionary + Mask
-a 7  # Mask + Dictionary
```

### Common Hash Modes (-m)
```bash
-m 0    # MD5
-m 100  # SHA1
-m 1400 # SHA256
-m 1700 # SHA512
-m 1000 # NTLM
-m 3000 # LM
-m 13400 # KeePass
-m 17200 # PKZIP
-m 13600 # WinZip
-m 2500  # WPA/WPA2
-m 22000 # WPA-PMKID
```

### Useful Options
```bash
-O          # Optimize kernel (faster)
-w 3        # Performance profile (1-4)
--force     # Force execution
--show      # Show cracked passwords
--status    # Show progress
--status-timer=1  # Update every X seconds
-d 1        # Use specific device (GPU)
-d 2        # Use CPU
```

---

## 4. JOHN THE RIPPER BASIC COMMANDS

### General Structure
```bash
john --format=<format> --wordlist=dictionary.txt hash.txt
```

### Common Formats
```bash
--format=Raw-MD5    # MD5
--format=NT         # NTLM
--format=LM         # LM
--format=KeePass    # KeePass
--format=PKZIP      # ZIP
--format=rar        # RAR
```

### Useful Options
```bash
--rules=best64      # Apply mutation rules
--external=name     # Use custom external rule
--show              # Show results
--restore           # Resume session
--incremental       # Brute-force
```

---

## 5. MASK ATTACKS (HASHCAT)

### Character Sets
```bash
?l  # lowercase letters (a-z)
?u  # uppercase letters (A-Z)
?d  # digits (0-9)
?s  # special characters (!@#$%...)
?a  # all of the above combined
?h  # hexadecimal (0-9, a-f)
?H  # hexadecimal (0-9, A-F)
```

### Mask Examples
```bash
hashcat -a 3 hash.txt ?d?d?d?d?d?d?d?d
hashcat -a 3 hash.txt ?l?l?l?l?l?l
hashcat -a 3 hash.txt ?a?a?a?a?a?a?a?a
hashcat -a 3 hash.txt palabra?d?d?d?d
hashcat -a 3 hash.txt ?u?l?l?l?l?d?d?d?s
```

---

## 6. COMBINATION ATTACKS

```bash
hashcat -a 1 hash.txt dict1.txt dict2.txt
hashcat -a 6 hash.txt dict.txt ?d?d?d?d
hashcat -a 7 hash.txt ?u?l?l?l dict.txt
```

---

## 7. MUTATION RULES

### Common Rules in John
```bash
c       # Capitalize
u       # Uppercase
l       # Lowercase
r       # Reverse
$!      # Append ! at the end
^1      # Prepend 1 at the beginning
```

### Using Rules
```bash
john --wordlist=dictionary.txt --rules=best64 hash.txt
hashcat -a 0 hash.txt dictionary.txt -r /usr/share/hashcat/rules/best64.rule
```

---

## 8. WORDLIST MANAGEMENT

### Download Wordlists
```bash
wget https://download.weakpass.com/wordlists/90/rockyou.txt.gz
gunzip rockyou.txt.gz
git clone https://github.com/danielmiessler/SecLists.git
ls /usr/share/wordlists/
```

### Create Custom Wordlists
```bash
echo "word" > dict.txt
seq 1000 9999 > numbers.txt
cat dict1.txt dict2.txt > combined.txt
crunch 8 8 abc123 -o dict.txt
```

---

## 9. PERFORMANCE OPTIMIZATION

```bash
hashcat -O -w 3 -d 1 hash.txt dict.txt
hashcat -d 1,2 hash.txt dict.txt
watch -n 1 nvidia-smi
hashcat hash.txt --status --status-timer=1
```

---

## 10. ADDITIONAL USEFUL COMMANDS

### Verify and Show Results
```bash
hashcat -m 0 hash.txt --show
john --show hash.txt
cat ~/.hashcat/hashcat.potfile
cat ~/.john/john.pot
```

### Format Conversion
```bash
zip2john archive.zip > hash.txt
rar2john archive.rar > hash.txt
keepass2john database.kdbx > hash.txt
pdf2john archive.pdf > hash.txt
```

### Troubleshooting
```bash
hashid hash.txt
hashcat -m 0 --example-hashes
hashcat --force hash.txt dict.txt
```

---

## 11. QUICK TIPS

1. **GPU > CPU**: Always use GPU if available.
2. **Wordlist with rules**: More effective than pure brute-force.
3. **Masks**: Useful for known patterns.
4. **Verify format**: Ensure the hash is in the correct format.
5. **Monitor progress**: Use `--status` to see progress.

---

## 12. QUICK REFERENCE OF COMMON MODES

```bash
MD5: 0 | SHA1: 100 | SHA256: 1400 | SHA512: 1700
NTLM: 1000 | LM: 3000 | KeePass: 13400
ZIP: 17200 | RAR: 13000 | WPA: 2500
```

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
sudo pacman -S nvidia nvidia-utils opencl-nvidia opencl-headers ocl-icd clinfo
sudo pacman -S hashcat john
nvidia-smi
clinfo
hashcat -I
sudo apt install nvidia-driver-550 nvidia-cuda-toolkit
sudo apt install opencl-headers clinfo ocl-icd-libopencl1 ocl-icd-dev
sudo apt install hashcat john
hash-identifier
hashid "hash_here"
hashcat -m 0 hash.txt --example-hashes
hashcat -m <mode> -a <attack> hash.txt dictionary.txt [options]
-a 0
-a 1
-a 3
-a 6
-a 7
-m 0
-m 100
-m 1400
-m 1700
-m 1000
-m 3000
-m 13400
-m 17200
-m 13600
-m 2500
-m 22000
-O
-w 3
--force
--show
--status
--status-timer=1
-d 1
-d 2
john --format=<format> --wordlist=dictionary.txt hash.txt
--format=Raw-MD5
--format=NT
--format=LM
--format=KeePass
--format=PKZIP
--format=rar
--rules=best64
--external=name
--show
--restore
--incremental
?l
?u
?d
?s
?a
?h
?H
hashcat -a 3 hash.txt ?d?d?d?d?d?d?d?d
hashcat -a 3 hash.txt ?l?l?l?l?l?l
hashcat -a 3 hash.txt ?a?a?a?a?a?a?a?a
hashcat -a 3 hash.txt palabra?d?d?d?d
hashcat -a 3 hash.txt ?u?l?l?l?l?d?d?d?s
hashcat -a 1 hash.txt dict1.txt dict2.txt
hashcat -a 6 hash.txt dict.txt ?d?d?d?d
hashcat -a 7 hash.txt ?u?l?l?l dict.txt
c
u
l
r
$!
^1
john --wordlist=dictionary.txt --rules=best64 hash.txt
hashcat -a 0 hash.txt dictionary.txt -r /usr/share/hashcat/rules/best64.rule
wget https://download.weakpass.com/wordlists/90/rockyou.txt.gz
gunzip rockyou.txt.gz
git clone https://github.com/danielmiessler/SecLists.git
ls /usr/share/wordlists/
echo "word" > dict.txt
seq 1000 9999 > numbers.txt
cat dict1.txt dict2.txt > combined.txt
crunch 8 8 abc123 -o dict.txt
hashcat -O -w 3 -d 1 hash.txt dict.txt
hashcat -d 1,2 hash.txt dict.txt
watch -n 1 nvidia-smi
hashcat hash.txt --status --status-timer=1
hashcat -m 0 hash.txt --show
john --show hash.txt
cat ~/.hashcat/hashcat.potfile
cat ~/.john/john.pot
zip2john archive.zip > hash.txt
rar2john archive.rar > hash.txt
keepass2john database.kdbx > hash.txt
pdf2john archive.pdf > hash.txt
hashid hash.txt
hashcat -m 0 --example-hashes
hashcat --force hash.txt dict.txt
MD5: 0 | SHA1: 100 | SHA256: 1400 | SHA512: 1700
NTLM: 1000 | LM: 3000 | KeePass: 13400
ZIP: 17200 | RAR: 13000 | WPA: 2500
```
