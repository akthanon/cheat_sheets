# FIREWALL, IDS & IPS CHEAT SHEET

## 1. WHAT IS THIS?

Defensive Linux infrastructure: **firewall rules**, **brute-force protection**, and **IDS/IPS**. Essential for hardening your attack box or setting up a lab.

- **UFW** – Simple firewall (Debian/Ubuntu)
- **iptables** – Advanced packet filtering
- **Fail2ban** – Auto-ban brute force
- **Suricata / Snort** – IDS/IPS

**Key fact:** Knowing defenses helps you bypass them. This is also how you protect your C2.

---

## 2. UFW — UNCOMPLICATED FIREWALL

### 2.1 Status & Control

```bash
sudo ufw status verbose
sudo ufw enable
sudo ufw disable
sudo ufw reset
sudo ufw reload
```

### 2.2 Basic Rules

```bash
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 23
sudo ufw allow from 192.168.1.0/24
```

### 2.3 Advanced Rules

```bash
sudo ufw allow from 192.168.1.100 to any port 3306
sudo ufw deny from 10.0.0.5
sudo ufw allow 8000
```

### 2.4 Logging

```bash
sudo ufw logging on
sudo ufw logging high
sudo tail -f /var/log/ufw.log
```

---

## 3. IPTABLES — ADVANCED FIREWALL

### 3.1 View Rules

```bash
sudo iptables -L -v -n
sudo iptables -F                    # flush all rules
```

### 3.2 Basic Rules

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -s 192.168.1.100 -j DROP
sudo iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
```

### 3.3 Default Policies

```bash
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP
```

### 3.4 NAT & Port Redirection

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### 3.5 Save & Restore

```bash
sudo iptables-save > ~/iptables_backup.rules
sudo iptables-restore < ~/iptables_backup.rules
```

---

## 4. FAIL2BAN — BRUTE FORCE PROTECTION

### 4.1 Status

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### 4.2 Configuration Files

```
/etc/fail2ban/jail.conf         # general (do not edit)
/etc/fail2ban/jail.local        # your overrides
/var/log/fail2ban.log           # events
/var/log/auth.log               # source log
```

### 4.3 Example Jail (jail.local)

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 600
findtime = 600
```

### 4.4 Manage Bans

```bash
sudo fail2ban-client set sshd unbanip 192.168.1.100
sudo fail2ban-client set sshd banip 192.168.1.100
sudo systemctl restart fail2ban
```

---

## 5. SURICATA / SNORT — IDS/IPS

### 5.1 Suricata — Install & Basic Run

```bash
sudo apt install suricata
sudo suricata -T -c /etc/suricata/suricata.yaml    # test config
sudo suricata -i eth0 -c /etc/suricata/suricata.yaml
sudo suricata-update                               # update rules
```

### 5.2 Snort — Install & Basic Run

```bash
sudo apt install snort
sudo snort -T -c /etc/snort/snort.conf             # test config
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0
```

### 5.3 Common Rule Paths

```
/var/lib/suricata/rules/
/etc/suricata/rules/
/etc/snort/rules/
```

### 5.4 Log Locations

```
/var/log/suricata/
/var/log/suricata/eve.json
/var/log/snort/
```

---

## 6. TIPS

1. Always use `ufw` if you don't need advanced rules — it's simpler and safer.
2. Save iptables rules before rebooting — they don't persist by default.
3. `fail2ban` + `ufw` is the minimum for any exposed Linux box.
4. `suricata-update` before running Suricata — old rules miss modern attacks.
5. Combine `iptables` with `conntrack` for stateful firewall.
6. Test rules with `--dry-run` when available.
7. Monitor `/var/log/ufw.log` and `/var/log/fail2ban.log` regularly.
8. Use `iptables -L -v -n --line-numbers` to see rule order.

---
