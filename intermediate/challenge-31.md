# Challenge 31: Top CPU Consuming Processes in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Identifying which processes are consuming the most CPU is one of the first steps when a system becomes slow, unresponsive, or has a high load average. Linux provides several tools — from simple snapshots to real-time interactive monitors — to quickly find CPU-hungry processes.

Common scenarios:
- System feels sluggish and you need to find the cause
- Load average is high but you do not know which process is responsible
- A runaway script or application is monopolizing CPU
- Monitoring a server for unexpected CPU spikes

---

## 🧩 Task

Find which processes are consuming the most CPU on the system.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Real-time interactive view sorted by CPU
top

# Sorted snapshot — top CPU consumers
ps aux --sort=-%cpu | head -10

# Custom columns sorted by CPU
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10
```

### How it works

| Command | Description |
|---------|-------------|
| `top` | Real-time interactive monitor — press `P` to sort by CPU |
| `ps aux --sort=-%cpu` | Snapshot sorted by CPU descending (highest first) |
| `ps -eo ...` | Custom column selection with sort |

---

## 🔢 Understanding CPU in `ps aux`

```bash
ps aux --sort=-%cpu | head -5
```

```
USER       PID  %CPU  %MEM    VSZ    RSS  TTY   STAT  START   TIME  COMMAND
appuser  12345  85.2   3.2  2145000 512000 ?    R    10:00  45:12  python3 /opt/app/server.py
mysql     2341  14.3   2.0  1892000 324000 ?    Sl   08:00   5:21  /usr/sbin/mysqld
www-data  5201   2.1   0.2   456000  32000 ?    S    09:00   0:45  nginx: worker process
root         1   0.0   0.1    16952   5612 ?    Ss   Mar20   0:05  /sbin/init
```

| Column | Description |
|--------|-------------|
| `%CPU` | **Current CPU usage percentage** |
| `TIME` | **Total CPU time** consumed since start |
| `STAT` | `R` = Running (actively using CPU), `S` = Sleeping |

> ⚠️ `%CPU` in `ps` shows a **lifetime average** — not necessarily the current real-time usage. Use `top` for current usage.

---

## 📖 Extended Examples

### Example 1 — Interactive `top` (Real-Time)

```bash
top
```

```bash
top - 10:35:22 up 3 days, load average: 3.42, 2.81, 2.10
Tasks: 215 total,   3 running, 212 sleeping
%Cpu(s): 78.2 us, 10.3 sy,  0.0 ni,  5.1 id,  6.2 wa
MiB Mem : 15,820.0 total,  1,212.0 free, 13,341.0 used

  PID USER     %CPU  %MEM     TIME+ COMMAND
12345 appuser  85.2   3.2  45:12.56 python3
 2341 mysql    14.3   2.0  15:21.10 mysqld
 5201 www-data  2.1   0.2   0:45.20 nginx
```

**Key `top` shortcuts for CPU investigation:**

| Key | Action |
|-----|--------|
| `P` | Sort by **CPU** (default) |
| `1` | Toggle **per-core** CPU view |
| `k` | **Kill** a process by PID |
| `r` | **Renice** (lower priority) a process |
| `u` | Filter by **user** |
| `q` | **Quit** |

---

### Example 2 — `ps` Snapshot Sorted by CPU

```bash
# Top 10 CPU consumers
ps aux --sort=-%cpu | head -10

# Skip header line and show top 5
ps aux --sort=-%cpu | awk 'NR==1 || NR<=6'
```

---

### Example 3 — Custom Columns with `ps`

```bash
# Clean output: PID, user, %CPU, command
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10
```

```bash
  PID USER     %CPU CMD
12345 appuser  85.2 python3 /opt/app/server.py
 2341 mysql    14.3 /usr/sbin/mysqld
 5201 www-data  2.1 nginx: worker process
```

---

### Example 4 — Show Only Processes Above a CPU Threshold

```bash
# Show only processes consuming more than 10% CPU
ps -eo pid,user,%cpu,cmd --sort=-%cpu | awk 'NR==1 || $3 > 10'
```

```bash
  PID USER     %CPU CMD
12345 appuser  85.2 python3 /opt/app/server.py
 2341 mysql    14.3 /usr/sbin/mysqld
```

---

### Example 5 — Per-Core CPU Usage with `mpstat`

```bash
# Install sysstat if needed
sudo apt install sysstat

# CPU usage per core
mpstat -P ALL 1 3
```

```bash
CPU    %usr   %sys   %iow   %idle
all    78.2   10.3    6.2    5.1
  0    99.4    0.3    0.1    0.2   ← Core 0 maxed = single-threaded bottleneck
  1    62.1   15.4    8.2   14.3
  2    70.0    8.1    5.8   16.1
  3    64.3   17.5    9.3    8.9
```

> 💡 If one core is at 99% while others are idle, it indicates a **single-threaded bottleneck** — one process is maxing out a single CPU core.

---

### Example 6 — Track a Process CPU Over Time with `pidstat`

```bash
# Monitor a specific PID every second
pidstat -p 12345 1

# Monitor all processes, refresh every 2 seconds
pidstat 2

