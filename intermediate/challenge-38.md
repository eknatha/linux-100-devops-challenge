# Challenge 38: Monitor System Performance in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Real-time system performance monitoring** gives you a live view of every critical resource — CPU, memory, disk, network, and processes — all at once. Instead of running individual commands for each subsystem, performance monitoring tools aggregate everything into a single, continuously updating display.

Knowing how to read and react to real-time performance data is one of the most valuable skills for diagnosing slow systems, capacity planning, and preventing outages before they happen.

---

## 🧩 Task

Monitor system resource usage — CPU, memory, disk, and network — in real time.

---

## ✅ Solution


<details>
<summary>💡 Click to view solution</summary>

```bash
# Classic interactive process monitor
top

# Enhanced real-time monitor (recommended)
htop

# Comprehensive all-in-one monitor
vmstat 1

# Disk and CPU combined
iostat -x 1
```

### How it works

| Command | Description |
|---------|-------------|
| `top` | Real-time process list with CPU/memory summary |
| `htop` | Enhanced color-coded interactive monitor |
| `vmstat 1` | CPU, memory, swap, I/O sampled every 1 second |
| `iostat -x 1` | Extended disk I/O statistics per second |

---

## 🔢 The Performance Monitoring Stack

```
┌─────────────────────────────────────────────────────┐
│                System Performance                   │
├──────────────┬──────────────┬──────────────┬────────┤
│     CPU      │    Memory    │     Disk     │Network │
├──────────────┼──────────────┼──────────────┼────────┤
│ top / htop   │ free -h      │ iostat -x    │ ss -s  │
│ mpstat       │ vmstat       │ iotop        │ netstat│
│ pidstat      │ /proc/meminfo│ dstat        │ iftop  │
│ sar -u       │ sar -r       │ sar -d       │ sar -n │
└──────────────┴──────────────┴──────────────┴────────┘
         Combined: htop | glances | dstat
```

---

## 📖 Extended Examples

### Example 1 — `top` — Classic Real-Time Monitor

```bash
top
```

```bash
top - 10:35:22 up 3 days,  1:35,  2 users,  load average: 1.42, 1.21, 0.98
Tasks: 215 total,   2 running, 213 sleeping,   0 stopped,   0 zombie
%Cpu(s): 18.2 us,  3.3 sy,  0.0 ni, 76.8 id,  1.5 wa,  0.0 hi,  0.2 si
MiB Mem : 15,820.0 total,  5,812.0 free,  9,341.0 used,    667.0 buff/cache
MiB Swap:  2,048.0 total,  2,048.0 free,      0.0 used.  6,021.0 avail Mem
```

**`top` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `P` | Sort by **CPU** |
| `M` | Sort by **Memory** |
| `1` | Toggle **per-core** CPU view |
| `k` | **Kill** a process |
| `r` | **Renice** a process |
| `u` | Filter by **user** |
| `q` | **Quit** |
| `h` | **Help** |

---

### Example 2 — `htop` — Enhanced Interactive Monitor

```bash
htop
```

```bash
# Install if needed
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL
```

**What makes `htop` better than `top`:**
- Color-coded per-CPU bars at the top
- Visual memory and swap bars
- Mouse support — click to sort columns
- Process tree view (`F5`)
- Filter by name (`F4`)
- Kill without knowing PID (`F9`)

**`htop` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `F2` | **Setup** — configure display |
| `F4` | **Filter** by process name |
| `F5` | Toggle **tree** view |
| `F6` | **Sort** by column |
| `F9` | **Kill** selected process |
| `u` | Filter by **user** |
| `/` | **Search** |
| `q` | **Quit** |

---

### Example 3 — `vmstat` — CPU, Memory, and I/O Trends

```bash
vmstat 1
```

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 5812000 262144 667000   0    0     8    22  210  430 18  3 77  1  0
 1  0      0 5810000 262144 667000   0    0     0    12  195  410 15  2 82  1  0
