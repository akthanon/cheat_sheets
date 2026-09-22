# LINUX SYSTEM & DESKTOP TUNING CHEAT SHEET

## 1. WHAT IS THIS?

This is a **system administration / tuning** cheat sheet for a Linux workstation used in pentesting. Topics:

- **Power management** (TLP, powertop)
- **CPU/GPU tuning** (cpupower, intel-undervolt, envycontrol)
- **Persistent services** (systemd)
- **Shell customization** (bashrc)
- **Desktop tweaks** (fastfetch, KDE reset)

**Note:** This is not offensive security content — it's for keeping your attack machine lean and personalized.

---

## 2. SHELL CUSTOMIZATION (BASHRC)

### 2.1 Useful .bashrc Snippets

```bash
# Only run for interactive shells
[[ $- != *i* ]] && return

# Colorized aliases
alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias ll='ls -lah'
alias la='ls -A'

# Nice prompt
PS1='\[\e[1;32m\]\u@\h:\w\$\[\e[0m\] '

# Custom PATH additions
export PATH="$HOME/.local/bin:$PATH"

# Load SSH agent
eval "$(ssh-agent -s)" >/dev/null 2>&1
```

### 2.2 Reload Bashrc

```bash
source ~/.bashrc
```

---

## 3. FASTFETCH + IMAGE CONVERSION

### 3.1 Convert PNG to Fastfetch BIN (kitty icat)

```bash
kitten icat -n --align=left --transfer-mode=stream image.png > image.bin
cat image.bin && echo END
```

### 3.2 Load in .bashrc

```bash
fastfetch --raw ~/Imágenes/FASTFETCHBIN/RANDOM.png.bin
```

### 3.3 Batch Convert PNG → BIN (Bash Script)

```bash
#!/bin/bash
SRC_DIR="$HOME/Imágenes/FASTFETCH"
DST_DIR="$HOME/Imágenes/FASTFETCHBIN"
TEMP_DIR="$HOME/Imágenes/FASTFETCH_TEMP"
FIXED_SIZE="512x512"

mkdir -p "$DST_DIR" "$TEMP_DIR"

for img in "$SRC_DIR"/*.png; do
    base_name=$(basename "$img" .png)
    temp_file="$TEMP_DIR/$base_name.png"
    dst_file="$DST_DIR/$base_name.bin"

    convert "$img" -resize ${FIXED_SIZE}\> -background none -gravity center -extent $FIXED_SIZE "$temp_file"
    kitten icat -n --align=left --transfer-mode=stream "$temp_file" > "$dst_file"

    [[ -s "$dst_file" ]] && echo "✅ $img -> $dst_file" || echo "❌ $img"
done

rm -rf "$TEMP_DIR"
```

### 3.4 Random Image on Shell Start

```bash
BIN_DIR="$HOME/Imágenes/FASTFETCHBIN"
shopt -s nullglob
bins=("$BIN_DIR"/*.bin)
shopt -u nullglob

if [ ${#bins[@]} -gt 0 ]; then
    random_bin="${bins[RANDOM % ${#bins[@]}]}"
    fastfetch --raw "$random_bin"
fi
```

---

## 4. SYSTEMD SERVICE CREATION

### 4.1 Create a Service File

```bash
sudo nano /etc/systemd/system/my_service.service
```

```ini
[Unit]
Description=My service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /path/to/script.py
Restart=always
User=my_user
WorkingDirectory=/path/to/

[Install]
WantedBy=multi-user.target
```

### 4.2 Enable and Start

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable my_service
sudo systemctl start my_service
sudo systemctl status my_service
```

### 4.3 Manage

```bash
sudo systemctl restart my_service
sudo systemctl stop my_service
sudo systemctl disable my_service
journalctl -u my_service -f
```

---

## 5. POWER MANAGEMENT (TLP)

### 5.1 Install TLP

```bash
sudo pacman -S tlp tlp-rdw
# or
sudo apt install tlp tlp-rdw
```

### 5.2 Configure /etc/tlp.conf

```ini
# ---- CPU ----
CPU_SCALING_GOVERNOR_ON_AC=schedutil
CPU_SCALING_GOVERNOR_ON_BAT=powersave

