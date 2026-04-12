# Challenge 28: System Information in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Knowing how to gather **system information** is fundamental to Linux administration. Whether you are onboarding to a new server, troubleshooting a performance issue, preparing a system inventory, or documenting infrastructure — understanding your hardware, OS version, kernel, CPU, and memory is essential.

Linux provides many built-in commands to inspect every layer of the system — from the kernel version to CPU architecture, installed RAM, and distribution details.

---

## 🧩 Task

Display comprehensive information about the Linux system — OS version, kernel, hardware, and resources.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Kernel and OS information
uname -a

# Distribution details
cat /etc/os-release

# CPU information
lscpu

# Memory information
free -h

# All hardware info
lshw -short
```

### How it works

| Command | Description |
|---------|-------------|
| `uname -a` | Print **all** system/kernel information |
| `cat /etc/os-release` | Show Linux **distribution** details |
| `lscpu` | Display **CPU** architecture information |
| `free -h` | Show **memory** and swap usage |
| `lshw -short` | List **all hardware** in short format |

---

## 🔢 Understanding `uname -a` Output

```bash
uname -a
```

```
Linux hostname 5.15.0-91-generic #101-Ubuntu SMP Tue Nov 14 13:30:08 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

| Field | Example | Description |
|-------|---------|-------------|
| OS | `Linux` | Operating system name |
| Hostname | `hostname` | Computer's network name |
| Kernel version | `5.15.0-91-generic` | Running kernel version |
| Build info | `#101-Ubuntu SMP ...` | Kernel build number and date |
| Architecture | `x86_64` | CPU architecture |
| OS type | `GNU/Linux` | Operating system type |

---

## 📖 Extended Examples

### Example 1 — Kernel Information

```bash
# Full kernel information
uname -a

# Just the kernel version
uname -r

# Just the hostname
uname -n

# Just the architecture
uname -m

# Just the OS name
uname -s
```

```bash
# uname -r output:
5.15.0-91-generic

# uname -m output:
x86_64
```

| `uname` Flag | Description |
|--------------|-------------|
| `-a` | All information |
| `-r` | Kernel **release** (version) |
| `-n` | **Node** name (hostname) |
| `-m` | **Machine** hardware (CPU arch) |
| `-s` | **System** OS name |
| `-v` | Kernel **version** (build info) |
| `-p` | **Processor** type |
| `-o` | **Operating system** |

---

### Example 2 — Distribution Details

```bash
cat /etc/os-release
```

```bash
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 22.04.3 LTS"
VERSION_ID="22.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
```

```bash
# Extract just the distribution name and version
grep -E "^NAME|^VERSION=" /etc/os-release

# Using lsb_release (if available)
lsb_release -a
```

```bash
# lsb_release -a output:
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy
```

---

### Example 3 — CPU Information

```bash
lscpu
```

```bash
Architecture:            x86_64
CPU op-mode(s):          32-bit, 64-bit
Byte Order:              Little Endian
CPU(s):                  4
On-line CPU(s) list:     0-3
Thread(s) per core:      2
Core(s) per socket:      2
Socket(s):               1
Vendor ID:               GenuineIntel
Model name:              Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
CPU MHz:                 1600.000
CPU max MHz:             3400.0000
L1d cache:               128 KiB
L2 cache:                512 KiB
L3 cache:                6 MiB
```

```bash
# Quick CPU summary
lscpu | grep -E "^CPU\(s\)|^Model name|^Architecture|^CPU MHz"

# Number of CPU cores
nproc

# Detailed CPU info from /proc
cat /proc/cpuinfo | grep "model name" | head -1
cat /proc/cpuinfo | grep "cpu cores" | head -1
```

---

### Example 4 — Memory Information

```bash
# Human-readable memory overview
free -h

# Detailed memory info
cat /proc/meminfo | head -20
```

