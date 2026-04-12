# Challenge 68: Check System Load in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**System load** measures how much work the CPU and I/O subsystem are doing over time. Understanding load averages is one of the first skills in Linux performance monitoring — they give you an instant snapshot of whether a system is idle, busy, or overloaded.

The `uptime` command is the quickest way to check system load. It displays how long the system has been running and the **load averages** for the last 1, 5, and 15 minutes.

---

## 🧩 Task

Check the system's load average to assess current CPU and I/O activity.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
uptime
```

### How it works

```bash
 10:32:15 up 5 days, 3:14,  2 users,  load average: 0.45, 0.62, 0.71
```

| Field | Example | Description |
|-------|---------|-------------|
| Current time | `10:32:15` | System clock time |
| Uptime | `up 5 days, 3:14` | How long since last reboot |
| Users | `2 users` | Number of logged-in users |
| Load avg (1 min) | `0.45` | Average load over the last **1 minute** |
| Load avg (5 min) | `0.62` | Average load over the last **5 minutes** |
| Load avg (15 min) | `0.71` | Average load over the last **15 minutes** |

---

## 🔢 Understanding Load Average

Load average represents the **average number of processes waiting for CPU time or I/O** over the given period. The key reference point is the **number of CPU cores**.

```
Load Average Interpretation
─────────────────────────────────────────────────────
Load = 1.0  on a 1-core system  →  100% utilized (just right)
Load = 2.0  on a 1-core system  →  200% overloaded (1 process waiting)
Load = 4.0  on a 4-core system  →  100% utilized (all cores busy)
Load = 8.0  on a 4-core system  →  200% overloaded

Rule of thumb:  Load / CPU Cores > 1.0  =  System is overloaded
```

```bash
# Check number of CPU cores
nproc

# Or for detailed CPU info
lscpu | grep "^CPU(s):"
```

### Load Average States

| Load / Core Ratio | State | Action |
|-------------------|-------|--------|
| < 0.7 | ✅ Healthy — plenty of capacity | None |
| 0.7 – 1.0 | ⚠️ Getting busy | Monitor closely |
| 1.0 – 1.5 | 🟠 High — system under pressure | Investigate processes |
| > 1.5 | 🔴 Overloaded — performance degraded | Take immediate action |

---

## 📖 Extended Examples

### Example 1 — Basic `uptime`

```bash
uptime
```

```bash
 10:32:15 up 5 days,  3:14,  2 users,  load average: 0.45, 0.62, 0.71
```

---

### Example 2 — Load Average Only (`uptime -p` and `cut`)

```bash
# Human-readable uptime
uptime -p
```

```bash
up 5 days, 3 hours, 14 minutes
```

```bash
# Extract just the load averages
uptime | awk -F'load average:' '{print $2}'
```

```bash
0.45, 0.62, 0.71
```

---

### Example 3 — Check CPU Count to Interpret Load

```bash
# Number of logical CPUs
nproc
```

```bash
4
```

```bash
# Combine with uptime for context
echo "CPUs: $(nproc) | Load: $(uptime | awk -F'load average:' '{print $2}')"
```

```bash
CPUs: 4 | Load: 0.45, 0.62, 0.71
```

> On a 4-core system, a load of `0.71` is very light — only ~18% utilized.

---

### Example 4 — Check Load with `top`

```bash
top
```

```bash
top - 10:32:15 up 5 days, 3:14,  2 users,  load average: 0.45, 0.62, 0.71
Tasks: 210 total,   1 running, 209 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 92.8 id,  0.5 wa,  0.0 hi,  0.2 si
MiB Mem :  15,820.4 total,   8,142.0 free,   4,321.2 used,   3,357.2 buff/cache
MiB Swap:   2,048.0 total,   2,048.0 free,      0.0 used.  11,021.2 avail Mem
```

| `top` CPU Field | Description |
|----------------|-------------|
| `us` | User space processes |
| `sy` | Kernel/system processes |
| `id` | Idle — `100 - id` = total CPU usage |
| `wa` | I/O wait — high value means disk/network bottleneck |
| `hi` / `si` | Hardware/software interrupt handling |

---

### Example 5 — Check Load with `htop` (Interactive)

```bash
htop
```

> `htop` is an interactive, color-coded alternative to `top` with mouse support and per-core CPU bars. Install it if not present:
> ```bash
> sudo dnf install htop        # RHEL/CentOS
> sudo apt install htop        # Debian/Ubuntu
> ```

---

### Example 6 — Check Load with `vmstat`

```bash
vmstat 1 5
```

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 8142000 512000 3357000   0    0     5    12  210  430  5  1 93  1  0
 0  0      0 8141800 512000 3357000   0    0     0     0  195  410  4  1 95  0  0
```

