# Challenge 87: Debug Slow System in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

A slow Linux system can be caused by many different bottlenecks — CPU saturation, memory pressure, disk I/O congestion, network latency, or a combination of all four. The key to diagnosing slowness is a **systematic, layered approach**: start with the high-level overview, identify which resource is the bottleneck, then drill deeper into that specific subsystem.

The three tools in this challenge cover all major subsystems:
- **`top`** — real-time process and CPU/memory overview
- **`vmstat 1`** — CPU, memory, swap, and I/O activity over time
- **`iostat -x 1`** — detailed disk I/O performance metrics

---

## 🧩 Task

Systematically identify which resource subsystem is causing a Linux system to run slowly.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Real-time process and resource overview
top

# CPU, memory, swap, and I/O summary (1-second intervals)
vmstat 1

# Detailed disk I/O statistics (1-second intervals)
iostat -x 1
```

### How it works

| Command | Description |
|---------|-------------|
| `top` | Interactive real-time view of processes, CPU, and memory |
| `vmstat 1` | Virtual memory statistics — sampled every **1 second** |
| `iostat -x 1` | Extended I/O statistics per device — sampled every **1 second** |

---

## 🔢 The Slowness Diagnosis Framework

```
System is slow
      │
      ├── Check top → High CPU?
      │         ├── Yes → Find the process (ps, pidstat, perf)
      │         └── No  → Continue
      │
      ├── Check vmstat → High wa (I/O wait)?
      │         ├── Yes → Disk or NFS bottleneck → iostat -x
      │         └── No  → Continue
      │
      ├── Check vmstat → High si/so (swap)?
      │         ├── Yes → Memory pressure → free -h, ps --sort=-%mem
      │         └── No  → Continue
      │
      ├── Check vmstat → High r (run queue)?
      │         ├── Yes → CPU bottleneck → mpstat -P ALL
      │         └── No  → Continue
      │
      └── Check network → High latency or packet loss?
                └── Yes → ping, mtr, ss -s
```

---

## 📖 Extended Examples

### Example 1 — Start with `top` for a High-Level Overview

```bash
top
```

```bash
top - 10:35:22 up 5 days, 3:17, 2 users, load average: 8.42, 7.81, 6.10
Tasks: 215 total,   5 running, 210 sleeping,   0 stopped,   0 zombie
%Cpu(s): 78.2 us, 10.3 sy,  0.0 ni,  5.1 id,  6.2 wa,  0.0 hi,  0.2 si
MiB Mem : 15,820.0 total,   1,212.0 free,  13,341.0 used,   1,267.0 buff/cache
MiB Swap:  2,048.0 total,   1,843.0 used,    205.0 free.    421.0 avail Mem

  PID USER    PR NI  VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
12345 appuser 20  0 2145000 512000 12000 R  85.2  3.2  45:12.56 python3
 2341 mysql   20  0 1892000 324000  8000 D  14.3  2.0  15:21.10 mysqld
```

#### Reading the Critical Lines

```
load average: 8.42, 7.81, 6.10   ← Load / CPU count > 1 = overloaded
%Cpu: 78.2 us, 6.2 wa            ← High user CPU AND high I/O wait
MiB Swap: 1,843 used             ← Heavy swap usage = memory pressure
mysqld: D state                  ← Uninterruptible sleep = blocked on I/O
```

| `top` Signal | What It Means |
|-------------|---------------|
| High `us` CPU | Application code consuming CPU |
| High `sy` CPU | Kernel overhead — check syscall rate |
| High `wa` CPU | **I/O wait** — disk/network bottleneck |
| High load average | More work than CPU cores can handle |
| Process in `D` state | Stuck waiting for I/O — disk problem |
| High swap used | Memory pressure — system is paging |

---

### Example 2 — `vmstat 1` for Trend Analysis

```bash
vmstat 1
```

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 5  3 1884992 124000 65536 643000 1024  512 12288 45056  850 1450 78 10  6  6  0
 4  2 1884992 118000 65536 641000  512  256  8192 32768  720 1210 75 12  4  9  0
 6  4 1884992 112000 65536 639000 2048 1024 18432 65536  980 1620 82  8  2  8  0
```

