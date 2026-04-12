# Challenge 27: Check System Uptime in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**System uptime** is how long the system has been running since its last boot. Monitoring uptime is important for:

- Verifying a server reboot actually occurred (or did not occur unexpectedly)
- Tracking system stability — high uptime indicates a stable system
- Checking SLA (Service Level Agreement) compliance for servers
- Diagnosing unexpected reboots caused by kernel panics or power failures
- Understanding system load trends over time

Linux provides several tools to check uptime — `uptime`, `w`, `/proc/uptime`, and `systemd-analyze`.

---

## 🧩 Task

Check how long the system has been running since its last reboot.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Quick uptime with load averages
uptime

# Human-readable uptime
uptime -p

# Last boot time
who -b
```

### How it works

| Command | Description |
|---------|-------------|
| `uptime` | Shows current time, uptime, users, and load averages |
| `uptime -p` | Shows uptime in **pretty** human-readable format |
| `who -b` | Shows the **last boot time** |

---

## 🔢 Understanding `uptime` Output

```bash
uptime
```

```
 10:35:22 up 5 days,  3:14,  2 users,  load average: 0.45, 0.62, 0.71
```

| Field | Example | Description |
|-------|---------|-------------|
| Current time | `10:35:22` | Current system clock time |
| Uptime | `5 days, 3:14` | How long since last reboot |
| Users | `2 users` | Number of currently logged-in users |
| Load (1 min) | `0.45` | Average load over last 1 minute |
| Load (5 min) | `0.62` | Average load over last 5 minutes |
| Load (15 min) | `0.71` | Average load over last 15 minutes |

> 💡 Load average represents the average number of processes waiting for CPU. A value above the number of CPU cores means the system is overloaded.

---

## 📖 Extended Examples

### Example 1 — Basic Uptime

```bash
uptime
```

```bash
 10:35:22 up 5 days,  3:14,  2 users,  load average: 0.45, 0.62, 0.71
```

---

### Example 2 — Human-Readable Format

```bash
uptime -p
```

```bash
up 5 days, 3 hours, 14 minutes
```

> 💡 `-p` (pretty) gives a clean, easy-to-read format — great for scripts and reports.

---

### Example 3 — Show Last Boot Time

```bash
# Show when the system last booted
who -b
```

```bash
         system boot  2025-03-16 07:21
```

```bash
# Using uptime with -s flag
uptime -s
```

```bash
2025-03-16 07:21:35
```

> 💡 `uptime -s` shows the exact date and time of the last boot — more precise than `who -b`.

---

### Example 4 — Check Uptime with `w`

```bash
w
```

```bash
 10:35:22 up 5 days,  3:14,  2 users,  load average: 0.45, 0.62, 0.71
USER     TTY      FROM             LOGIN@   IDLE WHAT
devuser  pts/0    192.168.1.5      10:00    0.00s w
adminuser pts/1   10.0.0.5         09:45    5:10  bash
```

> The first line of `w` output is identical to `uptime` — it shows uptime plus who is logged in and what they are doing.

---

### Example 5 — Check Raw Uptime from `/proc`

```bash
# Raw uptime in seconds since boot
cat /proc/uptime
```

```bash
456054.23 1821832.47
```

| Column | Description |
|--------|-------------|
| `456054.23` | Seconds the system has been up |
| `1821832.47` | Seconds the CPU(s) have been idle |

```bash
# Convert to human-readable
awk '{print int($1/86400)"d "int($1%86400/3600)"h "int($1%3600/60)"m"}' /proc/uptime
```

```bash
5d 6h 47m
```

---

### Example 6 — Check Last Boot with `last reboot`

```bash
# Show all reboots from history
last reboot
```

```bash
reboot   system boot  5.15.0-generic   Mon Mar 16 07:21   still running
reboot   system boot  5.15.0-generic   Wed Mar 12 09:05 - 07:21 (3+22:15)
reboot   system boot  5.15.0-generic   Mon Mar 10 14:30 - 09:05 (1+18:34)

wtmp begins Mon Mar 10 14:30:00 2025
```

> 💡 `last reboot` shows the **history of all reboots** — invaluable for investigating unexpected restarts.

---

### Example 7 — Check Uptime with `systemd-analyze`

```bash
# Show boot time breakdown
systemd-analyze

# Show time spent in each phase
systemd-analyze blame | head -10

# Show a visual timeline
systemd-analyze plot > boot_timeline.svg
```

```bash
# systemd-analyze output:
Startup finished in 3.521s (kernel) + 8.431s (userspace) = 11.952s
graphical.target reached after 8.283s in userspace.
```

```bash
# systemd-analyze blame (slowest services first):
         4.201s NetworkManager-wait-online.service
         1.832s snapd.service
         1.124s apt-daily-upgrade.service
         0.891s dev-sda1.device
