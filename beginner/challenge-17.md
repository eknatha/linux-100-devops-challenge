# Challenge 17: Check Memory Usage in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Monitoring **memory (RAM)** is a fundamental Linux administration task. When a system runs low on memory, it starts using **swap space** (disk-based virtual memory), which is much slower — causing sluggish performance, high load averages, and eventually application crashes triggered by the **OOM (Out Of Memory) killer**.

Knowing how to quickly check memory status helps you:
- Identify memory pressure before services start failing
- Find which processes are consuming the most RAM
- Understand the difference between used, free, and cached memory
- Detect swap usage — a key indicator of memory problems

---

## 🧩 Task

Check how much memory is used, free, and available on the system.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Human-readable memory overview
free -h

# Detailed memory info from the kernel
cat /proc/meminfo
```

### How it works

| Command | Description |
|---------|-------------|
| `free -h` | Show RAM and swap usage in **human-readable** format |
| `cat /proc/meminfo` | Show detailed kernel memory statistics |

---

## 🔢 Understanding `free -h` Output

```bash
free -h
```

```
              total        used        free      shared  buff/cache   available
Mem:           15Gi        9.1Gi       1.2Gi       312Mi       4.7Gi       5.8Gi
Swap:           2Gi        1.8Gi       200Mi
```

| Column | Description |
|--------|-------------|
| `total` | Total physical RAM installed |
| `used` | RAM in use by processes |
| `free` | RAM not used at all |
| `shared` | Memory shared between processes (tmpfs) |
| `buff/cache` | Kernel buffers and page cache — **can be reclaimed** |
| `available` | **Estimated RAM available for new processes** — the most important field |
| `Swap used` | Swap in active use — high value means memory pressure |

> 💡 **`available` is the key field** — not `free`. Linux uses spare RAM as disk cache (`buff/cache`), which is reclaimed when needed. `available` accounts for this and is the true measure of free memory.

---

## 📖 Extended Examples

### Example 1 — Basic Memory Check

```bash
free -h
```

```bash
              total        used        free      shared  buff/cache   available
Mem:           15Gi        9.1Gi       1.2Gi       312Mi       4.7Gi       5.8Gi
Swap:           2Gi          0Gi        2Gi
```

> Swap at 0 = no memory pressure. `available` at 5.8Gi = plenty of headroom.

---

### Example 2 — Memory in Different Units

```bash
# Human-readable (KB, MB, GB)
free -h

# In megabytes
free -m

# In gigabytes
free -g

# In bytes (raw)
free -b

# In kilobytes (default)
free -k
```

---

### Example 3 — Watch Memory in Real Time

```bash
# Refresh every 2 seconds
free -h -s 2

# Watch with full screen refresh
watch -n 2 free -h
```

```bash
# watch output refreshes in place:
              total        used        free      shared  buff/cache   available
Mem:           15Gi       10.2Gi       0.9Gi       312Mi       3.9Gi       4.5Gi
Swap:           2Gi        0.8Gi       1.2Gi
```

---

### Example 4 — Detailed Memory Info from Kernel

```bash
cat /proc/meminfo
```

```bash
MemTotal:       16199680 kB    ← Total RAM
MemFree:         1228800 kB    ← Completely free
MemAvailable:    5939200 kB    ← Available for new processes
Buffers:          524288 kB    ← Kernel read buffers
Cached:          4300800 kB    ← Page cache (reclaimable)
SwapTotal:       2097152 kB    ← Total swap space
SwapFree:        2097152 kB    ← Free swap (0 used here)
Dirty:             12288 kB    ← Data waiting to be written to disk
```

```bash
# Filter for key fields
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree"
```

---

### Example 5 — Find Top Memory-Consuming Processes

```bash
# Sort processes by memory usage
ps aux --sort=-%mem | head -10

# Show PID, user, %mem, RSS, and command
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10
```

```bash
  PID USER    %MEM     RSS CMD
12345 mysql    14.2  2248000 /usr/sbin/mysqld
 4821 appuser  10.5  1048576 python3 /opt/app/server.py
 5201 www-data  1.2   196608 nginx: worker process
```

| Column | Description |
|--------|-------------|
| `%MEM` | Percentage of total RAM used |
| `RSS` | **Resident Set Size** — actual physical RAM in use (KB) |

> 💡 **RSS** is the most accurate measure of actual memory used by a process.

---

### Example 6 — Check Memory with `vmstat`

```bash
# Memory statistics with swap activity
vmstat -s -S M
```

```bash
         15820 M total memory
          9341 M used memory
          1228 M free memory
           512 M buffer memory
          4789 M swap cache
          2048 M total swap
             0 M used swap
          2048 M free swap
```

```bash
# Watch swap in/out activity (si and so columns)
vmstat 1 5
```

```bash
procs ---memory-- ---swap-- --io-- -system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so  us sy id wa
 2  0      0 1228800 524288 4800000  0    0   5  1 93  1
```

| Column | Description |
|--------|-------------|
| `swpd` | Total swap in use |
| `si` | **Swap in** — pages read from swap (high = memory pressure) |
| `so` | **Swap out** — pages written to swap (high = memory pressure) |

---

### Example 7 — Check Swap Usage

```bash
# Show all swap spaces
swapon --show