```bash
# free -h output:
              total        used        free      shared  buff/cache   available
Mem:           15Gi        9.1Gi       1.2Gi       312Mi       4.7Gi       5.8Gi
Swap:           2Gi          0Gi        2Gi

# Key /proc/meminfo fields:
MemTotal:       16199680 kB
MemFree:         1228800 kB
MemAvailable:    5939200 kB
SwapTotal:       2097152 kB
SwapFree:        2097152 kB
```

---

### Example 5 — Hostname Information

```bash
# Show the hostname
hostname

# Show the Fully Qualified Domain Name
hostname -f

# Show all addresses for the hostname
hostname -I

# Detailed hostname info with systemd
hostnamectl
```

```bash
# hostnamectl output:
   Static hostname: myserver
         Icon name: computer-server
           Chassis: server
        Machine ID: abc123def456...
           Boot ID: xyz789...
  Operating System: Ubuntu 22.04.3 LTS
            Kernel: Linux 5.15.0-91-generic
      Architecture: x86-64
```

---

### Example 6 — Disk Information

```bash
# Disk partitions and usage
df -h

# List block devices (disks and partitions)
lsblk

# Detailed disk info
sudo fdisk -l

# Disk hardware info
sudo hdparm -i /dev/sda
```

```bash
# lsblk output:
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   50G  0 disk
├─sda1   8:1    0   49G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0    1G  0 part [SWAP]
sdb      8:16   0  100G  0 disk /data
```

---

### Example 7 — Network Information

```bash
# Show all network interfaces and IPs
ip addr show

# Quick IP addresses only
ip addr | grep "inet " | awk '{print $2}'

# Show network interfaces
ip link show

# Show routing table
ip route show

# Show hostname and DNS
cat /etc/resolv.conf
```

---

### Example 8 — All Hardware with `lshw`

```bash
# Short hardware summary
sudo lshw -short

# Full hardware details
sudo lshw

# HTML report
sudo lshw -html > hardware_report.html

# Specific class only
sudo lshw -class cpu
sudo lshw -class memory
sudo lshw -class network
sudo lshw -class disk
```

```bash
# lshw -short output:
H/W path        Device    Class      Description
===============================================================
                          system     VMware Virtual Platform
/0                        bus        440BX Desktop Reference...
/0/0                      memory     128KiB BIOS
/0/1                      memory     15GiB System Memory
/0/1/0                    memory     15GiB DIMM RAM
/0/100                    bridge     440BX/ZX/DX - 82443BX/ZX...
```

---

### Example 9 — System Info with `neofetch` / `fastfetch`

```bash
# Install neofetch for a pretty system summary
sudo apt install neofetch    # Debian/Ubuntu
sudo dnf install neofetch    # RHEL

# Display system info with ASCII art logo
neofetch
```

```bash
# neofetch output example:
            .-/+oossssoo+/-.               devuser@myserver
        `:+ssssssssssssssssss+:`           ─────────────────
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 22.04.3 LTS x86_64
    .ossssssssssssssssssdMMMNysssso.       Kernel: 5.15.0-91-generic
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Uptime: 5 days, 3 hours, 14 mins
  +ssssssssshmydMMMMMMMNddddyssssssss+     Packages: 1432 (dpkg)
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: bash 5.1.16
```

---

### Example 10 — Complete System Info Script

```bash
#!/bin/bash

echo "╔══════════════════════════════════════════╗"
echo "║         SYSTEM INFORMATION REPORT        ║"
echo "╚══════════════════════════════════════════╝"
echo ""

echo "── OS & Kernel ──────────────────────────────"
echo "OS:       $(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)"
echo "Kernel:   $(uname -r)"
echo "Arch:     $(uname -m)"
echo "Hostname: $(hostname)"
echo ""

echo "── CPU ──────────────────────────────────────"
echo "Model:    $(lscpu | grep 'Model name' | awk -F': ' '{print $2}' | xargs)"
echo "Cores:    $(nproc)"
echo "Sockets:  $(lscpu | grep 'Socket(s)' | awk '{print $2}')"
echo ""

echo "── Memory ───────────────────────────────────"
free -h | awk 'NR==2{printf "RAM:      %s total, %s used, %s available\n", $2, $3, $7}'
free -h | awk 'NR==3{printf "Swap:     %s total, %s used\n", $2, $3}'
echo ""

echo "── Disk ─────────────────────────────────────"
df -h | awk 'NR>1 && $1 !~ /tmpfs/ {printf "%-20s %s used of %s (%s)\n", $6, $3, $2, $5}'
echo ""

echo "── Network ──────────────────────────────────"
ip addr | awk '/inet / {print "IP:", $2}' | head -5
echo ""

echo "── Uptime ───────────────────────────────────"
uptime -p
echo "Boot time: $(uptime -s)"
```

