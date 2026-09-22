# REVERSING COMPLETE CHEAT SHEET

## 1. WHAT IS REVERSING?

**Reverse Engineering (RE)** is the process of analyzing a compiled binary (EXE, DLL, ELF, Mach-O, APK) to understand its logic, structure, and behavior without access to source code. Use cases:

- **Malware analysis** (IOCs, C2, capabilities)
- **Vulnerability research** (find bugs in closed-source software)
- **CTF challenges** (crackmes, license checks)
- **Protocol reverse engineering** (proprietary network protocols)
- **Firmware analysis** (IoT, routers, embedded)
- **Binary patching** (bypass license, anti-debug)
- **Interoperability** (replace legacy closed-source software)

**Key fact:** RE is iterative — static first, dynamic second, back to static. Tools change, methodology doesn't.

---

## 2. FILE IDENTIFICATION

### 2.1 Basic Identification

```bash
file suspicious.bin
file suspicious.exe
file suspicious.elf

# Hashes
md5sum suspicious.exe
sha1sum suspicious.exe
sha256sum suspicious.exe
sha512sum suspicious.exe

# Entropy (packed vs plaintext)
ent suspicious.exe
ent -t suspicious.exe
```

### 2.2 Headers by Format

**PE (Windows):**
```bash
pefile suspicious.exe
pescan suspicious.exe
pecheck.py suspicious.exe
```

**ELF (Linux):**
```bash
readelf -h suspicious.elf
readelf -a suspicious.elf
```

**Mach-O (macOS):**
```bash
otool -h suspicious.macho
otool -L suspicious.macho
```

**APK (Android):**
```bash
unzip -l suspicious.apk
apktool d suspicious.apk
jadx -d output/ suspicious.apk
```

---

## 3. STATIC ANALYSIS

**Definition:** Analyze the binary **without executing it**. Extract structure, code, strings, and logic from the file itself.

### 3.1 Static Methodology

```
1. Identify format (file, pefile, readelf)
2. Extract strings (strings, FLOSS)
3. Check entropy (ent, binwalk)
4. Identify packer (DIE, PEiD)
5. If packed → unpack first
6. List imports/exports (objdump, readelf)
7. Disassemble (radare2, objdump, Ghidra, IDA)
8. Decompile (Ghidra, IDA, Binary Ninja)
9. Look for suspicious APIs (network, crypto, process)
10. Map to MITRE ATT&CK (capa)
```

### 3.2 Strings & Entropy

```bash
# ASCII strings
strings -n 6 suspicious.exe
strings -a -n 4 suspicious.exe

# Unicode (UTF-16LE, common in Windows)
strings -e l suspicious.exe > unicode_strings.txt
strings -e b suspicious.exe > big_endian.txt

# Filter interesting strings
strings suspicious.exe | grep -iE "http|https|api|token|password|secret"
strings suspicious.exe | grep -iE "\.dll|\.exe|\.sys"

# Extract URLs
strings suspicious.exe | grep -oE 'https?://[^ ]+'

# Entropy per section
ent suspicious.exe
binwalk suspicious.exe
binwalk -E suspicious.exe           # entropy graph

# FLOSS — obfuscated string extraction
floss suspicious.exe
floss --no-static-strings suspicious.exe
floss -n 6 suspicious.exe
floss -o output.txt suspicious.exe
```

### 3.3 Headers & Symbols

```bash
# objdump
objdump -d ./binary                 # disassemble
objdump -d -M intel ./binary        # Intel syntax
objdump -D ./binary                 # all sections
objdump -h ./binary                 # section headers
objdump -x ./binary                 # all headers
objdump -t ./binary                 # symbol table
objdump -T ./binary                 # dynamic symbols
objdump -s -j .rodata ./binary      # dump section
objdump -R ./binary                 # relocations
objdump -p ./binary                 # private headers
objdump -i ./binary                 # supported architectures

# nm
nm ./binary                         # symbols
nm -D ./binary                      # dynamic symbols
nm -C ./binary                      # demangle C++
nm -u ./binary                      # undefined symbols

# readelf
readelf -h ./binary                 # header
readelf -S ./binary                 # sections
readelf -s ./binary                 # symbols
readelf -d ./binary                 # dynamic section
readelf -n ./binary                 # notes
readelf -r ./binary                 # relocations
readelf -x .rodata ./binary         # hex dump of section
readelf -l ./binary                 # program headers
```

