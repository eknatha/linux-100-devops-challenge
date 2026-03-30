# Challenge 71: Diagnose High CPU Usage in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

High CPU usage can slow a system to a crawl, cause service timeouts, and degrade user experience. Quickly identifying **which processes are consuming the most CPU** and understanding **why** is a critical sysadmin and DevOps skill.

Linux provides several powerful tools — `top`, `ps`, `htop`, `mpstat`, and `pidstat` — each suited for different depths of investigation, from instant snapshots to sustained monitoring and per-core analysis.

---

## 🧩 Task

Identify which processes are causing high CPU usage on the system.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Interactive real-time view
top

# Sorted snapshot — top CPU consumers
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

### How it works

#### `top`

| Key | Description |
|-----|-------------|
| `top` | Launches an interactive, real-time process monitor |
| Press `P` | Sort by CPU usage (default) |
| Press `M` | Sort by memory usage |
| Press `k` | Kill a process by PID |
| Press `q` | Quit |

#### `ps -eo pid,cmd,%cpu --sort=-%cpu | head`

| Part | Description |
|------|-------------|
| `ps` | Process status snapshot |
| `-e` | List **all** processes |
| `-o pid,cmd,%cpu` | Output only PID, command, and CPU% columns |
| `--sort=-%cpu` | Sort by CPU usage **descending** (highest first) |
| `\| head` | Show only the top 10 results |

---

## 📖 Extended Examples

### Example 1 — Interactive `top` View

```bash
top
```

```bash
top - 10:35:22 up 5 days,  3:17,  2 users,  load average: 3.42, 2.81, 2.10
Tasks: 215 total,   3 running, 212 sleeping,   0 stopped,   0 zombie
%Cpu(s): 78.2 us, 10.3 sy,  0.0 ni, 10.1 id,  1.2 wa,  0.0 hi,  0.2 si
MiB Mem : 15,820.0 total,   4,212.0 free,   9,341.0 used,   2,267.0 buff/cache

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
12345 appuser   20   0 2145000 512000  12000 R  85.2   3.2  12:34.56 python3
 2341 mysql     20   0 1892000 324000   8000 S  14.3   2.0   5:21.10 mysqld
 5201 www-data  20   0  456000  32000   4000 S   2.1   0.2   0:45.20 nginx
```

#### Understanding `top` CPU Line

```
%Cpu(s): 78.2 us, 10.3 sy,  0.0 ni, 10.1 id,  1.2 wa
          ^^^^     ^^^^              ^^^^       ^^^^
          User     System            Idle       I/O wait
```

| Field | Meaning |
|-------|---------|
| `us` | User space — application CPU time |
| `sy` | Kernel/system CPU time |
| `ni` | Nice — CPU time for processes with adjusted priority |
| `id` | Idle — `100 - id` = total CPU busy percentage |
| `wa` | I/O wait — high value indicates disk/network bottleneck |
| `hi` | Hardware interrupt handling |
| `si` | Software interrupt handling |
| `st` | Steal — CPU stolen by hypervisor (relevant in VMs) |

---

### Example 2 — Sorted Snapshot with `ps`

```bash
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

```bash
  PID CMD                         %CPU
12345 python3 /opt/app/server.py  85.2
 2341 /usr/sbin/mysqld             14.3
 5201 nginx: worker process         2.1
    1 /sbin/init                    0.2
  801 /usr/sbin/sshd                0.1
```

---

### Example 3 — Extended `ps` with More Columns

```bash
ps -eo pid,ppid,user,cmd,%cpu,%mem --sort=-%cpu | head -15
```

```bash
  PID  PPID USER     CMD                         %CPU  %MEM
12345  1200 appuser  python3 /opt/app/server.py  85.2   3.2
 2341     1 mysql    /usr/sbin/mysqld             14.3   2.0
 5201  5200 www-data nginx: worker process         2.1   0.2
```

---

### Example 4 — One-liner: CPU Usage Over 10%

```bash
ps -eo pid,user,cmd,%cpu --sort=-%cpu | awk 'NR==1 || $4 > 10'
```

```bash
  PID USER     CMD                         %CPU
12345 appuser  python3 /opt/app/server.py  85.2
 2341 mysql    /usr/sbin/mysqld             14.3
