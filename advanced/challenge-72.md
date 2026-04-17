# Challenge 72: Diagnose High Memory Usage in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Memory exhaustion is one of the most common causes of system instability, OOM (Out Of Memory) kills, and application crashes. Unlike CPU which recovers instantly when a process ends, **memory pressure builds gradually** — by the time you notice, the system may already be swapping heavily or the OOM killer may have already terminated processes.

Linux provides a rich set of tools to monitor memory at every level — from system-wide summaries to per-process breakdowns and kernel-level details.

---

## 🧩 Task

Identify overall memory usage and find which processes are consuming the most memory.

---



<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# System-wide memory overview
free -h

# Top memory-consuming processes
ps -eo pid,cmd,%mem --sort=-%mem | head
```

### How it works

#### `free -h`

| Part | Description |
|------|-------------|
| `free` | Display system-wide memory usage |
| `-h` | **Human-readable** output (KB, MB, GB) |

#### `ps -eo pid,cmd,%mem --sort=-%mem | head`

| Part | Description |
|------|-------------|
| `-e` | List **all** processes |
| `-o pid,cmd,%mem` | Output PID, command name, and memory % |
| `--sort=-%mem` | Sort by memory usage **descending** (highest first) |
| `\| head` | Show only the top 10 results |

---

## 🔢 Understanding `free -h` Output

```bash
free -h
```

```
              total        used        free      shared  buff/cache   available
Mem:           15Gi        9.1Gi       4.2Gi       312Mi       1.7Gi      6.0Gi
Swap:           2Gi          0Gi        2Gi
```

| Column | Description |
|--------|-------------|
| `total` | Total physical RAM installed |
| `used` | RAM currently in use by processes |
| `free` | RAM not used at all |
| `shared` | Memory used by `tmpfs` and shared between processes |
| `buff/cache` | Memory used by kernel buffers and page cache (can be reclaimed) |
| `available` | **Estimated RAM available for new processes** — the most useful field |
| `Swap used` | How much swap is active — high swap = memory pressure |

> 💡 **`available` is the key field** — not `free`. Linux uses spare RAM for disk cache (`buff/cache`), which is reclaimed when needed. The `available` value accounts for this.

---

## 📖 Extended Examples

### Example 1 — System Memory Overview

```bash
free -h
```

```
              total        used        free      shared  buff/cache   available
Mem:           15Gi        9.1Gi       4.2Gi       312Mi       1.7Gi      6.0Gi
Swap:           2Gi        1.8Gi       200Mi
```

> ⚠️ Swap at 1.8Gi / 2Gi means the system is under **severe memory pressure** — the OOM killer may be close to activating.

---

### Example 2 — Top Memory Consumers with `ps`

```bash
ps -eo pid,cmd,%mem --sort=-%mem | head
```

```bash
  PID CMD                         %MEM
12345 java -Xmx4g -jar app.jar   28.4
 2341 /usr/sbin/mysqld            14.2
 8901 python3 /opt/ml/train.py    10.5
 5201 nginx: worker process        1.2
  801 /usr/sbin/sshd               0.3
```

---

### Example 3 — Extended Process Memory Info

```bash
ps -eo pid,ppid,user,cmd,%mem,rss,vsz --sort=-%mem | head -15
```

```bash
  PID  PPID USER    CMD                    %MEM     RSS      VSZ
12345  1200 appuser java -Xmx4g -jar app  28.4  4505600  8912000
 2341     1 mysql   /usr/sbin/mysqld       14.2  2248000  4523000
```

| Column | Description |
|--------|-------------|
| `%MEM` | Percentage of total RAM used |
| `RSS` | **Resident Set Size** — actual physical RAM used (KB) |
| `VSZ` | **Virtual Size** — total virtual memory allocated (KB) — often much larger than RSS |

> 💡 **RSS** is the most accurate measure of actual memory consumption. **VSZ** includes memory that is mapped but not yet loaded — do not use it to gauge real usage.

---

### Example 4 — Memory Usage Sorted by RSS (Absolute RAM)

```bash
ps -eo pid,user,cmd,rss --sort=-rss | head -10 | awk '{printf "%-8s %-12s %-40s %s MB\n", $1, $2, $3, int($4/1024)}'
```

```bash
PID      USER         CMD                                      RSS MB
12345    appuser      java                                     4400 MB
2341     mysql        /usr/sbin/mysqld                         2195 MB
8901     ml-user      python3                                  1024 MB
```

---

### Example 5 — Check Memory with `vmstat`

```bash
vmstat -s -S M
```

```bash
         15820 M total memory
          9341 M used memory
          6124 M active memory
          2215 M inactive memory
          4212 M free memory
           512 M buffer memory
          1756 M swap cache
          2048 M total swap
          1843 M used swap
           205 M free swap