### 3.4 Disassembly

**radare2 / rizin:**
```bash
r2 ./binary
rizin ./binary

aaa                 # analyze all
afl                 # list functions
pdf @ main          # disassemble main
s main              # seek to main
V                   # visual mode
VV                  # graph mode
iI                  # info about binary
iz                  # strings in data sections
izz                 # all strings
iS                  # sections
iH                  # headers
ie                  # entrypoints
ii                  # imports
iE                  # exports
axt @ sym.imp.printf # xrefs to printf
/R pop rdi          # search for ROP gadgets
```

**Cutter (GUI for rizin):**
```bash
sudo apt install cutter
cutter ./binary
```

**objdump:**
```bash
objdump -d -M intel ./binary
objdump -d --start-address=0x400000 --stop-address=0x401000 ./binary
```

### 3.5 Decompilation

**Ghidra (NSA, free, powerful):**
```bash
sudo apt install ghidra
ghidraRun

# Workflow
#  1. New Project → Import binary → Analyze
#  2. Decompiler panel (right)
#  3. Search → For Strings
#  4. Xrefs to see who calls functions
#  5. Data Type Manager for structs
#  6. Script Manager for automation

# Headless mode
analyzeHeadless /path/to/project ProjectName -import binary -postScript script.py
```

**Binary Ninja (commercial, fast):**
```bash
# https://binary.ninja

# Python API
import binaryninja
bv = binaryninja.open_view("/path/to/binary")
for func in bv.functions:
    print(func.name)
```

**IDA Pro / IDA Free:**
```bash
# IDA Free: https://hex-rays.com/ida-free/
# IDA Pro: commercial

# Workflow
#  1. File → Open → select binary
#  2. Auto-analysis
#  3. Shift+F12 → strings window
#  4. X → xrefs
#  5. Space → graph view
#  6. F5 → decompiler (Hex-Rays)
#  7. Rename (N), comment (;), retype (Y)

# Batch mode
idat -A -S"script.idc" binary
idat64 -A -S"script.py" binary
```

### 3.6 Packer Detection & Unpacking

**Detect It Easy (DIE):**
```bash
sudo apt install diec
diec suspicious.exe
```

Detects: PE packers, compilers, protectors (VMProtect, Themida, Enigma, ASProtect).

**UPX:**
```bash
upx -l suspicious.exe               # detect UPX
upx -d suspicious.exe -o unpacked.exe
```

**Binwalk:**
```bash
binwalk suspicious.bin
binwalk -e suspicious.bin           # extract embedded files
binwalk -Me suspicious.bin          # recursive extraction
```

**Manual unpacking (Windows):**
```
1. Open in x64dbg
2. Set breakpoint on VirtualAlloc / VirtualProtect
3. Run until OEP (original entry point)
4. Dump process memory
5. Rebuild IAT with Scylla
6. Save unpacked binary
```

### 3.7 Capability Detection

**capa (MITRE ATT&CK mapping):**
```bash
capa suspicious.exe
capa -v suspicious.exe
capa -j suspicious.exe -o output.json
capa -r rules/ suspicious.exe
```

**YARA Rules:**
```bash
sudo apt install yara

yara rule.yar suspicious.exe
yara -r rules/ directory/

# Example rule
cat > example.yar <<EOF
rule Detect_Mimikatz {
    meta:
        description = "Detects Mimikatz"
    strings:
        \$a = "sekurlsa::logonpasswords"
        \$b = "lsadump::sam"
    condition:
        any of them
}
EOF
yara example.yar suspicious.exe
```

**PE-bear / PEStudio / CFF Explorer:**
```
- PE-bear:    free PE editor/analyzer
- PEStudio:   static analysis + VirusTotal integration
- CFF Explorer: PE header editing

# CFF Explorer → check:
#   - Optional Header (ASLR, DEP, CFG)
#   - Section Headers (entropy, flags)
#   - Import Table (suspicious APIs)
#   - Resources (embedded)
```

