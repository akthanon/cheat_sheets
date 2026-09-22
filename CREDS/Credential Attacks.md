# CREDENTIAL ATTACKS COMPLETE CHEAT SHEET

## 1. WHAT IS THIS?

Credential attacks include brute force, hash cracking (already in Hashcat/John sheet), and **password analysis**:

- **Hydra** — online brute force (SSH, FTP, HTTP, RDP, etc.)
- **Pipal** — password dump statistics (offline analysis to build better wordlists)

**Key fact:** Online brute force is slow and noisy; use it only when no hashes are available.

---

## 2. HYDRA — ONLINE BRUTE FORCE

### 2.1 Basic Syntax

```bash
hydra -l USER -P wordlist.txt protocol://IP
```

### 2.2 Common Examples

```bash
# SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# FTP
hydra -L users.txt -P pass.txt ftp://192.168.1.100

# HTTP POST form
hydra -l admin -P pass.txt 192.168.1.100 http-post-form "/login.php:user=^USER^&pass=^PASS^:Login failed"

# RDP
hydra -t 4 -V -f -L users.txt -P passwords.txt rdp://192.168.1.100

# MySQL
hydra -L users.txt -P passwords.txt mysql://192.168.1.100

# VNC (no password check)
hydra -P /dev/null vnc://192.168.1.100
```

### 2.3 Flags

```
-l USER       fixed user
-L users.txt  user list
-p PASS       fixed password
-P pass.txt   password list
-t N          threads
-vV           verbose
-f            stop on first success
-s PORT       custom port
-o file       save results
-U            list supported services
```

### 2.4 Save Results

```bash
hydra -L users.txt -P pass.txt ssh://192.168.1.100 -o results.txt
```

---

## 3. PIPAL — PASSWORD DUMP ANALYSIS

### 3.1 What It Does

Pipal analyzes password dumps to show:
- Length distribution
- Common suffixes / prefixes
- Base words
- Patterns (e.g., `Password1`, `qwerty123`)

It does **not** crack passwords — it helps you build better wordlists.

### 3.2 Install

```bash
sudo apt install pipal
```

### 3.3 Usage

```bash
pipal passwords.txt
pipal -t 20 passwords.txt          # top 20
pipal -o report.txt passwords.txt  # save
pipal --list-checkers              # list checkers
pipal -v passwords.txt             # verbose
```

### 3.4 Workflow

```
1. Get a password dump (from breach, cracked hashes, etc.).
2. Run pipal on it.
3. Read stats: lengths, patterns, common words.
4. Build targeted wordlists for hashcat/john.
5. Crack with rules based on findings.
```

---

## 4. TIPS

1. Hydra is loud — check for **account lockout** before running.
2. Use `-f` to stop at first success.
3. Combine hydra output with a Mutated wordlist (CeWL or john rules).
4. For SSH, use `-t 4` or lower — high threads can drop connections.
5. For HTTP forms, capture the POST request in Burp first, then use `http-post-form`.
6. Pipal is for **strategy**, not cracking — pair it with hashcat rules.
7. Always use authorized wordlists (rockyou is fine for CTFs; not for prod).
8. Cross-reference pipal findings with `best64.rule` or `OneRuleToRuleThemAll`.