| Column | Description | Warning Level |
|--------|-------------|--------------|
| `r` | **Run queue** — processes waiting for CPU | > CPU count |
| `b` | **Blocked** — processes in uninterruptible I/O wait | > 2 |
| `swpd` | Swap in use (KB) | > 0 means memory pressure |
| `free` | Free RAM (KB) | Dropping continuously = problem |
| `si` | **Swap in** — pages read from swap/sec | > 0 consistently |
| `so` | **Swap out** — pages written to swap/sec | > 0 consistently |
| `bi` | Blocks read from device/sec (disk reads) | Context-dependent |
| `bo` | Blocks written to device/sec (disk writes) | Context-dependent |
| `in` | Interrupts per second | Spikes = hardware events |
| `cs` | **Context switches per second** | High = many short tasks |
| `us` | User CPU % | > 80% sustained = CPU bottleneck |
| `sy` | System/kernel CPU % | > 20% = kernel overhead |
| `id` | Idle CPU % | < 10% = system is saturated |
| `wa` | **I/O wait %** | > 10% = disk/network bottleneck |

> 💡 **The first `vmstat` row shows averages since boot** — ignore it. The subsequent rows show real-time activity.

---

### Example 3 — `iostat -x 1` for Disk Performance

```bash
iostat -x 1
```

```bash
Device  r/s    w/s   rkB/s   wkB/s  r_await  w_await  aqu-sz  %util
sda     45.0  480.0  3600.0 38400.0    2.5    182.4     8.1    99.8
sdb      0.2    1.0    12.8    64.0    0.8      1.2     0.0     0.1
```

| Column | Warning | Critical |
|--------|---------|---------|
| `r_await` (read latency ms) | > 20ms | > 100ms |
| `w_await` (write latency ms) | > 20ms | > 100ms |
| `aqu-sz` (queue depth) | > 1 | > 4 |
| `%util` (device busy %) | > 70% | > 90% |

> `sda` at `%util=99.8%` with `w_await=182.4ms` is severely saturated — this is the bottleneck.

---

### Example 4 — Identify CPU Bottleneck

```bash
# Find what is consuming CPU
ps -eo pid,user,cmd,%cpu --sort=-%cpu | head -10

# Per-core CPU distribution
mpstat -P ALL 1 5

# CPU usage over time for a specific process
pidstat -p 12345 1
```

```bash
# mpstat output showing one core maxed:
CPU    %usr   %sys   %iow   %idle
all    78.2   10.3    6.2    5.1
  0    99.4    0.3    0.1    0.2    ← Core 0 maxed = single-threaded bottleneck
  1    62.1   15.4    8.2   14.3
  2    70.0    8.1    5.8   16.1
  3    64.3   17.5    9.3    8.9
```

---

### Example 5 — Identify Memory Bottleneck

```bash
# System memory overview
free -h

# Find top memory consumers
ps -eo pid,user,cmd,%mem,rss --sort=-%mem | head -10

# Watch swap activity in real time
vmstat 1 | awk '{print $7, $8, $16, $17}'
# si   so   wa   id
```

```bash
# Signs of memory pressure in vmstat:
si > 0  # Swap in — system is reading from swap (pages needed in RAM)
so > 0  # Swap out — system is pushing pages to swap (RAM is full)
free dropping continuously — memory is being consumed faster than released
```

```bash
# Find the OOM killer in action
sudo journalctl -k | grep -i "oom\|killed process"
```

---

### Example 6 — Identify Disk I/O Bottleneck

```bash
# Which process is doing the most I/O?
sudo iotop -o

# Detailed disk stats with history
sar -d 1 10

# Check if writes are being flushed (dirty pages)
cat /proc/vmstat | grep dirty
```

```bash
# iotop -o output — only processes actively doing I/O:
Total DISK READ: 3.5 MB/s  | Total DISK WRITE: 37.5 MB/s
  TID USER     DISK READ   DISK WRITE  IO>   COMMAND
 2341 mysql    0.3 MB/s    36.8 MB/s  92.1%  mysqld
12345 appuser  3.2 MB/s     0.7 MB/s   7.9%  python3
```

---

### Example 7 — Identify Network Bottleneck

```bash
# Check network interface statistics
ip -s link show eth0

# Network connections summary
ss -s

# Bandwidth usage per process
sudo nethogs eth0    # Install: apt install nethogs

# Check for retransmissions (sign of congestion or packet loss)
ss -ti | grep retrans

# Interface error and drop counters
netstat -i
```

```bash
# ss -s output:
Total: 245 (kernel 312)
TCP:   210 (estab 145, closed 45, orphaned 5, synrecv 0, timewait 12)
# High timewait = many short-lived connections (HTTP without keepalive)
# High estab = many persistent connections
```

---