---

## 4. DYNAMIC ANALYSIS

**Definition:** Execute the binary **in a controlled environment** and observe behavior at runtime.

### 4.1 Dynamic Methodology + VM Setup

**Golden rules:**
- **Never** run untrusted binaries on your host.
- Use isolated VMs (VMware, VirtualBox, QEMU, Hyper-V).
- **Disable** shared folders, clipboard, drag-and-drop.
- **Fake** network with INetSim or FakeNet-NG.
- Take a **snapshot** before running.

**Setup checklist:**
```
[ ] Isolated VM (no bridged network)
[ ] Snapshot taken
[ ] Process Monitor running
[ ] Process Hacker running
[ ] Wireshark capturing
[ ] FakeNet-NG / INetSim running
[ ] Regshot before/after
[ ] API Monitor attached
[ ] x64dbg / WinDbg attached (if needed)
```

### 4.2 Sandboxes (Automated)

**Cuckoo Sandbox (self-hosted):**
```bash
sudo apt install cuckoo
cuckoo init
cuckoo web runserver
cuckoo submit suspicious.exe
cuckoo submit --timeout 120 --memory 2048 suspicious.exe
```

**CAPE (Cuckoo fork, malware-focused):**
```bash
git clone https://github.com/kevoreilly/CAPEv2
cd CAPEv2
./installer.sh
python3 cuckoo.py
python3 utils/submit.py suspicious.exe
```

**Online (careful with sensitive samples):**
- https://any.run
- https://app.joesecurity.org
- https://hybrid-analysis.com
- https://www.virustotal.com (behavior tab)
- https://tria.ge

### 4.3 Debuggers

**GDB (Linux):**
```bash
gdb -q ./binary
(gdb) set disassembly-flavor intel
(gdb) info functions
(gdb) break *0x4005a0
(gdb) break main
(gdb) run
(gdb) x/20i $rip
(gdb) x/32wx $rsp
(gdb) x/s 0x7fffffffe220
(gdb) info registers
(gdb) bt
(gdb) stepi
(gdb) nexti
(gdb) continue
(gdb) set follow-fork-mode child
(gdb) set pagination off
(gdb) set logging on
```

**pwndbg / GEF / PEDA (GDB extensions):**
```bash
# pwndbg
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# Inside gdb
context          # registers / stack / asm / source
vmmap            # memory map
got              # GOT entries
checksec         # mitigations
heap             # heap view
bin              # bins view
rop              # search ROP gadgets
```

**x64dbg (Windows):**
```
Workflow:
  1. Open EXE
  2. Search → Strings
  3. Breakpoints on imports (recv, VirtualAlloc, CreateFileW)
  4. F9 → run
  5. Inspect stack/heap/registers
  6. F7 → step into
  7. F8 → step over
  8. F4 → run to cursor

Plugins:
  - Scylla           → IAT reconstruction, unpacking
  - xAnalyzer        → advanced analysis
  - ScyllaHide       → anti-anti-debug
  - TitanHide        → kernel-level hiding
  - SharpOD          → anti-debug bypass
  - ret-sync         → sync with IDA/Ghidra
```

**WinDbg:**
```windbg
!analyze -v         # crash analysis
lm                  # loaded modules
lm m kernel32       # specific module
k                   # stack
kb                  # stack with args
dps rsp             # dump stack
u rip               # disassemble
bp kernel32!CreateFileW
ba r1 0x12345678    # hardware breakpoint on read
dt _PEB             # data type
!peb                # PEB
!teb                # TEB
!handle             # handles
!process 0 0        # list processes
```

### 4.4 Tracing

**strace (Linux syscalls):**
```bash
strace ./binary
strace -f ./binary                          # follow forks
strace -e trace=open,read,write ./binary    # specific calls
strace -e trace=network ./binary            # network only
strace -e trace=file ./binary               # file ops
strace -o trace.log ./binary                # output to file
strace -p PID                               # attach to process
strace -c ./binary                          # summary
```

**ltrace (Linux library calls):**
```bash
ltrace ./binary
ltrace -e "printf+scanf" ./binary
ltrace -o ltrace.log ./binary
ltrace -p PID
```

