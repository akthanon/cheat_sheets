# WEB RECON TOOLS COMPLETE CHEAT SHEET

## 1. WHAT IS THIS?

Additional recon tools for **web and network discovery**:

- **ffuf** — fast web fuzzer
- **Gobuster** — directory/DNS/vhost brute-force
- **CeWL** — custom wordlist generator
- **Nuclei** — template-based vuln scanner
- **Masscan** — fast network scanner

**Key fact:** These are the modern replacements for slow legacy tools. Combine them in pipelines.

---

## 2. FFUF — FAST WEB FUZZER

### 2.1 Basic

```bash
ffuf -u <URL/FUZZ> -w <wordlist>
ffuf -u http://example.com/FUZZ -w /usr/share/wordlists/raft-large-directories.txt
```

### 2.2 Options

```bash
-u <url>            # URL with FUZZ
-w <wordlist>       # wordlist
-e .php,.html       # extensions
-t 40               # threads
-H "Header: val"    # custom header
-X POST -d "user=admin&pass=FUZZ"   # POST body fuzz
```

### 2.3 Filtering

```bash
-mc 200,301,302     # match only these status
-fc 404             # exclude 404
-fs <bytes>         # filter by size
-fw <words>         # filter by words
```

### 2.4 Output

```bash
-o results.json -of json
-o vhosts.csv -of csv
```

### 2.5 Examples

```bash
ffuf -u http://example.com/FUZZ -w /usr/share/wordlists/raft-large-directories.txt -t 50 -mc 200,301 -o results.json -of json
ffuf -u http://FUZZ.example.com/ -w vhosts.txt -t 30 -mc 200 -o vhosts.csv -of csv
ffuf -u "http://example.com/page.php?id=FUZZ" -w params.txt -t 40 -fc 404 -o params.json -of json
```

---

## 3. GOBUSTER — DIRECTORY / DNS / VHOST

### 3.1 Install

```bash
sudo apt install gobuster
```

### 3.2 Modes

```
dir     → directories and files
dns     → subdomains
vhost   → virtual hosts
s3      → AWS S3 buckets
fuzz    → parameter fuzzing
gcs     → Google Cloud Storage
```

### 3.3 DIR Mode

```bash
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://example.com -w wordlist.txt -x php,html,txt
gobuster dir -u http://example.com -w wordlist.txt -s 200,204,301,302,307,403
gobuster dir -u http://example.com -w wordlist.txt -t 50 -o result.txt
gobuster dir -u http://example.com -w wordlist.txt -H "Authorization: Bearer token123"
gobuster dir -u http://example.com -w wordlist.txt --proxy http://127.0.0.1:8080
gobuster dir -u http://example.com -w wordlist.txt -r
```

### 3.4 DNS Mode

```bash
gobuster dns -d example.com -w /usr/share/wordlists/dns/subdomains-top1million-5000.txt
gobuster dns -d example.com -w wordlist.txt -o result.txt
gobuster dns -d example.com -w wordlist.txt -i
gobuster dns -d example.com -w wordlist.txt -q
gobuster dns -d example.com -w wordlist.txt -r 8.8.8.8
```

### 3.5 VHOST Mode

```bash
gobuster vhost -u http://example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt
gobuster vhost -u http://example.com -w wordlist.txt --no-redirect
```

### 3.6 FUZZ Mode

```bash
gobuster fuzz -u "http://example.com/FUZZ" -w wordlist.txt
gobuster fuzz -u "http://example.com/login" -w params.txt -H "Content-Type: application/x-www-form-urlencoded" -d "user=admin&pass=FUZZ"
```

### 3.7 Common Options

```
-u URL
-w wordlist
-t threads
-o output
-x extensions (dir)
-s status codes
-r recursive
-H header
--proxy proxy
--timeout seconds
```

---

## 4. CEWL — CUSTOM WORDLIST GENERATOR

```bash
cewl <url> -w wordlist.txt
cewl -d 3 -m 5 -w site_words.txt https://example.com
cewl -d <depth> -m <min_length> -w <file> -e --meta <url>
```

**Options:**
```
-d depth        # spider depth (default 2)
-m min_length   # min word length
-w file         # output file
-e              # extract emails
--meta          # include metadata (title, description)
```

**Tips:** Combine output with john/hashcat for targeted cracking.

---

## 5. NUCLEI — TEMPLATE-BASED VULN SCANNER

### 5.1 Install

```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
nuclei -version
nuclei -update-templates
nuclei -ut
```

### 5.2 Basic Use

```bash
nuclei -u https://example.com
nuclei -l targets.txt
nuclei -severity medium,high,critical
nuclei -u https://example.com -o resultados.txt
nuclei -u https://example.com -vv
nuclei -u https://example.com -tags cve
nuclei -silent
```

### 5.3 Templates

```bash
nuclei -tl                             # list all
nuclei -tl | grep wordpress            # find specific
nuclei -t cves/                        # by folder
nuclei -t http/misconfig/              # by type
nuclei -t cves/2023/CVE-2023-1234.yaml -u https://example.com
```

### 5.4 Flags

```
-quiet          no logs
-silent         no banner
-debug          show req/resp
-no-color       disable colors
-json           JSON output
-rate-limit X   req/sec
-concurrency X  threads
-retries X      retries
-timeout X      seconds
-proxy http://127.0.0.1:8080
-header "Key: Value"
-var key=value
```

### 5.5 Pipelines

```bash
subfinder -d example.com -silent | nuclei -t cves/ -o vulns.txt
subfinder -d example.com -silent | httpx -silent | nuclei -silent -o report.txt
cat domains.txt | httpx -silent | nuclei -tags cve,misconfig -o scan.txt
```

### 5.6 Custom Templates

```bash
nuclei -nt -t mytemplate.yaml        # generate base
nuclei -t mytemplate.yaml -u https://example.com -debug
nuclei -t template.yaml -var user=admin -u https://example.com
```

**Template locations:**
```
Linux:   ~/.local/nuclei-templates/
MacOS:   ~/Library/nuclei-templates/
Windows: C:\Users\USERNAME\nuclei-templates\
```

---

## 6. MASSCAN — FAST NETWORK SCANNER

### 6.1 Install

```bash
sudo apt update && sudo apt install masscan
```

### 6.2 Basic Use

```bash
masscan <targets> -p <ports> --rate <pps> -oL output.txt
masscan 192.168.1.0/24 -p22,80 --rate 1000 -oL scan.txt
masscan 10.0.0.0/16 --top-ports 100 --rate 100000 -oX top100.xml
sudo masscan 0.0.0.0/0 -p0-65535 --rate 100000 --excludefile exclude.txt -oL internet.txt
```

### 6.3 Options

```
-p <ports>          # 22,80 or 1-65535 or --top-ports N
--rate <pps>        # packets per second
-oL file            # list output
-oX file.xml        # XML output
-oJ file.json       # JSON output
--append-output     # append instead of overwrite
--pcap file         # save packets
--banners           # banner grab (slow)
-iL file            # read targets from file
--exclude <cidr>    # exclude ranges
--excludefile file  # exclude from file
--open-only         # show only open ports
--ping              # ICMP
--iflist            # list interfaces
```

### 6.4 Recommended Flow

```
1. Define targets and excludefile.
2. masscan for speed (find hosts/ports).
3. Export (-oX / -oL).
4. Rescan with nmap for service details.
```

### 6.5 Warnings

- Masscan generates **lots of traffic**.
- **Never scan without written authorization.**
- May trigger alerts, blocks, or legal issues.
- Use `--excludefile` and lower rates on unstable networks.