```

```bash
# Sampled memory activity (every 1 second, 5 times)
vmstat 1 5
```

```bash
procs ---memory--  ---swap-- --io-- -system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so   bi   bo  id wa
 2  1 1884992 4312000 524288 1798000 1024 512  128  256  10  8
```

| `vmstat` Column | Description |
|----------------|-------------|
| `swpd` | Total swap in use (KB) |
| `free` | Free physical memory (KB) |
| `si` | Swap **in** — pages read from swap (high = memory pressure) |
| `so` | Swap **out** — pages written to swap (high = memory pressure) |

---

### Example 6 — Check Swap Usage and OOM Risk

```bash
# Check swap usage
swapon --show

# Or with free
free -h

# See if OOM killer has been triggered recently
sudo journalctl -k | grep -i "oom\|out of memory\|killed process"
```

```bash
# OOM killer log output example:
Mar 21 10:30:01 hostname kernel: Out of memory: Kill process 12345 (python3) score 890 or sacrifice child
Mar 21 10:30:01 hostname kernel: Killed process 12345 (python3) total-vm:8912kB, anon-rss:4505kB
```

---

### Example 7 — Per-Process Memory with `pmap`

```bash
# Full memory map for a process
sudo pmap -x 12345 | tail -5
```

```bash
Address   Kbytes     RSS   Dirty Mode  Mapping
...
00007f8a  2048000  512000  128000 rw--- [heap]
00007ffd    8192    4096    4096 rw--- [stack]
----------------  -------  ------
total kB  8912000 4505600  892000
```

> 💡 `pmap` shows a breakdown of how a process's memory is allocated — heap, stack, shared libs, mapped files — invaluable for memory leak investigation.

---

### Example 8 — `/proc` Memory Details

```bash
# System-wide memory summary
cat /proc/meminfo
```

```bash
MemTotal:       16199680 kB
MemFree:         4313088 kB
MemAvailable:    6144000 kB
Buffers:          524288 kB
Cached:          1798144 kB
SwapTotal:       2097152 kB
SwapFree:         210000 kB
Dirty:             12288 kB
```

```bash
# Per-process memory breakdown
cat /proc/12345/status | grep -E "^(VmRSS|VmSize|VmSwap|Name)"
```

```bash
Name:   python3
VmSize:   8912000 kB    ← Virtual memory
VmRSS:    4505600 kB    ← Physical RAM in use
VmSwap:    204800 kB    ← Swap being used by this process
```

---

### Example 9 — Check Memory with `htop`

```bash
htop
```

**Useful `htop` memory features:**

| Display | Meaning |
|---------|---------|
| Green bar | Used memory |
| Blue bar | Buffer memory |
| Yellow/Orange bar | Cache memory |
| Red/Orange in Mem bar | Used swap |

**Key shortcuts:**

| Key | Action |
|-----|--------|
| `M` | Sort by memory |
| `F4` | Filter processes |
| `T` | Show process tree |
| `u` | Filter by user |

---

### Example 10 — Smem — Accurate Shared Memory Accounting

Standard tools like `ps` and `top` count shared libraries separately for each process, which inflates totals. `smem` calculates **PSS (Proportional Set Size)** — a fair accounting of shared memory.

```bash
sudo apt install smem    # Debian/Ubuntu
sudo dnf install smem    # RHEL

# Top memory users with accurate shared memory accounting
smem -r -s pss | head -15

# User-level memory summary
smem -r -u

# System-wide totals
smem -t
```

```bash
  PID User     Command                   Swap      USS      PSS      RSS
12345 appuser  java -Xmx4g -jar app.jar     0  4380000  4412000  4505600
 2341 mysql    /usr/sbin/mysqld             0  2180000  2201000  2248000
```

| Metric | Description |
|--------|-------------|
| `USS` | Unique Set Size — memory used **exclusively** by this process |
| `PSS` | Proportional Set Size — fair share of shared memory included |
| `RSS` | Resident Set Size — all physical memory, shared counted fully |

---

### Example 11 — Memory Monitoring Script with OOM Alert

```bash
#!/bin/bash

THRESHOLD_PCT=85           # Alert if memory used > 85%
SWAP_THRESHOLD_PCT=50      # Alert if swap used > 50%
LOGFILE="/var/log/mem_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Get memory stats
TOTAL=$(free | awk '/^Mem:/ {print $2}')
USED=$(free  | awk '/^Mem:/ {print $3}')
AVAIL=$(free | awk '/^Mem:/ {print $7}')
PCT=$(( USED * 100 / TOTAL ))