**ftrace (function tracer):**
```bash
ftrace ./binary
ftrace -e "printf" ./binary
```

**Procmon (Windows, sysinternals):**
```
Filters:
  Process Name is suspicious.exe
  Operation is RegSetValue, CreateFile, WriteFile, TCP Connect
Highlighted events:
  - Registry modifications
  - File creation
  - Network connections
  - Process creation
Save to CSV or PML for offline analysis
```

**Process Hacker (Windows):**
```
- Live view of processes, threads, handles, memory
- Right-click → Properties for details
- Modules tab → loaded DLLs
- Memory tab → injected regions
- Network tab → active connections
```

### 4.5 Dynamic Instrumentation

**Frida (cross-platform):**
```bash
pip install frida-tools
npm install -g frida

# List processes
frida-ps
frida-ps -U

# Attach
frida -U -f com.app.name            # spawn + attach
frida -U -n process_name            # attach by name
frida -U -p PID                     # attach by PID

# Basic script
frida -U -f target.exe -l hook.js --no-pause

# hook.js
Interceptor.attach(Module.findExportByName(null, "CreateFileW"), {
  onEnter: function (args) {
    console.log("CreateFileW called with: " + args[0].readUtf16String());
  },
  onLeave: function (retval) {
    console.log("Return: " + retval);
  }
});
```

**Frida for Android:**
```bash
adb shell
frida-ps -U
frida -U -f com.target.app -l bypass.js --no-pause

# SSL pinning bypass
# https://github.com/httptoolkit/frida-android-unpinning
```

**Frida for iOS:**
```bash
frida-ps -Ua
frida -U -f com.target.app -l bypass.js
# Requires jailbroken device or frida-server
```

**API Monitor (Windows):**
```
- Attach to process
- Monitor Windows API calls in real time
- Filter by category (filesystem, network, registry, crypto)
- Export to file for offline analysis
```

### 4.6 Network Analysis

**Wireshark:**
```bash
sudo wireshark
# Filters:
#   ip.addr == 1.2.3.4
#   tcp.port == 443
#   http.request
#   dns.qry.name contains "evil"
#   tls.handshake.extensions_server_name
```

**tcpdump:**
```bash
sudo tcpdump -i any -w capture.pcap
sudo tcpdump -i any port 80 -A
```

**FakeNet-NG (Windows, INetSim alternative):**
```bash
# https://github.com/fireeye/flare-fakenet-ng
fakenet.exe
# Simulates DNS, HTTP, HTTPS, SMTP, FTP to capture C2
```

**INetSim (Linux):**
```bash
sudo apt install inetsim
sudo inetsim
# Configure VM to use INetSim as DNS/gateway
```

**mitmproxy (intercept HTTPS):**
```bash
mitmproxy -p 8080
# Install CA on target VM
```

### 4.7 Memory Forensics

**Volatility 3:**
```bash
pip install volatility3

# Identify image
vol -f memory.dump windows.info

# List processes
vol -f memory.dump windows.pslist
vol -f memory.dump windows.pstree
vol -f memory.dump windows.psscan

# Network
vol -f memory.dump windows.netscan
vol -f memory.dump windows.netstat

# Files
vol -f memory.dump windows.filescan
vol -f memory.dump windows.dumpfiles --virtaddr 0x...

# Registry
vol -f memory.dump windows.registry.hivelist
vol -f memory.dump windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"

# Malware detection
vol -f memory.dump windows.malfind
vol -f memory.dump windows.hollowfind
vol -f memory.dump windows.cmdline
vol -f memory.dump windows.dlllist

# Dump suspicious process
vol -f memory.dump windows.dumpfiles --pid 1234
```

**PE-sieve / hollows_hunter (live memory):**
```bash
pe-sieve.exe /pid 1234
pe-sieve.exe /pname suspicious.exe
pe-sieve.exe /pname suspicious.exe /shellc /dir dumps
hollows_hunter.exe
```

---

## 5. EMULATION (When Binary Won't Run)

**Qiling (full-system emulation):**
```bash
pip install qiling
qltool run -f binary.exe --rootfs rootfs/x86_windows
```