| Column | Description |
|--------|-------------|
| `r` | Processes **running** or waiting for CPU |
| `b` | Processes in **uninterruptible sleep** (I/O wait) |
| `us` | User CPU time % |
| `sy` | System CPU time % |
| `id` | Idle CPU % |
| `wa` | I/O wait % |

> 💡 `vmstat 1 5` samples every 1 second for 5 iterations — shows a trend, not just a snapshot.

---

### Example 7 — Check Load with `sar` (Historical Data)

```bash
# Install sysstat if needed
sudo dnf install sysstat        # RHEL
sudo apt install sysstat        # Debian/Ubuntu

# Current CPU load
sar -u 1 5

# Historical CPU load for today
sar -u

# Load average history
sar -q
```

```bash
# sar -q output (load average history):
10:00:01  runq-sz  plist-sz  ldavg-1  ldavg-5  ldavg-15
10:10:01        2       315     0.42     0.55      0.61
10:20:01        1       312     0.38     0.50      0.58
10:30:01        3       318     0.71     0.63      0.60
```

> 💡 `sar` keeps historical performance data in `/var/log/sa/` — invaluable for investigating performance issues that happened overnight or over the weekend.

---

### Example 8 — Find Processes Causing High Load

```bash
# Sort processes by CPU usage
ps aux --sort=-%cpu | head -15

# Sort by memory usage
ps aux --sort=-%mem | head -15

# One-liner: top 5 CPU-consuming processes
ps aux --sort=-%cpu | awk 'NR<=6 {printf "%-10s %-8s %-5s %s\n", $1, $2, $3, $11}'
```

```bash
USER       PID      %CPU COMMAND
root       12345    85.2 /usr/bin/python3 heavy_script.py
mysql      2341     14.3 /usr/sbin/mysqld
nginx      5201      2.1 nginx: worker process
```

---

### Example 9 — Monitor Load Over Time with a Script

```bash
#!/bin/bash

INTERVAL=5          # Check every 5 seconds
THRESHOLD=2.0       # Alert if 1-min load exceeds this
CORES=$(nproc)

echo "Monitoring load on $CORES-core system (threshold: $THRESHOLD)"
echo "Press Ctrl+C to stop."
echo "────────────────────────────────────────────────"

while true; do
  LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk -F',' '{print $1}' | tr -d ' ')
  TIMESTAMP=$(date +"%H:%M:%S")

  if (( $(echo "$LOAD > $THRESHOLD" | bc -l) )); then
    echo "[$TIMESTAMP] 🔴 HIGH LOAD: $LOAD (threshold: $THRESHOLD, cores: $CORES)"
  else
    echo "[$TIMESTAMP] ✅ Load OK  : $LOAD"
  fi

  sleep $INTERVAL
done
```

```bash
[10:30:00] ✅ Load OK  : 0.45
[10:30:05] ✅ Load OK  : 0.51
[10:30:10] 🔴 HIGH LOAD: 2.34 (threshold: 2.0, cores: 4)
```

---

### Example 10 — Schedule Load Monitoring with Cron

```bash
crontab -e
```