```

**Key columns to watch:**

| Column | Watch For |
|--------|-----------|
| `r` | Run queue > CPU count = overloaded |
| `b` | > 2 = processes blocked on I/O |
| `si`/`so` | > 0 = active swap = memory pressure |
| `wa` | > 10% = disk bottleneck |
| `id` | < 10% = system heavily loaded |

---

### Example 4 — `iostat` — Disk Performance

```bash
iostat -x 1
```

```bash
Device  r/s   w/s  rMB/s  wMB/s  r_await  w_await  aqu-sz  %util
sda     10.0  45.0    0.5    3.5     1.2      8.4     0.4    42.0
nvme0n1  0.5   2.0    0.1    0.5     0.1      0.2     0.0     0.5
```

---

### Example 5 — `glances` — All-in-One Monitor

```bash
# Install glances
sudo apt install glances    # Debian/Ubuntu
sudo dnf install glances    # RHEL
pip install glances         # via pip

# Launch
glances
```

```bash
# glances shows everything in one screen:
# CPU usage bars, memory, swap
# Disk I/O, network I/O
# Process list with alerts
# Filesystem usage
# Sensors (temperature)
```

> 💡 `glances` is the most comprehensive single-screen monitor — it covers CPU, memory, disk, network, temperatures, and processes all at once with color-coded alerts.

---

### Example 6 — `dstat` — Combined Real-Time Stats

```bash
# Install dstat
sudo apt install dstat

# CPU + disk + network + memory combined
dstat -cdnm 1

# All stats
dstat -a 1
```

```bash
# dstat -cdnm output:
----total-usage---- -dsk/total- -net/total- ------memory-usage-----
usr sys idl wai stl| read  writ| recv  send| used  free  buff  cach
 18   3  77   1   0| 512k 3500k| 12.3k 45.1k|9341M 5812M  256M  642M
 15   2  82   1   0|    0 1200k| 8.1k  32.4k|9342M 5811M  256M  642M
```

> `dstat` combines all subsystems in one scrolling view — perfect for correlating CPU spikes with I/O or network activity.

---

### Example 7 — `sar` — Comprehensive Historical and Real-Time Stats

```bash
# Install sysstat
sudo apt install sysstat

# Real-time CPU every 2 seconds
sar -u 2 5

# Real-time memory every 2 seconds
sar -r 2 5

# Real-time disk I/O
sar -d 2 5

# Real-time network
sar -n DEV 2 5

# Historical data for today (all metrics)
sar
```

```bash
# sar -u output:
10:35:22    %user  %system  %iowait  %idle
10:35:24    18.20     3.30     1.50  76.80
10:35:26    15.10     2.20     0.80  81.60
```

---

### Example 8 — `watch` — Refresh Any Command Periodically

```bash
# Refresh free memory every 2 seconds
watch -n 2 free -h

# Watch disk usage
watch -n 5 'df -h | grep -v tmpfs'

# Watch top processes
watch -n 2 'ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -10'

# Watch network connections
watch -n 2 'ss -s'
```

---

### Example 9 — `/proc` — Raw Performance Data

```bash
# CPU stats from kernel
cat /proc/stat

# Memory from kernel
cat /proc/meminfo | grep -E "MemTotal|MemAvailable|SwapTotal|SwapFree"

# I/O stats from kernel
cat /proc/diskstats

# Network stats from kernel
cat /proc/net/dev

# Load average
cat /proc/loadavg
```

```bash
# /proc/loadavg output:
1.42 1.21 0.98 2/215 4821
# 1-min 5-min 15-min running/total lastPID
```

---

### Example 10 — Quick System Health Snapshot Script

```bash
#!/bin/bash

echo "╔══════════════════════════════════════════╗"
echo "║      SYSTEM PERFORMANCE SNAPSHOT         ║"
echo "║      $(date '+%Y-%m-%d %H:%M:%S')        ║"
echo "╚══════════════════════════════════════════╝"
echo ""

echo "── Load Average ─────────────────────────────"
uptime

echo -e "\n── CPU Usage ────────────────────────────────"
top -bn1 | grep "Cpu(s)" | awk '{print "User:", $2, "| System:", $4, "| Idle:", $8, "| Wait:", $10}'

echo -e "\n── Memory ───────────────────────────────────"
free -h

echo -e "\n── Disk I/O ─────────────────────────────────"
iostat -x 1 2 | grep -v "^$\|^Linux\|^Device" | tail -5

echo -e "\n── Top 5 CPU Processes ──────────────────────"
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -6

echo -e "\n── Top 5 Memory Processes ───────────────────"
ps -eo pid,user,%cpu,%mem,cmd --sort=-%mem | head -6