**Unicorn (CPU emulation):**
```python
from unicorn import *
from unicorn.x86_const import *

mu = Uc(UC_ARCH_X86, UC_MODE_32)
mu.mem_map(0x1000, 0x1000)
mu.mem_write(0x1000, b"\x90\x90")   # NOP NOP
mu.emu_start(0x1000, 0x1002)
```

**Speakeasy (FireEye, malware emulation):**
```bash
speakeasy -t suspicious.exe -o report.json
speakeasy -t suspicious.exe -r    # recursive
```

**QEMU (user-mode, cross-arch):**
```bash
qemu-arm -L /usr/arm-linux-gnueabi ./binary
qemu-arm -g 1234 ./binary       # gdb stub
gdb-multiarch ./binary
(gdb) target remote :1234
```

---

## 6. ANTI-ANALYSIS BYPASSES

### 6.1 Anti-Debug Checks

| Check | Bypass |
|-------|--------|
| `IsDebuggerPresent` | Patch return value to 0 |
| `CheckRemoteDebuggerPresent` | Patch to return FALSE |
| `NtQueryInformationProcess` | Hook and modify |
| `PEB.BeingDebugged` | Set to 0 in x64dbg |
| `RDTSC` timing checks | NOP the timing code |
| `INT 3` traps | Patch or handle in debugger |
| Debugger windows (`FindWindow`) | Rename debugger window or hook |
| Parent process check | Rename debugger or use ScyllaHide |

**ScyllaHide** automates all of these in x64dbg.

### 6.2 Anti-VM Checks

| Check | Bypass |
|-------|--------|
| Registry keys (`VMware`, `VBox`) | Patch or hide |
| MAC address OUI | Spoof |
| CPUID hypervisor bit | Patch in VM settings |
| Disk/file names | Rename or use custom VM |
| Timing attacks | Patch or use bare metal |
| Hardware IDs | Patch or use hardware passthrough |

### 6.3 Anti-Disassembly

| Technique | Bypass |
|-----------|--------|
| Opaque predicates | Patch or use symbolic execution |
| Junk bytes / overlapping instructions | Realign in disassembler |
| Control flow flattening | Use D810 or similar deobfuscators |
| JMP obfuscation | Trace manually or use Triton |
| VM-based obfuscation | Use devirtualizers (VMHunt, etc.) |

### 6.4 Anti-Dump

| Technique | Bypass |
|-----------|--------|
| Section encryption | Dump after OEP |
| Runtime unpacking | Breakpoint on VirtualAlloc + dump |
| Anti-dump flags | Use Scylla with anti-dump patch |
| Memory wiping | Dump fast, use snapshots |

---

## 7. BINARY DIFFING

**BinDiff (commercial, free for research):**
```bash
bindiff primary.BinExport secondary.BinExport
# Generates similarity graph in IDA
```

**Diaphora (free IDA plugin):**
```bash
# https://github.com/joxeankoret/diaphora
# In IDA: File → Script file → diaphora.py
```

**Ghidra Version Tracking:**
```
Tools → Version Tracking
Add source and destination binaries → correlate
```

---

## 8. PATCHING

**patchelf (ELF):**
```bash
patchelf --set-interpreter /lib/ld-linux.so.2 ./binary
patchelf --set-rpath /custom/lib ./binary
patchelf --replace-needed libc.so.6 libc_custom.so ./binary
patchelf --print-interpreter ./binary
```

**Hex editors:**
- **HxD** (Windows)
- **010 Editor** (Windows, commercial)
- **ImHex** (cross-platform, free)
- **xxd** (CLI)

```bash
# Patch single byte
printf '\x90' | dd of=./binary bs=1 seek=0x1234 count=1 conv=notrunc
```

**Binary Ninja patching:**
```python
bv = binaryninja.load("/path/to/binary")
bv.write(0x1234, b"\x90\x90")
bv.save("patched")
```

---

## 9. FIRMWARE / EMBEDDED RE

**Extraction:**
```bash
binwalk firmware.bin
binwalk -e firmware.bin
binwalk -Me firmware.bin

unsquashfs filesystem.squashfs
jefferson filesystem.jffs2 -d output/
ubireader_extract_images firmware.ubi
ubireader_extract_files -o output/ firmware.ubi
```