SWAP_TOTAL=$(free | awk '/^Swap:/ {print $2}')
SWAP_USED=$(free  | awk '/^Swap:/ {print $3}')
SWAP_PCT=$(( SWAP_TOTAL > 0 ? SWAP_USED * 100 / SWAP_TOTAL : 0 ))

# Memory check
if [ "$PCT" -gt "$THRESHOLD_PCT" ]; then
  echo "[$DATE] ⚠️  HIGH MEMORY: ${PCT}% used (threshold: ${THRESHOLD_PCT}%)" | tee -a "$LOGFILE"
  echo "[$DATE] Top consumers:" >> "$LOGFILE"
  ps -eo pid,user,cmd,%mem --sort=-%mem | head -6 >> "$LOGFILE"
else
  echo "[$DATE] ✅ Memory OK: ${PCT}% used" >> "$LOGFILE"
fi

# Swap check
if [ "$SWAP_PCT" -gt "$SWAP_THRESHOLD_PCT" ]; then
  echo "[$DATE] 🔴 HIGH SWAP: ${SWAP_PCT}% used — OOM risk!" | tee -a "$LOGFILE"
fi
```

---

## 🛠️ Memory Diagnostic Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `free -h` | Snapshot | Quick system-wide overview |
| `ps --sort=-%mem` | Snapshot | Ranked list of memory consumers |
| `top` / `htop` | Interactive | Real-time per-process monitoring |
| `vmstat` | Sampled | Memory + swap activity trends |
| `pmap` | Deep dive | Memory map breakdown for one process |
| `/proc/meminfo` | Raw | Kernel-level memory details |
| `smem` | Accurate | Fair shared-memory accounting (PSS) |
| `sar -r` | Historical | Investigating past memory pressure |

---

## 🔢 Memory Health Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Available RAM | > 30% | 15–30% | < 15% |
| Swap used | 0% | < 50% | > 50% |
| Swap in/out (`si`/`so`) | 0 | Occasional | Sustained |
| OOM kills | 0 | — | Any |

---

## 🚀 Full Workflow — Diagnose and Resolve High Memory Usage

```bash
# Step 1: Get system memory overview
free -h

# Step 2: Check if swap is being used (= memory pressure)
swapon --show
vmstat 1 3   # Watch si/so columns for swap activity

# Step 3: Identify top memory consumers
ps -eo pid,user,cmd,%mem,rss --sort=-%mem | head -10

# Step 4: Check if OOM killer has already fired
sudo journalctl -k | grep -i "oom\|killed process"

# Step 5: Investigate the biggest consumer
HIGH_PID=12345
cat /proc/$HIGH_PID/status | grep -E "^(Name|VmRSS|VmSize|VmSwap)"
sudo pmap -x $HIGH_PID | tail -3

# Step 6: Decide on action
# Option A: Kill the offending process
kill -SIGTERM $HIGH_PID

# Option B: Restart a leaking service
sudo systemctl restart myapp.service

# Option C: Add more swap as a temporary buffer
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Step 7: Verify memory has recovered
free -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Panicking because `free` is near zero | Check `available` — Linux uses spare RAM for cache, which is normal |
| Using `VSZ` to judge memory consumption | Use `RSS` or `PSS` — `VSZ` includes unmapped virtual space and is inflated |
| High memory but no obvious process | Check kernel memory: `cat /proc/meminfo` for `Slab` and `KernelStack` |
| Swap used but no memory pressure visible | Check `vmstat si/so` — if 0, swap is just parked data not actively paging |
| `ps %mem` adds up to more than 100% | Shared memory is counted per-process — use `smem` for accurate totals |
| OOM kills happened overnight | Check `journalctl -k --since yesterday | grep -i oom` |

---

## 🔑 Key Takeaways

- `free -h` `available` column is the most useful field — not `free`. Linux caches aggressively.
- `ps --sort=-%mem` gives an instant ranked list of memory hogs — use `rss` for absolute byte values.
- **RSS** is the real-world memory consumption; **VSZ** is inflated virtual memory — always use RSS.
- High **swap usage** combined with high `si`/`so` in `vmstat` = active memory pressure = danger zone.
- The **OOM killer** fires silently — always check `journalctl -k | grep -i oom` after unexplained process deaths.
- `pmap -x PID` reveals heap growth and memory leaks for a specific process.
- Use `smem` for the most accurate accounting when shared libraries make `ps` totals misleading.

---

## 📚 Related Concepts

- [Linux OOM Killer](https://www.kernel.org/doc/html/latest/admin-guide/mm/concepts.html)
- [proc/meminfo Reference](https://www.kernel.org/doc/html/latest/filesystems/proc.html)
- [smem Documentation](https://www.selenic.com/smem/)

</details>