```bash
# Log load average every 5 minutes
*/5 * * * * echo "$(date) | CPUs: $(nproc) | Load: $(uptime | awk -F'load average:' '{print $2}')" >> /var/log/load_monitor.log
```

```bash
# View the trend log
tail -20 /var/log/load_monitor.log
```

```bash
2025-03-21 10:00:01 | CPUs: 4 | Load:  0.42, 0.55, 0.61
2025-03-21 10:05:01 | CPUs: 4 | Load:  0.38, 0.50, 0.58
2025-03-21 10:10:01 | CPUs: 4 | Load:  2.71, 1.63, 1.00
2025-03-21 10:15:01 | CPUs: 4 | Load:  0.41, 0.90, 0.88
```

---

## 🛠️ Load Monitoring Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `uptime` | Instant snapshot | Quick load check |
| `top` | Interactive | Real-time process + load overview |
| `htop` | Interactive (color) | Visual per-core CPU + process tree |
| `vmstat` | Sampled output | CPU, memory, I/O trends over time |
| `sar` | Historical | Investigating past performance issues |
| `iostat` | I/O focused | Diagnosing disk I/O bottlenecks |
| `mpstat` | Per-core CPU | Diagnosing uneven CPU distribution |
| `glances` | All-in-one | Comprehensive system overview |

---

## 🔢 Key Metrics Reference

| Metric | Healthy Value | Warning | Critical |
|--------|--------------|---------|----------|
| Load / Core | < 0.7 | 0.7 – 1.0 | > 1.0 |
| CPU idle (`id`) | > 50% | 20–50% | < 20% |
| I/O wait (`wa`) | < 5% | 5–20% | > 20% |
| Run queue (`r` in vmstat) | ≤ cores | cores × 2 | > cores × 2 |

---

## 🚀 Full Workflow — Diagnose High System Load

```bash
# Step 1: Check current load and uptime
uptime

# Step 2: Confirm CPU count for context
nproc

# Step 3: Get a live view with top
top

# Step 4: Find the processes causing the load
ps aux --sort=-%cpu | head -10

# Step 5: Check I/O wait — is the bottleneck disk or CPU?
vmstat 1 5

# Step 6: Investigate a suspicious process
# (replace PID with the actual process ID from step 4)
ls -l /proc/PID/exe
cat /proc/PID/cmdline

# Step 7: Kill an offending process if needed
kill -15 PID        # Graceful
kill -9 PID         # Force

# Step 8: Log load over time to spot patterns
*/5 * * * * echo "$(date) | Load: $(uptime | awk -F'load average:' '{print $2}')" >> /var/log/load_monitor.log
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| High load but low CPU usage | Check I/O wait with `vmstat` — disk bottleneck is likely |
| Interpreting load without knowing core count | Always run `nproc` alongside `uptime` for context |
| 1-min load spike vs sustained load | Use the 15-min average for a stable picture of system health |
| Load looks fine but system is slow | Check memory pressure — `free -h` and swap usage in `vmstat` |
| `sar` returns no data | Enable `sysstat` service: `sudo systemctl enable --now sysstat` |

---

## 🔑 Key Takeaways

- `uptime` gives three load averages: **1-min**, **5-min**, and **15-min**.
- A load average only makes sense relative to the **number of CPU cores** — always check `nproc`.
- **Load / Cores > 1.0** means the system is overloaded — processes are waiting.
- High **I/O wait** (`wa` in `top`/`vmstat`) means the bottleneck is disk or network, not CPU.
- Use the **15-minute average** for stable trend analysis; use the **1-minute** for immediate alerting.
- `ps aux --sort=-%cpu` immediately reveals which processes are consuming the most CPU.
- Historical load data via `sar` is invaluable for diagnosing issues that happened in the past.

---

## 📚 Related Concepts

- [Linux `proc` filesystem](https://linux.die.net/man/5/proc) — raw process and system data
- [sysstat tools](https://github.com/sysstat/sysstat) — `sar`, `iostat`, `mpstat` and more

</details>