**Emulation with FAT:**
```bash
git clone https://github.com/attify/firmware-analysis-toolkit
./fat.py firmware.bin
```

**QEMU for firmware:**
```bash
qemu-system-mips -M malta -kernel vmlinux -hda rootfs.img -append "root=/dev/sda console=ttyS0" -nographic
```

---

## 10. AUTOMATION

**IDAPython:**
```python
# Rename all functions
import idautils
for func_ea in idautils.Functions():
    name = idc.get_func_name(func_ea)
    if name.startswith("sub_"):
        idc.set_name(func_ea, f"myfunc_{func_ea:x}")
```

**Ghidra Scripting:**
```python
# Print all function names
from ghidra.program.model.listing import Function
fm = currentProgram.getFunctionManager()
for func in fm.getFunctions(True):
    print(func.getName())
```

**r2pipe (Python):**
```python
import r2pipe
r2 = r2pipe.open("./binary")
r2.cmd("aaa")
print(r2.cmd("afl"))
r2.quit()
```

**Binary Ninja API:**
```python
import binaryninja
bv = binaryninja.open_view("./binary")
for func in bv.functions:
    print(hex(func.start), func.name)
```

---

## 11. MALWARE ANALYSIS TOOLKITS

**FLARE-VM (Windows all-in-one):**
```bash
# https://github.com/mandiant/flare-vm
# Installs IDA Free, Ghidra, x64dbg, PE-bear, PEStudio, CFF Explorer,
# Process Hacker, Procmon, Wireshark, FakeNet-NG, FLOSS, capa,
# PE-sieve, hollows_hunter, and more
```

**REMnux (Linux malware analysis):**
```bash
# https://remnux.org
# Pre-configured Linux distribution with RE tools
```

---

## 12. QUICK REFERENCE BY TASK

| Task | Tool |
|------|------|
| Identify file | `file`, `binwalk`, `DIE` |
| Extract strings | `strings`, `FLOSS` |
| Disassemble | `objdump`, `radare2`, `Ghidra`, `IDA` |
| Decompile | `Ghidra`, `IDA Pro`, `Binary Ninja`, `dnSpy` |
| Debug (Linux) | `gdb` + `pwndbg`, `radare2` |
| Debug (Windows) | `x64dbg`, `WinDbg` |
| Trace syscalls | `strace` |
| Trace libcalls | `ltrace` |
| Monitor Windows APIs | Procmon, API Monitor |
| Dynamic instrument | `Frida` |
| Sandbox | Cuckoo, CAPE, ANY.RUN |
| Unpack | `upx`, `Scylla`, `PE-sieve` |
| Detect capabilities | `capa` |
| Detect malware | `YARA` |
| Diff binaries | `BinDiff`, `Diaphora` |
| Patch binaries | `patchelf`, hex editor |
| Extract firmware | `binwalk`, `unsquashfs`, `ubireader` |
| Emulate firmware | `FAT`, `QEMU`, `Qiling` |
| Memory forensics | `Volatility` |
| Network analysis | Wireshark, FakeNet-NG, INetSim |

---

## 13. TIPS

1. **Static first, dynamic second, back to static.**
2. **Strings are gold** — start with `strings` and `FLOSS`.
3. **Imports reveal intent** — `CreateRemoteThread`, `VirtualAlloc`, `InternetOpen`.
4. **Xrefs are essential** — find who calls a function to understand data flow.
5. **Rename everything** — `sub_401000` is useless, `decrypt_config` is not.
6. **Snapshot before running malware** — always.
7. **FakeNet-NG over real internet** — never let malware reach your real network.
8. **capa for triage** — automatically maps to MITRE ATT&CK.
9. **ScyllaHide for anti-debug** — saves hours.
10. **Frida for Android/iOS** — best dynamic tool for mobile.
11. **Volatility for memory** — catches things disk doesn't.
12. **Qiling for evasive malware** — when VM detection blocks execution.
13. **Combine tools** — no single tool is complete.
14. **Always test on your own binaries or authorized targets.**
