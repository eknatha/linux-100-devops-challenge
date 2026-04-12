# Challenge 32: Top Memory Consuming Processes in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-intermediate-yellow) 

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Memory exhaustion is one of the most common causes of system instability, application crashes, and OOM (Out Of Memory) kills. Identifying **which processes are consuming the most RAM** is a critical diagnostic skill — it lets you pinpoint memory leaks, oversized services, and runaway processes before they bring down the system.

Key insight: Linux uses spare RAM as **disk cache** (shown as `buff/cache`) — this is normal and healthy. The real concern is when the `available` RAM is low and **swap is actively being used**.

---

## 🧩 Task

Find which processes are consuming the most memory on the system.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Sorted snapshot — top memory consumers
ps aux --sort=-%mem | head -10

# Custom columns sorted by memory
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10

# Real-time interactive view
top    # then press M to sort by memory
```

### How it works

| Command | Description |
|---------|-------------|
| `ps aux --sort=-%mem` | Snapshot sorted by memory descending |
| `ps -eo pid,user,%mem,rss,cmd` | Custom columns — includes **RSS** (actual RAM) |
| `top` (then `M`) | Real-time interactive view sorted by memory |

---

## 🔢 Understanding Memory Columns in `ps`

```bash
ps aux --sort=-%mem | head -5
```

```
USER       PID  %CPU  %MEM    VSZ      RSS  TTY  STAT START  TIME COMMAND
mysql     2341  14.3  14.2  1892000 2248000  ?   Sl   08:00  5:21 /usr/sbin/mysqld
appuser  12345   5.1  10.5  8912000 1048576  ?   S    10:00  2:34 python3 /opt/app/server.py
www-data  5201   2.1   1.2   456000  196608  ?   S    09:00  0:45 nginx: worker process
```

| Column | Description |
|--------|-------------|
| `%MEM` | **Percentage** of total RAM used |
| `VSZ` | **Virtual Size** — total virtual memory (includes unmapped pages — inflated) |
| `RSS` | **Resident Set Size** — **actual physical RAM** currently in use (KB) |

> 💡 **RSS is the most accurate** measure of real memory usage. **VSZ is inflated** — it includes memory mapped but not loaded into RAM. Always use RSS for actual consumption.

---

## 📖 Extended Examples

### Example 1 — Snapshot Sorted by Memory

```bash
ps aux --sort=-%mem | head -10
```

---

### Example 2 — Custom Columns with RSS in MB

```bash
# Show PID, user, %mem, RSS in MB, and command
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | \
  awk 'NR==1{print} NR>1{printf "%-8s %-12s %-6s %-8s %s\n", $1, $2, $3, int($4/1024)"MB", $5}' | head -10
```

```bash
PID      USER         %MEM   RSS      CMD
2341     mysql        14.2   2195MB   /usr/sbin/mysqld
12345    appuser      10.5   1024MB   python3
5201     www-data      1.2    192MB   nginx: worker process
```

---

### Example 3 — Show Only Processes Above a Memory Threshold

```bash
# Show processes using more than 5% RAM
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | awk 'NR==1 || $3 > 5'
```

```bash
  PID USER     %MEM    RSS CMD
 2341 mysql    14.2  2248000 /usr/sbin/mysqld
12345 appuser  10.5  1048576 python3 /opt/app/server.py
```

---

### Example 4 — Sort by RSS (Absolute RAM, not Percentage)

```bash
# Sort by RSS — most accurate for finding the real memory hog
ps -eo pid,user,rss,cmd --sort=-rss | head -10 | \
  awk 'NR==1{print "PID     USER         RSS_MB   CMD"} NR>1{printf "%-8s %-12s %-8s %s\n", $1, $2, int($3/1024)"MB", $4}'