```

> 💡 `awk 'NR==1 || $4 > 10'` keeps the header row (`NR==1`) and any process consuming more than 10% CPU.

---

### Example 5 — Per-Core CPU Usage with `mpstat`

```bash
# Install sysstat if needed
sudo dnf install sysstat     # RHEL
sudo apt install sysstat     # Debian/Ubuntu

# Per-core CPU usage sampled every 1 second, 5 times
mpstat -P ALL 1 5
```

```bash
10:35:22  CPU    %usr   %sys   %iow   %idle
10:35:23  all   78.2   10.3    1.2    10.1
10:35:23    0   95.4   3.2     0.1     1.3
10:35:23    1   82.1   5.4     0.2    12.3
10:35:23    2   71.0   8.1     0.8    20.1
10:35:23    3   64.3  24.5     3.3     7.9
```

> 💡 `mpstat -P ALL` reveals if load is distributed evenly across cores or if one core is maxed while others are idle — which can indicate a single-threaded bottleneck.

---

### Example 6 — Per-Process CPU Over Time with `pidstat`

```bash
# Monitor specific PID every second
pidstat -p 12345 1

# Monitor all processes, refresh every 2 seconds
pidstat 2

# Monitor processes above a CPU threshold
pidstat -p ALL 1 | awk '$8 > 10'
```

```bash
10:35:22   UID  PID   %usr  %system  %CPU  CPU  Command
10:35:23  1001 12345  84.0    1.2   85.2    2  python3
10:35:24  1001 12345  83.5    1.5   85.0    2  python3
10:35:25  1001 12345  85.1    1.1   86.2    2  python3
```

> 💡 `pidstat` is ideal for **tracking a specific process over time** to see if CPU usage is steady, spiking, or trending up.

---

### Example 7 — Investigate a High-CPU Process

```bash
# Step 1: Get the PID
HIGH_CPU_PID=12345

# Step 2: See the full command line
cat /proc/$HIGH_CPU_PID/cmdline | tr '\0' ' '

# Step 3: See which files it has open
sudo lsof -p $HIGH_CPU_PID | head -20

# Step 4: See which network connections it has
sudo ss -tp | grep $HIGH_CPU_PID

# Step 5: Check the process's own resource limits
cat /proc/$HIGH_CPU_PID/limits

# Step 6: Thread-level CPU breakdown (useful for multi-threaded apps)
ps -p $HIGH_CPU_PID -L -o pid,tid,pcpu,comm
```

---

### Example 8 — Use `htop` for Visual Investigation

```bash
htop
```

**Useful `htop` shortcuts:**

| Key | Action |
|-----|--------|
| `P` | Sort by CPU |
| `M` | Sort by memory |
| `T` | Show process tree |
| `F4` | Filter by process name |
| `F9` | Send signal (kill) to process |
| `u` | Filter by user |
| `/` | Search for a process |

> `htop` shows per-core CPU meters at the top — immediately visible if a single core is maxed while others are idle.

---

### Example 9 — CPU Usage Summary with `sar`

```bash
# CPU usage sampled every 1 second, 10 times
sar -u 1 10

# Historical CPU usage for today
sar -u

# CPU usage for a specific hour from logs
sar -u -f /var/log/sa/sa21 --start-time 09:00:00 --end-time 10:00:00
```

```bash
Average:  %user  %system  %iowait  %idle
Average:   78.2    10.3      1.2    10.1
```

> 💡 `sar` is invaluable for investigating CPU spikes that happened in the past — before you were alerted.

---

### Example 10 — CPU Usage Monitoring Script with Alert

```bash
#!/bin/bash

THRESHOLD=80          # Alert if any process exceeds this CPU %
LOGFILE="/var/log/cpu_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "[$DATE] Checking CPU usage..." >> "$LOGFILE"

ps -eo pid,user,cmd,%cpu --sort=-%cpu | awk -v t="$THRESHOLD" -v date="$DATE" '
NR > 1 && $4 > t {
  print "[" date "] ⚠️  HIGH CPU: PID=" $1 " USER=" $2 " CPU=" $4 "% CMD=" $3
}' | tee -a "$LOGFILE"

# Optional: send email alert
# | mail -s "High CPU Alert on $(hostname)" admin@example.com
```

---

### Example 11 — Kill or Renice a High-CPU Process

```bash
# Option 1: Graceful termination
kill -SIGTERM 12345

