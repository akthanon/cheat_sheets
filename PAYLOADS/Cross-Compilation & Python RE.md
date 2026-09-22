# CROSS-COMPILATION & PYTHON REVERSING CHEAT SHEET

## 1. WHAT IS CROSS-COMPILATION / PYTHON RE?

- **Cross-compilation:** Build binaries for another architecture (e.g., Windows EXE on Linux, ARM on x86_64).
- **Python RE:** Extract and decompile PyInstaller-packaged executables (`.exe` → `.pyc` → `.py`).

**Use cases:**
- Building Windows payloads from Linux
- Running x86_64 Windows binaries on ARM Linux
- Reversing Python-based malware / tools

---

## 2. WINE — RUN WINDOWS BINARIES ON LINUX

### 2.1 Install Wine

```bash
# Arch
sudo pacman -S wine wine-mono wine-gecko winetricks

# Debian/Ubuntu (64-bit only)
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine64
sudo apt install wine32:i386
sudo apt install libc6:i386
```

### 2.2 Run a Windows Binary

```bash
wine program.exe
wine64 program.exe
WINEDEBUG=+all wine program.exe 2> wine.log
```

---

## 3. BOX64 — RUN x86_64 BINARIES ON ARM

**Use case:** You're on ARM (Raspberry Pi, Orange Pi, etc.) and want to run x86_64 Windows/Linux binaries.

### 3.1 Install box64

```bash
git clone https://github.com/ptitSeb/box64
cd box64
mkdir build
cd build
cmake .. -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
sudo make install
```

### 3.2 Run x86_64 Windows Binary via box64 + wine

```bash
box64 ./wine64/bin/wine64 program.exe
```

---

## 4. CROSS-COMPILATION WITH MINGW

### 4.1 Install MinGW

```bash
# Arch
sudo pacman -S mingw-w64-gcc

# Debian/Ubuntu
sudo apt install gcc-mingw-w64 g++-mingw-w64
```

### 4.2 Compile C to Windows EXE

```bash
# 32-bit
i686-w64-mingw32-gcc code.c -o program.exe

# 64-bit
x86_64-w64-mingw32-gcc code.c -o program.exe

# Static (recommended)
x86_64-w64-mingw32-gcc -static -O2 code.c -o program.exe

# With strip (smaller)
x86_64-w64-mingw32-strip program.exe
```

### 4.3 Compile for Both Linux and Windows

```bash
# Linux
gcc functions.c -O2 -o functions_linux

# Windows (via MinGW)
x86_64-w64-mingw32-gcc functions.c -O2 -o functions.exe

# Windows (via MinGW static)
x86_64-w64-mingw32-gcc -static functions.c -O2 -o functions.exe
```

---

## 5. PYINSTALLER EXTRACTION

### 5.1 Extract PyInstaller EXE

```bash
wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/refs/heads/master/pyinstxtractor.py
python pyinstxtractor.py program.exe
```

**Output:** `program.exe_extracted/` — contains `.pyc` files and bundled resources.

---

## 6. PYCDC — DECOMPILE .pyc TO .py

### 6.1 Install pycdc

```bash
git clone https://github.com/zrax/pycdc.git
cd pycdc
cmake .
make
```

### 6.2 Decompile

```bash
./pycdc program.pyc > program.py
```

### 6.3 Tips

- The main script is usually the one matching the EXE name.
- Other `.pyc` files are imported modules — decompile them for full context.
- Some Python versions may not decompile cleanly (2.x vs 3.x, 3.7 vs 3.11).
- If pycdc fails, try **decompyle3**, **uncompyle6**, or **pycdas** (disassembler).

**Alternative decompilers:**
```bash
pip install uncompyle6
uncompyle6 program.pyc > program.py

pip install decompyle3
decompyle3 program.pyc > program.py
```

---

## 7. TIPS

1. Use `box64 + wine` combo when you're on ARM and need to run Windows tools.
2. Always use `-static` with MinGW to avoid missing DLL issues.
3. `pyinstxtractor` works on both EXE (Windows) and ELF (Linux) PyInstaller binaries.
4. Rename the extracted main `.pyc` if it lacks the magic number header.
5. Try multiple Python decompilers — each handles a different version range.
6. Keep a copy of the original `.pyc` before decompiling — decompilation is non-destructive but reversible sometimes.
7. Test your Windows payloads in a VM before deployment.
8. Use `WINEDEBUG=+all` to debug why a Windows binary fails under Wine.

---

## FAST COMMANDS

```
sudo pacman -S wine wine-mono wine-gecko winetricks
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine64
sudo apt install wine32:i386
sudo apt install libc6:i386
wine program.exe
wine64 program.exe
WINEDEBUG=+all wine program.exe 2> wine.log
git clone https://github.com/ptitSeb/box64
cd box64
mkdir build
cd build
cmake .. -DARM_DYNAREC=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
sudo make install
box64 ./wine64/bin/wine64 program.exe
sudo pacman -S mingw-w64-gcc
sudo apt install gcc-mingw-w64 g++-mingw-w64
i686-w64-mingw32-gcc code.c -o program.exe
x86_64-w64-mingw32-gcc code.c -o program.exe
x86_64-w64-mingw32-gcc -static -O2 code.c -o program.exe
x86_64-w64-mingw32-strip program.exe
gcc functions.c -O2 -o functions_linux
x86_64-w64-mingw32-gcc functions.c -O2 -o functions.exe
x86_64-w64-mingw32-gcc -static functions.c -O2 -o functions.exe
wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/refs/heads/master/pyinstxtractor.py
python pyinstxtractor.py program.exe
git clone https://github.com/zrax/pycdc.git
cd pycdc
cmake .
make
./pycdc program.pyc > program.py
pip install uncompyle6
uncompyle6 program.pyc > program.py
pip install decompyle3
decompyle3 program.pyc > program.py
```