```

```bash
PID      USER         RSS_MB   CMD
2341     mysql        2195MB   /usr/sbin/mysqld
12345    appuser      1024MB   python3
9001     java          891MB   java -jar myapp.jar
```

---

### Example 5 — Memory in `top` (Real-Time)

```bash
top
```

Press `M` to sort by memory:

```bash
top - 10:35:22 up 3 days, load average: 0.45
Tasks: 215 total,   1 running, 214 sleeping
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 92.8 id
MiB Mem : 15,820.0 total,  1,212.0 free, 13,341.0 used, 1,267.0 buff/cache
MiB Swap:  2,048.0 total,  1,843.0 used,    205.0 free.   421.0 avail Mem

  PID USER    %CPU %MEM    VIRT    RES  COMMAND
 2341 mysql   14.3 14.2 1892000 2248000 mysqld
12345 appuser  5.1 10.5 8912000 1048576 python3
 9001 appuser  2.3  5.6 3145000  886784 java
```

**Key `top` shortcuts for memory investigation:**

| Key | Action |
|-----|--------|
| `M` | Sort by **memory** usage |
| `P` | Sort by **CPU** usage |
| `k` | **Kill** a process |
| `r` | **Renice** — lower priority |
| `q` | **Quit** |

---

### Example 6 — Visual Memory Monitoring with `htop`

```bash
htop
```

Press `M` or `F6 → MEM%` to sort by memory.

`htop` memory bar colors:
- 🟢 **Green** — used memory
- 🔵 **Blue** — buffer memory
- 🟡 **Yellow** — cache memory
- 🔴 **Red** — swap usage

---

### Example 7 — Detailed Process Memory with `pmap`

```bash
# Show detailed memory map of a process
sudo pmap -x 12345
```

```bash
Address     Kbytes     RSS   Dirty Mode  Mapping
00007f8a  2048000  512000  128000 rw--- [heap]
00007ffd    8192    4096    4096 rw--- [stack]
...
----------------  -------  ------
total kB  8912000 1048576  892000
```

> 💡 `pmap` shows a **breakdown of how memory is allocated** — heap, stack, shared libraries — invaluable for detecting memory leaks.

---

### Example 8 — Accurate Shared Memory with `smem`

Standard tools count shared libraries once per process — inflating totals. `smem` uses **PSS (Proportional Set Size)** for accurate accounting:

```bash
# Install smem
sudo apt install smem    # Debian/Ubuntu
sudo dnf install smem    # RHEL

# Show top memory consumers with accurate accounting
smem -r -s pss | head -10
```

```bash
  PID User     Command              Swap      USS      PSS      RSS
 2341 mysql    /usr/sbin/mysqld        0  2180000  2201000  2248000
12345 appuser  python3                 0  1012000  1024000  1048576
```

| Metric | Description |
|--------|-------------|
| `USS` | **Unique Set Size** — memory used exclusively by this process |
| `PSS` | **Proportional Set Size** — fair share of shared memory |
| `RSS` | **Resident Set Size** — all physical RAM (shared counted fully) |

---

### Example 9 — Check if OOM Killer Has Fired

```bash
# Check journal for OOM events
sudo journalctl -k | grep -i "oom\|out of memory\|killed process"

# Check dmesg
dmesg -T | grep -i "oom\|killed"
```

```bash
# OOM killer output:
[Mar 21 10:30:01] Out of memory: Kill process 12345 (python3) score 890 or sacrifice child
[Mar 21 10:30:01] Killed process 12345 (python3) total-vm:8912kB, anon-rss:4505kB
```

> The OOM score (0–1000) determines which process is killed first — higher score = more likely to be killed.

---

### Example 10 — Memory Monitoring Script with Alert

```bash
#!/bin/bash

THRESHOLD=85        # Alert if memory used > 85%
SWAP_WARN=50        # Warn if swap used > 50%
LOGFILE="/var/log/mem_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

TOTAL=$(free | awk '/^Mem:/ {print $2}')
USED=$(free  | awk '/^Mem:/ {print $3}')
PCT=$(( USED * 100 / TOTAL ))