```

---

### Example 8 — Monitor Uptime in Real Time

```bash
# Refresh uptime every 5 seconds
watch -n 5 uptime

# Watch with full screen update
watch -n 1 'uptime && echo && w'
```

---

### Example 9 — Uptime in a Script

```bash
#!/bin/bash

# Get uptime components
UPTIME_SECONDS=$(awk '{print int($1)}' /proc/uptime)
DAYS=$((UPTIME_SECONDS / 86400))
HOURS=$(( (UPTIME_SECONDS % 86400) / 3600 ))
MINUTES=$(( (UPTIME_SECONDS % 3600) / 60 ))

echo "System Uptime: ${DAYS}d ${HOURS}h ${MINUTES}m"
echo "Last Boot: $(uptime -s)"
echo "Load Average: $(uptime | awk -F'load average:' '{print $2}')"

# Alert if system was rebooted recently (within last hour)
if [ "$UPTIME_SECONDS" -lt 3600 ]; then
  echo "⚠️  WARNING: System rebooted less than 1 hour ago!"
fi
```

```bash
System Uptime: 5d 3h 14m
Last Boot: 2025-03-16 07:21:35
Load Average:  0.45, 0.62, 0.71
```

---

### Example 10 — Check Uptime Across Multiple Servers

```bash
# Check uptime on multiple remote servers
for SERVER in server1 server2 server3; do
  echo -n "$SERVER: "
  ssh "$SERVER" uptime -p 2>/dev/null || echo "unreachable"
done
```

```bash
server1: up 5 days, 3 hours, 14 minutes
server2: up 12 days, 7 hours, 42 minutes
server3: unreachable
```

---

## 🗂️ Uptime Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `uptime` | Uptime + load averages |
| `uptime -p` | Human-readable uptime only |
| `uptime -s` | Last boot date and time |
| `who -b` | Last boot time |
| `w` | Uptime + logged-in users + activity |
| `last reboot` | Full history of all reboots |
| `cat /proc/uptime` | Raw uptime in seconds |
| `systemd-analyze` | Boot time breakdown |

---

## 🔢 Load Average Interpretation

Load average only makes sense relative to the **number of CPU cores**:

```bash
# Check CPU core count
nproc
```

| Load / Core Ratio | Status |
|-------------------|--------|
| < 0.7 | ✅ Healthy — spare capacity |
| 0.7 – 1.0 | ⚠️ Getting busy |
| 1.0 – 1.5 | 🟠 Overloaded — investigate |
| > 1.5 | 🔴 Seriously overloaded |

---

## 🚀 Full Workflow — Investigate an Unexpected Reboot

```bash
# Step 1: Check current uptime
uptime

# Step 2: Find when the last reboot happened
uptime -s
who -b

# Step 3: See all recent reboots
last reboot | head -10

# Step 4: Check system logs around the reboot time
journalctl --since "2025-03-16 07:15:00" --until "2025-03-16 07:25:00"

# Step 5: Look for kernel panic or OOM before reboot
journalctl -b -1 | grep -i "panic\|oom\|killed\|error" | tail -20

# Step 6: Check if a scheduled reboot was planned
last reboot
grep -i "reboot\|shutdown" /var/log/syslog | tail -20

# Step 7: Check uptime of key services
systemctl status nginx mysql sshd | grep -E "Active|since"
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Confusing load average with CPU % | Load avg > core count = overloaded; it is not a percentage |
| High uptime is always good | Very high uptime (years) may mean missing security patches |
| `uptime -p` not available | Older systems may not support `-p` — use `awk` on `/proc/uptime` |
| Unexpected low uptime | Check `last reboot` for unplanned restarts and journal for causes |

---

## 🔑 Key Takeaways

- `uptime` shows how long the system has been running plus **load averages** for the last 1, 5, and 15 minutes.
- `uptime -p` gives a **clean human-readable** format — perfect for scripts and reports.
- `uptime -s` shows the **exact date and time of the last boot**.
- `last reboot` shows the **full history of all reboots** — use it to detect unexpected restarts.
- Load average only makes sense relative to **CPU core count** — check with `nproc`.
- `/proc/uptime` provides the **raw seconds** since boot — useful for scripting and precise calculations.
- High uptime is desirable, but very long uptimes may indicate missing security updates.

---

## 📚 Related Concepts

- [GNU uptime Manual](https://www.gnu.org/software/coreutils/manual/html_node/uptime-invocation.html)
- [systemd-analyze Manual](https://www.freedesktop.org/software/systemd/man/systemd-analyze.html)

</details>