### Example 8 — Check for Resource Limits

```bash
# System-wide open file limit
cat /proc/sys/fs/file-max
cat /proc/sys/fs/file-nr   # used, unused, max

# Per-process limits for a running process
cat /proc/<PID>/limits

# Check for processes hitting open file limits
lsof | wc -l

# Check kernel parameters that affect performance
sysctl vm.swappiness          # How aggressively to use swap (default 60)
sysctl vm.dirty_ratio         # % of RAM for dirty pages before flush
sysctl net.core.somaxconn     # Max socket listen backlog
```

---

### Example 9 — Use `sar` for Historical Analysis

```bash
# Install sysstat
sudo apt install sysstat    # Debian/Ubuntu
sudo dnf install sysstat    # RHEL

# Historical CPU data for today
sar -u

# Historical memory data
sar -r

# Historical disk I/O data
sar -d

# Historical load average
sar -q

# All in one — CPU, memory, swap, I/O for a specific hour
sar -A --start-time 09:00:00 --end-time 10:00:00
```

```bash
# sar -q showing load spike at 10:10:
10:00:01  runq-sz  plist-sz  ldavg-1  ldavg-5  ldavg-15
10:00:01        1       315     0.42     0.55      0.61
10:10:01        8       318     8.71     5.63      3.00   ← spike!
10:20:01        2       312     2.41     3.90      2.88
```

---

### Example 10 — Slow System Quick Diagnostic One-Liner Set

```bash
# Run all key diagnostics at once (snapshot)
echo "=== LOAD ===" && uptime
echo "=== CPU ===" && mpstat 1 3 | tail -4
echo "=== MEMORY ===" && free -h
echo "=== SWAP ACTIVITY ===" && vmstat 1 3 | awk 'NR>1{print "si="$7" so="$8" wa="$16}'
echo "=== TOP PROCESSES ===" && ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -6
echo "=== DISK ===" && iostat -x 1 2 | tail -5
echo "=== DISK CONSUMERS ===" && sudo iotop -b -n 1 -o 2>/dev/null | head -6
echo "=== NETWORK ===" && ip -s link show eth0 | grep -A1 "RX\|TX"
```

---

### Example 11 — Automated Slow System Alert Script

```bash
#!/bin/bash

CPU_THRESHOLD=80
MEM_THRESHOLD=85
SWAP_THRESHOLD=50
LOAD_MULTIPLIER=1.5     # Alert if load > CPUs * this value
LOG="/var/log/perf_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")
CPUS=$(nproc)

# CPU check
CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | awk '{print $8}' | tr -d '%,')
CPU_USED=$(echo "100 - $CPU_IDLE" | bc)

# Load check
LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk -F',' '{print $1}' | tr -d ' ')

# Memory check
MEM_PCT=$(free | awk '/^Mem:/{printf "%.0f", $3/$2*100}')

# Swap check
SWAP_USED=$(free | awk '/^Swap:/{print $3}')
SWAP_TOTAL=$(free | awk '/^Swap:/{print $2}')
SWAP_PCT=$(( SWAP_TOTAL > 0 ? SWAP_USED * 100 / SWAP_TOTAL : 0 ))

# I/O wait check
IOWAIT=$(vmstat 1 2 | tail -1 | awk '{print $16}')

ALERT=""
[ "$CPU_USED" -gt "$CPU_THRESHOLD" ] && ALERT+=" HIGH_CPU=${CPU_USED}%"
[ "$MEM_PCT" -gt "$MEM_THRESHOLD" ] && ALERT+=" HIGH_MEM=${MEM_PCT}%"
[ "$SWAP_PCT" -gt "$SWAP_THRESHOLD" ] && ALERT+=" HIGH_SWAP=${SWAP_PCT}%"
[ "$IOWAIT" -gt 10 ] && ALERT+=" HIGH_IOWAIT=${IOWAIT}%"

if [ -n "$ALERT" ]; then
  echo "[$DATE] ⚠️  SLOW SYSTEM:$ALERT (load=$LOAD, cpus=$CPUS)" | tee -a "$LOG"
  echo "  Top processes:" >> "$LOG"
  ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -4 >> "$LOG"
else
  echo "[$DATE] ✅ OK (cpu=${CPU_USED}% mem=${MEM_PCT}% load=$LOAD iowait=${IOWAIT}%)" >> "$LOG"
fi
```

---

## 🗂️ Bottleneck Identification — Quick Reference