```bash
# Output:
╔══════════════════════════════════════════╗
║         SYSTEM INFORMATION REPORT        ║
╚══════════════════════════════════════════╝

── OS & Kernel ──────────────────────────────
OS:       Ubuntu 22.04.3 LTS
Kernel:   5.15.0-91-generic
Arch:     x86_64
Hostname: myserver

── CPU ──────────────────────────────────────
Model:    Intel(R) Core(TM) i5-8250U @ 1.60GHz
Cores:    4
Sockets:  1
...
```

---

## 🗂️ System Info Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `uname -a` | Kernel and OS info |
| `uname -r` | Kernel version only |
| `hostname` | System hostname |
| `hostnamectl` | Full hostname and OS info |
| `cat /etc/os-release` | Distribution details |
| `lsb_release -a` | Distribution info (if available) |
| `lscpu` | CPU architecture details |
| `nproc` | Number of CPU cores |
| `free -h` | Memory and swap |
| `lsblk` | Block device layout |
| `df -h` | Disk usage |
| `ip addr` | Network interfaces and IPs |
| `lshw -short` | All hardware summary |
| `dmidecode` | BIOS/hardware info (root) |
| `neofetch` | Pretty system overview |

---

## 🚀 Full Workflow — Document a New Server

```bash
# Step 1: OS and kernel
uname -a
cat /etc/os-release

# Step 2: Hostname
hostnamectl

# Step 3: CPU
lscpu | grep -E "^CPU\(s\)|^Model name|^Architecture"
nproc

# Step 4: Memory
free -h

# Step 5: Disk layout
lsblk
df -h

# Step 6: Network
ip addr show
ip route show

# Step 7: All hardware (requires root)
sudo lshw -short

# Step 8: Save report to file
{
  echo "=== System Report: $(date) ==="
  uname -a
  cat /etc/os-release
  lscpu
  free -h
  lsblk
  df -h
  ip addr
} > system_report_$(date +%Y-%m-%d).txt
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `lshw` requires root | Use `sudo lshw -short` |
| `lsb_release` not installed | Use `cat /etc/os-release` instead |
| `neofetch` not installed | Install with `sudo apt install neofetch` |
| `/etc/os-release` format differs by distro | Parse with `grep` or `source /etc/os-release` |
| `uname -r` shows wrong kernel after update | Reboot is needed for the new kernel to be active |

---

## 🔑 Key Takeaways

- `uname -a` gives a quick overview of the **kernel, architecture, and hostname**.
- `cat /etc/os-release` is the **most reliable way** to get distribution details across all Linux distros.
- `lscpu` shows full **CPU architecture** — cores, threads, speed, cache.
- `free -h` shows **memory and swap** — always check the `available` column, not `free`.
- `lshw -short` (requires sudo) gives a **complete hardware inventory** in one command.
- `hostnamectl` combines hostname, OS, kernel, and architecture in one convenient output.
- Use `neofetch` for a **quick visual summary** — great for screenshots and documentation.

---

## 📚 Related Concepts

- [GNU uname Manual](https://www.gnu.org/software/coreutils/manual/html_node/uname-invocation.html)
- [lshw Documentation](https://ezix.org/project/wiki/HardwareLiSter)

</details>