# Check swap in free
free -h | grep Swap
```

```bash
# swapon --show output:
NAME      TYPE SIZE  USED PRIO
/dev/sda3 partition  2G  1.8G   -2
```

> ⚠️ Swap at 1.8G / 2G means the system is under **severe memory pressure** — processes may be killed soon.

---

### Example 8 — Check if OOM Killer Was Triggered

```bash
# Check for OOM kills in journal
sudo journalctl -k | grep -i "oom\|out of memory\|killed process"

# Check in dmesg
dmesg -T | grep -i "oom\|killed process"
```

```bash
# OOM killer output:
[Mar 21 10:30:01] Out of memory: Kill process 4821 (python3) score 890 or sacrifice child
[Mar 21 10:30:01] Killed process 4821 (python3) total-vm:8912kB, anon-rss:4505kB
```

> The OOM killer fires silently — always check these messages when a process unexpectedly disappears.

---

### Example 9 — Memory Usage Script with Alert

```bash
#!/bin/bash

THRESHOLD=85    # Alert if memory used > 85%
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Get total and used memory in KB
TOTAL=$(free | awk '/^Mem:/ {print $2}')
USED=$(free  | awk '/^Mem:/ {print $3}')
PCT=$(( USED * 100 / TOTAL ))

# Get swap usage
SWAP_TOTAL=$(free | awk '/^Swap:/ {print $2}')
SWAP_USED=$(free  | awk '/^Swap:/ {print $3}')
SWAP_PCT=$(( SWAP_TOTAL > 0 ? SWAP_USED * 100 / SWAP_TOTAL : 0 ))

echo "[$DATE] Memory: ${PCT}% used | Swap: ${SWAP_PCT}% used"

if [ "$PCT" -gt "$THRESHOLD" ]; then
  echo "⚠️  HIGH MEMORY: ${PCT}% used — top consumers:"
  ps -eo pid,user,%mem,cmd --sort=-%mem | head -5
fi
```

---

### Example 10 — Use `htop` for Visual Memory Overview

```bash
htop
```

```
  Mem[|||||||||||||||||||||||||||||||||9.10G/15.5G]
  Swp[|||||                           0.80G/2.00G]
```

> `htop` shows color-coded memory bars — green = used, blue = buffers, yellow = cache, red = swap.

```bash
# Install htop if not available
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL/CentOS
```

---

## 🗂️ `free` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-h` | **Human-readable** (KB, MB, GB) |
| `-m` | Output in **megabytes** |
| `-g` | Output in **gigabytes** |
| `-b` | Output in **bytes** |
| `-k` | Output in **kilobytes** (default) |
| `-s N` | **Repeat** output every N seconds |
| `-c N` | **Count** — repeat N times then stop |
| `--total` | Add a **total** row |

---

## 🔄 Memory Monitoring Tools — Comparison

| Tool | Best For |
|------|----------|
| `free -h` | Quick system-wide memory overview |
| `cat /proc/meminfo` | Detailed kernel memory breakdown |
| `ps --sort=-%mem` | Top memory-consuming processes |
| `vmstat` | Memory + swap activity trends over time |
| `htop` | Visual real-time interactive view |
| `top` | Real-time processes + memory/CPU |
| `smem` | Accurate shared memory accounting (PSS) |

---

## 🔢 Memory Health Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| `available` RAM | > 30% | 15–30% | < 15% |
| Swap used | 0% | < 50% | > 50% |
| Swap `si`/`so` | 0 | Occasional | Sustained |
| OOM kills | 0 | — | Any |

---

## 🚀 Full Workflow — Investigate High Memory Usage

```bash
# Step 1: Quick overview
free -h

# Step 2: Check if swap is being used
swapon --show
free -h | grep Swap

# Step 3: Find top memory consumers
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10

# Step 4: Check if OOM killer has fired
sudo journalctl -k | grep -i "oom\|killed"

# Step 5: Watch swap activity over time
vmstat 1 10

# Step 6: Investigate the biggest consumer
PID=12345
cat /proc/$PID/status | grep -E "^(Name|VmRSS|VmSize|VmSwap)"

# Step 7: Take action
# Kill the process if necessary
kill -SIGTERM $PID

# Step 8: Verify memory recovered
free -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Panicking because `free` is near zero | Check `available` — Linux uses spare RAM as cache, which is normal |
| Ignoring swap usage | High swap = active memory pressure even if RAM shows free |
| Using `used` instead of `available` | `used` includes cache — `available` is the true free memory |
| Not checking OOM kills | Run `journalctl -k | grep oom` after any unexplained process death |
| `ps %mem` adds up to over 100% | Shared memory counted per process — use `smem` for accurate totals |

---

## 🔑 Key Takeaways

- `free -h` is the fastest way to get a system memory overview — always look at **`available`**, not `free`.
- Linux uses spare RAM as **disk cache** (`buff/cache`) — this is normal and healthy, not wasted memory.
- **Swap usage > 0** means memory pressure; **active swap `si`/`so`** means the system is actively paging — a performance problem.
- `ps --sort=-%mem` immediately shows which processes are consuming the most RAM.
- **RSS** is the correct metric for per-process memory — `VSZ` is inflated virtual memory, not actual RAM.
- The **OOM killer** fires silently — always check `journalctl -k | grep oom` when processes disappear unexpectedly.
- Use `watch -n 2 free -h` for **continuous real-time monitoring** of memory usage.

---

## 📚 Related Concepts

- [GNU free Manual](https://linux.die.net/man/1/free)
- [Linux /proc/meminfo Reference](https://www.kernel.org/doc/html/latest/filesystems/proc.html)

</details>