| Symptom | Tool | Key Metric | Likely Cause |
|---------|------|-----------|-------------|
| Load > CPU count | `top` / `uptime` | Load average | CPU or I/O saturation |
| High `us` CPU | `top`, `mpstat` | `%us` > 80% | Application consuming CPU |
| High `wa` CPU | `top`, `vmstat` | `%wa` > 10% | Disk or network I/O wait |
| High `sy` CPU | `top`, `vmstat` | `%sy` > 20% | Kernel overhead, context switches |
| Processes in `D` state | `top`, `ps` | `STAT=D` | Blocked on disk I/O |
| Swap active | `vmstat` | `si`/`so` > 0 | Memory pressure |
| Disk latency high | `iostat -x` | `r/w_await` > 20ms | Slow disk or saturation |
| Disk 99% utilized | `iostat -x` | `%util` > 90% | Disk is saturated |
| Network drops/errors | `ip -s link` | `errors`, `dropped` | NIC or cable issue |
| High context switches | `vmstat` | `cs` very high | Too many threads/processes |

---

## 🚀 Full Workflow — Diagnose a Slow System

```bash
# Step 1: High-level snapshot
uptime         # Load average
free -h        # Memory and swap

# Step 2: Real-time overview
top            # See CPU, memory, process states
# Press P → sort by CPU
# Press M → sort by memory
# Look for: high load, high wa, D-state processes, swap usage

# Step 3: Trend analysis (watch for 30 seconds)
vmstat 1
# Watch: r (run queue), b (blocked), si/so (swap), wa (I/O wait)

# Step 4: Narrow to the bottleneck
# High CPU?
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10
mpstat -P ALL 1 3

# High I/O wait?
iostat -x 1
sudo iotop -o

# High memory/swap?
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10
sudo journalctl -k | grep -i oom

# Step 5: Drill into the offending process
PID=$(ps -eo pid,%cpu --sort=-%cpu | awk 'NR==2{print $1}')
cat /proc/$PID/cmdline | tr '\0' ' '
sudo lsof -p $PID | wc -l
sudo strace -c -p $PID    # What syscalls is it making?

# Step 6: Check historical data
sar -u -r -d
journalctl --since "1 hour ago" -p warning

# Step 7: Take action
# CPU: renice, kill, optimize application
# Memory: kill consumers, add swap, optimize
# Disk: identify I/O source, tune writes, add cache
# Network: check MTU, check for retransmits

# Step 8: Monitor for recovery
watch -n 2 'uptime; free -h; iostat -x 1 1 | tail -5'
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| High load but low CPU | Check `wa` — I/O wait doesn't show as CPU but raises load |
| First `vmstat` row misleading | Skip it — it shows since-boot averages, not current activity |
| Blaming the top CPU process when disk is the issue | Check `D`-state processes and `wa` in top before blaming CPU |
| Checking metrics once instead of watching trends | Use `vmstat 1`, `iostat -x 1` — watch at least 10 seconds |
| `sar` shows nothing | Enable sysstat: `sudo systemctl enable --now sysstat` |
| System slow but all metrics look normal | Check network latency, NFS mounts, or DNS resolution times |

---

## 🔑 Key Takeaways

- Use `top` for a **quick overview** — load average, CPU breakdown, and process states tell most of the story.
- **`vmstat 1`** reveals trends — watch `r` (run queue), `b` (blocked), `si`/`so` (swap activity), and `wa` (I/O wait) over time.
- **`iostat -x 1`** pinpoints disk bottlenecks — `%util > 90%` and `r/w_await > 20ms` confirm a disk problem.
- A process in **`D` state** (uninterruptible sleep) is blocked on I/O — this always points to a disk or NFS issue.
- **High `wa`** in CPU breakdown means processes are waiting on disk, not CPU — never fix this by adding CPU.
- **`si`/`so` > 0** in `vmstat` means active swap use — the system is under memory pressure even if it looks idle.
- Use **`sar`** for historical data — most production performance incidents are investigated after the fact.

---

## 📚 Related Concepts

- [Challenge 68 — Check System Load]
- [Challenge 71 — Diagnose High CPU Usage]
- [Challenge 72 — Diagnose High Memory Usage]
- [Challenge 74 — Disk Latency Analysis]
- [Challenge 75 — Debug Network Latency]
- [Linux Performance Tools by Brendan Gregg](https://www.brendangregg.com/linuxperf.html)
- [sysstat tools](https://github.com/sysstat/sysstat) — `sar`, `iostat`, `mpstat`, `pidstat`


</details>