# Show only processes above 10% CPU
pidstat 1 | awk 'NR<=3 || $8 > 10'
```

```bash
# pidstat output:
Time    UID    PID  %usr %system  %CPU  CPU Command
10:35  1001  12345  84.0     1.2  85.2    2  python3
10:36  1001  12345  83.5     1.5  85.0    2  python3
10:37  1001  12345  85.1     1.1  86.2    2  python3
```

> `pidstat` shows CPU usage **per second** rather than a lifetime average — much more accurate than `ps`.

---

### Example 7 — Visual CPU Monitoring with `htop`

```bash
htop
```

```bash
# Install if not available
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL
```

**`htop` advantages over `top`:**
- Color-coded per-core CPU bars at the top
- Mouse support — click to sort columns
- Press `F6` to choose sort field
- Press `F4` to filter by process name
- Press `F9` to send a signal (kill)
- Tree view shows parent-child relationships

---

### Example 8 — Find CPU-Heavy Process and Investigate

```bash
# Step 1: Find the top CPU process
PID=$(ps -eo pid,%cpu --sort=-%cpu | awk 'NR==2 {print $1}')

# Step 2: See its full command
cat /proc/$PID/cmdline | tr '\0' ' '

# Step 3: See what it's doing (syscalls)
sudo strace -c -p $PID

# Step 4: Check its open files
sudo lsof -p $PID | head -10

# Step 5: Check its network connections
sudo ss -tp | grep $PID
```

---

### Example 9 — CPU Monitoring Script with Alert

```bash
#!/bin/bash

THRESHOLD=80    # Alert if any process exceeds this CPU %
LOGFILE="/var/log/cpu_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Find processes exceeding threshold
HIGH_CPU=$(ps -eo pid,user,%cpu,cmd --sort=-%cpu | awk -v t="$THRESHOLD" 'NR>1 && $3 > t')

if [ -n "$HIGH_CPU" ]; then
  echo "[$DATE] ⚠️  HIGH CPU ALERT:" | tee -a "$LOGFILE"
  echo "$HIGH_CPU" | tee -a "$LOGFILE"
  echo "[$DATE] System load: $(uptime | awk -F'load average:' '{print $2}')" | tee -a "$LOGFILE"
else
  echo "[$DATE] ✅ CPU OK" >> "$LOGFILE"
fi
```

---

### Example 10 — Renice a CPU-Heavy Process

Instead of killing a CPU-hungry process, you can **lower its priority**:

```bash
# Lower priority of a process (nice value 0–19, higher = lower priority)
sudo renice +15 -p 12345

# Verify the new priority
ps -p 12345 -o pid,ni,cmd
```

```bash
  PID  NI CMD
12345  15 python3 /opt/app/server.py
```

```bash
# Launch a new process with lower priority
nice -n 15 python3 heavy_script.py

# Limit CPU usage with cpulimit
sudo apt install cpulimit
sudo cpulimit -p 12345 -l 30    # Limit to max 30% CPU
```

---

## 🗂️ CPU Monitoring Tools — Quick Reference

| Tool | Type | Best For |
|------|------|----------|
| `top` | Interactive | Real-time all-process overview |
| `htop` | Interactive | Visual, color-coded, mouse support |
| `ps aux --sort=-%cpu` | Snapshot | Quick sorted list for scripting |
| `mpstat -P ALL` | Sampled | Per-core CPU distribution |
| `pidstat -p PID` | Sampled | Tracking one process over time |
| `sar -u` | Historical | Investigating past CPU spikes |

---

## 🔢 CPU Metrics Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| `us` (user) | < 70% | 70–85% | > 85% |
| `sy` (system) | < 10% | 10–25% | > 25% |
| `wa` (I/O wait) | < 5% | 5–20% | > 20% |
| `id` (idle) | > 30% | 15–30% | < 15% |
| Load / Core | < 0.7 | 0.7–1.0 | > 1.0 |

---

## 🚀 Full Workflow — Diagnose High CPU Usage

```bash
# Step 1: Check load average
uptime

# Step 2: Find the top CPU consumers
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10

# Step 3: Confirm with real-time view
top    # Press P to sort by CPU

# Step 4: Check per-core distribution
mpstat -P ALL 1 3

# Step 5: Investigate the offending process
PID=12345
cat /proc/$PID/cmdline | tr '\0' ' '

# Step 6: Decide on action
# Option A: Lower priority (less disruptive)
sudo renice +15 -p $PID

# Option B: Graceful termination
kill -15 $PID

# Option C: Force kill
kill -9 $PID

# Step 7: Verify CPU normalizes
watch -n 2 'ps -eo pid,user,%cpu --sort=-%cpu | head -5'
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `ps %cpu` shows 0 for an active process | `ps` shows lifetime average — use `top` or `pidstat` for real-time |
| High load but low CPU in `top` | Check `wa` (I/O wait) — disk may be the bottleneck |
| One core at 100% — others idle | Single-threaded process — use `mpstat -P ALL` to confirm |
| Killing process that respawns | Check if a service manager (systemd) is restarting it |
| `%cpu` can exceed 100% | Normal for multi-threaded processes on multi-core systems |

---

## 🔑 Key Takeaways

- `top` is the first tool to use — press `P` to sort by CPU and see real-time usage.
- `ps aux --sort=-%cpu` gives a quick **non-interactive snapshot** — perfect for scripts.
- **`%CPU` in `ps` is a lifetime average** — use `top` or `pidstat` for current CPU usage.
- `mpstat -P ALL` reveals **single-core bottlenecks** that aggregate CPU % hides.
- Use `renice` to **lower process priority** rather than killing — less disruptive for production.
- High **`wa` (I/O wait)** in `top` means the bottleneck is disk, not CPU — don't just blame the process.
- `pidstat -p PID 1` tracks CPU usage of **one process per second** — the most accurate per-process metric.

---

## 📚 Related Concepts

- [GNU ps Manual](https://linux.die.net/man/1/ps)
- [sysstat (pidstat, mpstat)](https://github.com/sysstat/sysstat)

</details>