echo -e "\n── Disk Usage ───────────────────────────────"
df -h | grep -v tmpfs | awk '$5 > "50%"'

echo -e "\n── Network Connections ──────────────────────"
ss -s | grep -E "TCP|UDP"
```

---

### Example 11 — Continuous Performance Logging

```bash
#!/bin/bash
# Log performance metrics every 5 minutes

LOGFILE="/var/log/perf_$(date +%Y%m%d).log"

while true; do
  DATE=$(date +"%Y-%m-%d %H:%M:%S")
  LOAD=$(uptime | awk -F'load average:' '{print $2}')
  MEM_PCT=$(free | awk '/^Mem:/{printf "%.0f", $3/$2*100}')
  SWAP_PCT=$(free | awk '/^Swap:/{if ($2>0) printf "%.0f", $3/$2*100; else print 0}')
  IOWAIT=$(vmstat 1 2 | tail -1 | awk '{print $16}')
  DISK=$(df -h / | awk 'NR==2{print $5}')

  echo "[$DATE] Load:$LOAD | Mem:${MEM_PCT}% | Swap:${SWAP_PCT}% | IOWait:${IOWAIT}% | Disk:$DISK" \
    | tee -a "$LOGFILE"

  sleep 300
done
```

---

## 🗂️ Performance Monitoring Tools — Comparison

| Tool | CPU | Memory | Disk | Network | Interactive | Historical |
|------|-----|--------|------|---------|-------------|------------|
| `top` | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| `htop` | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| `vmstat` | ✅ | ✅ | Partial | ❌ | ❌ | ❌ |
| `iostat` | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| `dstat` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| `glances` | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| `sar` | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |

---

## 🔢 Performance Health Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Load / CPU cores | < 0.7 | 0.7–1.0 | > 1.0 |
| CPU idle (`id`) | > 50% | 20–50% | < 20% |
| I/O wait (`wa`) | < 5% | 5–20% | > 20% |
| Available RAM | > 30% | 15–30% | < 15% |
| Swap used | 0% | < 50% | > 50% |
| Disk `%util` | < 60% | 60–90% | > 90% |
| Disk latency | < 10ms | 10–50ms | > 50ms |

---

## 🚀 Full Workflow — Investigate a Performance Problem

```bash
# Step 1: Quick overview
uptime && free -h

# Step 2: Real-time interactive view
htop    # or top

# Step 3: Identify the bottleneck subsystem
vmstat 1 10
# High 'r' = CPU, high 'b'/'wa' = disk, high 'si'/'so' = memory

# Step 4a: If CPU bottleneck
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10
mpstat -P ALL 1 3

# Step 4b: If disk bottleneck
iostat -x 1
sudo iotop -o

# Step 4c: If memory bottleneck
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10
sudo journalctl -k | grep -i oom

# Step 5: Historical context
sar -u -r -d

# Step 6: Take action and monitor recovery
watch -n 2 'uptime; free -h'
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| High load but low CPU | Check `wa` — disk may be the bottleneck |
| Memory looks free but swap is used | Check `available`, not `free`; active swap = memory pressure |
| Tools not installed | `sudo apt install sysstat htop glances dstat` |
| First `vmstat`/`iostat` line is misleading | Shows since-boot average — skip first line |
| Monitoring uses too many resources | Use `sar` for low-overhead historical collection |

---

## 🔑 Key Takeaways

- **`htop`** is the best interactive real-time monitor — covers CPU and memory with visual bars.
- **`vmstat 1`** is the fastest way to see which subsystem (CPU/memory/disk) is the bottleneck.
- **High `wa`** (I/O wait) means disk is the bottleneck — not CPU, even if load is high.
- **`dstat -cdnm 1`** combines all subsystems in one scrolling view — ideal for correlation.
- **`glances`** gives the most complete single-screen overview — CPU, memory, disk, network, temps.
- **`sar`** is the only tool with **historical data** — essential for post-incident investigation.
- Always check multiple subsystems before concluding — performance problems rarely have a single cause.

---

## 📚 Related Concepts

- [glances Documentation](https://nicolargo.github.io/glances/)
- [sysstat (sar, iostat, vmstat)](https://github.com/sysstat/sysstat)

</details>