# Option 2: Force kill
kill -SIGKILL 12345

# Option 3: Lower the process priority (renice) instead of killing
# nice values: -20 (highest priority) to 19 (lowest priority)
sudo renice +15 -p 12345

# Option 4: Limit CPU using cpulimit
sudo apt install cpulimit    # Debian/Ubuntu
sudo cpulimit -p 12345 -l 30   # Limit to max 30% CPU
```

| Action | Command | Effect |
|--------|---------|--------|
| Graceful stop | `kill -SIGTERM PID` | Asks process to stop cleanly |
| Force stop | `kill -SIGKILL PID` | Immediately terminates |
| Lower priority | `renice +15 -p PID` | Reduces CPU scheduling priority |
| Throttle CPU | `cpulimit -p PID -l 30` | Hard limit to 30% CPU max |

---

## 🛠️ CPU Diagnostic Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `top` | Interactive | Real-time overview of all processes |
| `htop` | Interactive (visual) | Visual per-core CPU + process tree |
| `ps` | Snapshot | Scripting and one-time sorted lists |
| `mpstat` | Sampled | Per-core CPU distribution |
| `pidstat` | Sampled | Tracking one process over time |
| `sar` | Historical | Investigating past CPU spikes |
| `perf` | Profiling | Deep-dive: what code is the CPU running |
| `strace` | Tracing | What syscalls is the process making |

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

## 🚀 Full Workflow — Diagnose and Resolve High CPU

```bash
# Step 1: Get an instant overview
top
# Press P to sort by CPU — identify the top offender

# Step 2: Confirm with a non-interactive snapshot
ps -eo pid,user,cmd,%cpu --sort=-%cpu | head -10

# Step 3: Check if it's CPU-bound or I/O-bound
vmstat 1 5
# High 'wa' = I/O bottleneck; high 'us' = application CPU

# Step 4: Check per-core distribution
mpstat -P ALL 1 3

# Step 5: Investigate the specific process
HIGH_PID=12345
cat /proc/$HIGH_PID/cmdline | tr '\0' ' '
sudo lsof -p $HIGH_PID | head

# Step 6: Decide on action
# If it's a legitimate workload — monitor and optimize
# If it's a runaway process — renice or kill

sudo renice +15 -p $HIGH_PID     # Throttle priority
# OR
kill -SIGTERM $HIGH_PID           # Graceful stop
# OR
kill -SIGKILL $HIGH_PID           # Force stop

# Step 7: Verify CPU normalizes
top
# Check 'id' (idle) has recovered
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `ps %cpu` shows 0 for active processes | `ps` shows average since process start — use `top` or `pidstat` for current usage |
| High load but low CPU in `top` | Check `wa` (I/O wait) — disk or network may be the real bottleneck |
| Single core maxed, others idle | Single-threaded process — `mpstat -P ALL` reveals this |
| Killing a critical service accidentally | Always check the process name and owner before killing |
| High `sy` (system CPU) | Kernel-level bottleneck — check interrupt rates with `mpstat` or `sar -I ALL` |
| `sar` shows no historical data | Enable sysstat: `sudo systemctl enable --now sysstat` |

---

## 🔑 Key Takeaways

- `top` is the fastest interactive tool — press `P` to immediately sort by CPU.
- `ps -eo pid,cmd,%cpu --sort=-%cpu | head` gives a clean non-interactive snapshot for scripting.
- `ps %cpu` shows the **lifetime average** — use `top` or `pidstat` for **current** CPU usage.
- High `wa` (I/O wait) means the bottleneck is disk/network, not CPU — don't be misled by load averages.
- `mpstat -P ALL` reveals **single-core bottlenecks** that a combined CPU% would hide.
- Use `renice` to throttle a runaway process without killing it — less disruptive for production.
- `sar` enables **forensic analysis** of past CPU spikes when you're investigating after the fact.

---

## 📚 Related Concepts

- [Challenge 68 — Check System Load]
- [Challenge 69 — Identify Zombie Processes]
- [Challenge 70 — Debug Failed Services]
- [perf](https://perf.wiki.kernel.org/) — Linux performance profiling tool
- [strace](https://man7.org/linux/man-pages/man1/strace.1.html) — trace system calls made by a process
- [sysstat tools](https://github.com/sysstat/sysstat) — `sar`, `pidstat`, `mpstat`, `iostat`


</details>