CPU_SCALING_MAX_FREQ_ON_BAT=1800000
CPU_SCALING_MIN_FREQ_ON_BAT=400000

CPU_BOOST_ON_AC=1
CPU_BOOST_ON_BAT=0

# ---- PLATFORM PROFILE ----
PLATFORM_PROFILE_ON_AC=performance
PLATFORM_PROFILE_ON_BAT=low-power

# ---- WIFI / BLUETOOTH ----
WIFI_PWR_ON_BAT=on
DEVICES_TO_DISABLE_ON_BAT="bluetooth"

# ---- USB ----
USB_AUTOSUSPEND=1
USB_DENYLIST="046d:c534 1a2c:2d23"

# ---- PCIe ----
PCIE_ASPM_ON_AC=default
PCIE_ASPM_ON_BAT=powersupersave

# ---- SATA ----
SATA_LINKPWR_ON_AC=max_performance
SATA_LINKPWR_ON_BAT=med_power_with_dipm

# ---- AUDIO ----
SOUND_POWER_SAVE_ON_BAT=1
SOUND_POWER_SAVE_CONTROLLER=Y

# ---- RUNTIME PM ----
RUNTIME_PM_ON_AC=on
RUNTIME_PM_ON_BAT=auto

# ---- GPU ----
RADEON_DPM_STATE_ON_BAT=battery
RADEON_DPM_PERF_LEVEL_ON_BAT=low

# ---- DISK ----
DISK_IDLE_SECS_ON_BAT=2
```

### 5.3 Apply and Check

```bash
sudo tlp start
sudo tlp-stat -s
sudo tlp-stat -b
sudo tlp-stat -p
```

---

## 6. CPU / GPU TUNING

### 6.1 Powertop

```bash
sudo powertop
sudo powertop --auto-tune
```

### 6.2 Intel Undervolt

```bash
sudo intel-undervolt apply
sudo intel-undervolt measure
sudo nano /etc/intel-undervolt.conf
```

### 6.3 Envycontrol (NVIDIA Optimus)

```bash
sudo envycontrol -s integrated
sudo envycontrol -s hybrid
sudo envycontrol -s nvidia
```

### 6.4 CPU Frequency Scaling

```bash
sudo cpupower frequency-set -g powersave
sudo cpupower frequency-set -g performance
sudo cpupower frequency-info
```

### 6.5 NVIDIA Settings

```bash
nvidia-settings
nvidia-smi
watch -n 1 nvidia-smi
```

---

## 7. HARD DRIVE MOUNT IN FSTAB

```bash
sudo nano /etc/fstab
```

```ini
/dev/sda2   /mnt/hdd1   ntfs3   rw,uid=1000,gid=1000,umask=022,noatime,nofail   0 0
```

- `uid=1000,gid=1000` → your user owns the drive
- `umask=022` → permissive for user, restrictive for others
- `noatime` → don't update access times (faster)
- `nofail` → don't block boot if drive is missing

---

## 8. KDE / DESKTOP RESET

### 8.1 Reset Dolphin

```bash
mv ~/.config/dolphinrc ~/.config/dolphinrc.bak
mv ~/.local/share/dolphin ~/.local/share/dolphin.bak
```

### 8.2 SDDM Themes

```bash
ls /usr/share/sddm/themes
sudo nano /etc/sddm.conf.d/kde_settings.conf
cp /etc/sddm.conf.d/kde_settings.conf /etc/sddm.conf
```

---

## 9. MISC

### 9.1 Mapas en la Terminal

```bash
telnet mapscii.me
```

### 9.2 WiFi Hotspot

```bash
nmcli device wifi hotspot ifname wlan0 ssid RedEnrisexo password chocolate
sudo sysctl -w net.ipv4.ip_forward=1
```

### 9.3 Fonts (Noto)

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
```

### 9.4 Env Variables

```bash
export X-Forwarded-For=10.10.10.10
$IFS
```
