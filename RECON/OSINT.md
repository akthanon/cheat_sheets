# OSINT COMPLETE CHEAT SHEET

## 1. WHAT IS OSINT?

**OSINT (Open Source Intelligence)** is the collection and analysis of information from publicly available sources. In pentesting, it is the **first phase** of any engagement and is used to:

- Discover domains, subdomains, and IPs
- Find emails, usernames, and phone numbers
- Identify technologies and exposed services
- Map the attack surface
- Gather metadata from documents and images
- Profile people and organizations

**Key fact:** A well-executed OSINT phase can save hours of active scanning and reveal hidden assets.

---

## 2. INSTALLATION (COMMON DEPENDENCIES)

```bash
sudo apt install python3-venv python3-pip pipx git whois -y
pipx ensurepath
```

---

## 3. THEHARVESTER — EMAIL, SUBDOMAIN, HOST GATHERING

```bash
git clone https://github.com/laramies/theHarvester
cd theHarvester
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
pip install .

python3 theHarvester.py -h
python3 theHarvester.py -d utculiacan.edu.mx -b all
theHarvester -d utculiacan.edu.mx -b all
```

**Common options:**
```bash
-d <domain>         # Target domain
-b <source>         # Source (all, google, bing, duckduckgo, etc.)
-l <limit>          # Limit results
-f <file>           # Output file (HTML/XML)
```

---

## 4. SPIDERFOOT — AUTOMATED OSINT FRAMEWORK

```bash
git clone https://github.com/smicallef/spiderfoot
cd spiderfoot
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
python3 ./sf.py -l 127.0.0.1:5001
```

Access the web interface at `http://127.0.0.1:5001`.

---

## 5. SHERLOCK — USERNAME ENUMERATION

```bash
pipx install sherlock-project
pipx ensurepath
sherlock user1 user2 user3
```

**Options:**
```bash
sherlock --timeout 5 <username>
sherlock --output results.txt <username>
sherlock --csv <username>
```

---

## 6. HOLEHE — EMAIL REGISTRATION CHECK

```bash
pipx install holehe
pipx ensurepath
holehe test@gmail.com
```

---

## 7. SHODAN — INTERNET-CONNECTED DEVICE SEARCH

```bash
sudo apt install python3-venv -y
python3 -m venv shodan
source shodan/bin/activate
pip install shodan setuptools
pip install --upgrade distribute

shodan init TU_API_KEY
shodan search apache
shodan search nginx port:80 country:MX
shodan search webcamxp
shodan search "default password"
shodan host 8.8.8.8
shodan myip
```

**Useful CLI commands:**
```bash
shodan count apache
shodan stats nginx
shodan info
shodan alert create "MyAlert" 1.2.3.4
```

---

## 8. EXIFTOOL — METADATA EXTRACTION

```bash
sudo apt install libimage-exiftool-perl -y

exiftool image.jpg
exiftool document.pdf
exiftool -gpslatitude -gpslongitude image.jpg
exiftool -all= file.jpg
exiftool -Author -CreateDate -GPS* file.jpg
```

**Recursive scan:**
```bash
exiftool -r -csv -ext jpg -ext png -ext pdf ./folder > metadata.csv
```

---

## 9. WHOIS — DOMAIN AND IP REGISTRATION INFO

```bash
sudo apt install whois -y
whois example.com
whois 113.132.20.206
```

**Reverse WHOIS:**
```bash
whois -h whois.arin.net "n 8.8.8.8"
```

---

## 10. MALTEGO — VISUAL LINK ANALYSIS

1. Download the `.deb` package from the official site.
2. Install Java:
```bash
sudo apt install default-jdk -y
```
3. Install Maltego:
```bash
sudo dpkg -i maltego.deb
```
4. Launch:
```bash
maltego
```
5. Register a free account to use the Community Edition.

---

## 11. SOCIAL ANALYZER — USERNAME ACROSS SOCIAL NETWORKS