SWAP_TOTAL=$(free | awk '/^Swap:/ {print $2}')
SWAP_USED=$(free  | awk '/^Swap:/ {print $3}')
SWAP_PCT=$(( SWAP_TOTAL > 0 ? SWAP_USED * 100 / SWAP_TOTAL : 0 ))

echo "[$DATE] Memory: ${PCT}% | Swap: ${SWAP_PCT}%" >> "$LOGFILE"

if [ "$PCT" -gt "$THRESHOLD" ]; then
  echo "[$DATE] ⚠️  HIGH MEMORY: ${PCT}% used" | tee -a "$LOGFILE"
  echo "Top consumers:" | tee -a "$LOGFILE"
  ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -6 | tee -a "$LOGFILE"
fi

if [ "$SWAP_PCT" -gt "$SWAP_WARN" ]; then
  echo "[$DATE] 🔴 HIGH SWAP: ${SWAP_PCT}% — OOM risk!" | tee -a "$LOGFILE"
fi
```

---

## 🗂️ Memory Monitoring Tools — Quick Reference

| Tool | Type | Best For |
|------|------|----------|
| `ps aux --sort=-%mem` | Snapshot | Quick sorted list for scripting |
| `top` (press M) | Interactive | Real-time overview |
| `htop` | Interactive | Visual, color-coded memory bars |
| `pmap -x PID` | Deep dive | Memory map of one process |
| `smem` | Accurate | Fair PSS accounting for shared memory |
| `free -h` | System-wide | Total RAM and swap overview |
| `vmstat` | Sampled | Memory + swap activity over time |

---

## 🔢 Memory Metrics Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| `available` RAM | > 30% | 15–30% | < 15% |
| Swap used | 0% | < 50% | > 50% |
| Swap `si`/`so` in vmstat | 0 | Occasional | Sustained |
| OOM kills | 0 | — | Any |

---

## 🚀 Full Workflow — Diagnose High Memory Usage

```bash
# Step 1: Check overall memory and swap
free -h

# Step 2: Find top memory consumers
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10

# Step 3: Confirm with real-time view
top    # Press M to sort by memory

# Step 4: Check if OOM killer has fired
sudo journalctl -k | grep -i "oom\|killed process"

# Step 5: Check swap activity
vmstat 1 5    # Watch si/so columns

# Step 6: Investigate the biggest process
PID=2341
cat /proc/$PID/status | grep -E "^(Name|VmRSS|VmSize|VmSwap)"
sudo pmap -x $PID | tail -3

# Step 7: Take action
# Restart a leaking service
sudo systemctl restart mysqld

# Or kill the process
kill -15 $PID

# Step 8: Verify recovery
free -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Using VSZ instead of RSS | **VSZ is inflated** — always use RSS for actual memory |
| Panicking because `free` shows low free | Check `available` — Linux caches spare RAM (normal) |
| `ps %mem` adds up to over 100% | Shared libs counted per process — use `smem` for accurate totals |
| OOM killer fires but memory looks OK | Check swap `si/so` in `vmstat` — system was paging heavily |
| High memory but can't identify source | Check kernel memory: `cat /proc/meminfo | grep Slab` |

---

## 🔑 Key Takeaways

- `ps aux --sort=-%mem` gives a quick **snapshot sorted by memory** — `--sort=-%mem` means descending.
- **RSS (Resident Set Size)** is the correct metric — VSZ is inflated virtual memory, not actual RAM.
- Press **`M` in `top`** to sort by memory usage in real time.
- Low `available` RAM (not low `free`) is the real sign of memory pressure.
- **Swap `si`/`so` > 0** in `vmstat` means active memory paging — performance is degraded.
- Check `journalctl -k | grep oom` when processes disappear — the OOM killer fires silently.
- Use `smem` for **accurate PSS-based accounting** when shared libraries distort `ps` totals.

---

## 📚 Related Concepts

- [GNU ps Manual](https://linux.die.net/man/1/ps)
- [smem Documentation](https://www.selenic.com/smem/)

</details>