```bash
git clone https://github.com/qeeqbox/social-analyzer
cd social-analyzer
npm start
```

Access the web UI and search for a username across 1000+ sites.

---

## 12. SMOT — SOCIAL MEDIA OSINT TOOLS COLLECTION

Repository with curated tools:
```
https://github.com/osintambition/Social-Media-OSINT-Tools-Collection
```

---

## 13. PHONEINFOGA — PHONE NUMBER OSINT

```bash
bash <( curl -sSL https://raw.githubusercontent.com/sundowndev/phoneinfoga/master/support/scripts/install )
sudo install ./phoneinfoga /usr/local/bin/phoneinfoga
phoneinfoga version
phoneinfoga scan -n "+1 555-444-3333"
phoneinfoga scan -n "+52 667-204-2506"
```

---

## 14. DIRSEARCH — DIRECTORY AND FILE BRUTE FORCE

```bash
git clone https://github.com/maurosoria/dirsearch
cd dirsearch
pip install setuptools
pip install --upgrade distribute
python3 dirsearch.py -u http://www.example.com -w wordlist.txt
```

**Common options:**
```bash
-e php,html,js        # Extensions
-t 50                 # Threads
-x 404,403            # Exclude status codes
--random-agent        # Random user agent
-r                    # Recursive
```

---

## 15. SECLISTS — WORDLIST COLLECTION

```bash
git clone https://github.com/danielmiessler/SecLists
```

**Useful paths inside SecLists:**
```
Discovery/Web-Content/         # Directories and files
Discovery/DNS/                 # Subdomains
Passwords/                     # Password lists
Usernames/                     # Username lists
Fuzzing/                       # Fuzzing payloads
```

---

## 16. COMPLEMENTARY OSINT TOOLS

### 16.1 Subdomain Enumeration
```bash
subfinder -d example.com -o subdomains.txt
amass enum -d example.com
assetfinder example.com
```

### 16.2 DNS Enumeration
```bash
dnsrecon -d example.com
dnsenum example.com
fierce --domain example.com
```

### 16.3 Google Dorking
```
site:example.com filetype:pdf
site:example.com inurl:admin
site:example.com intitle:"index of"
inurl:".git" site:example.com
inurl:".env" site:example.com
```

### 16.4 Certificate Transparency
```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq .
```

### 16.5 Wayback Machine
```bash
curl "http://archive.org/wayback/available?url=example.com"
waybackurls example.com
```

### 16.6 Email Breach Check
```
https://haveibeenpwned.com/
https://dehashed.com/
```

### 16.7 Metadata from Public Documents
```bash
metagoofil -d example.com -t pdf,doc,docx -l 100 -n 50 -o output
```

---

## 17. TIPS FOR TESTING

1. Start with **passive** OSINT (WHOIS, Shodan, crt.sh) before touching the target.
2. Use **theHarvester** for emails and subdomains.
3. **Sherlock** and **Holehe** for username/email footprinting.
4. **Shodan** for exposed services (RDP, Elasticsearch, MongoDB, cameras).
5. **exiftool** on any document/image found publicly.
6. Combine **subfinder + amass + assetfinder** for maximum subdomain coverage.
7. Use **Google Dorks** for accidental exposure (backups, .env, .git).
8. Cross-reference findings with **Wayback Machine** for old endpoints.
9. Save everything in a structured folder per target.
10. Only use OSINT on targets you are authorized to test.

---

## 18. RECOMMENDED WORKFLOW

```
1. WHOIS           → ownership info
2. crt.sh          → subdomains via CT logs
3. subfinder/amass → more subdomains
4. theHarvester    → emails + hosts
5. Shodan          → exposed services
6. exiftool        → metadata from docs
7. Sherlock/Holehe → usernames/emails
8. dirsearch/ffuf  → directories on live hosts
9. Google Dorks    → accidental exposure
10. Waybackurls    → historical endpoints
```
