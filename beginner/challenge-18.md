# Challenge 18: View Running Processes in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

A **process** is any program currently running on the Linux system — from your terminal session to web servers, database engines, and background daemons. Viewing and understanding running processes is one of the most fundamental Linux skills.

Knowing what is running helps you:
- Monitor system resource usage
- Find and stop runaway processes
- Verify services are running correctly
- Identify suspicious or unauthorized processes
- Understand parent-child process relationships

---

## 🧩 Task

List all running processes on the system.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Snapshot of all running processes
ps aux

# Interactive real-time process monitor
top
```

### How it works

| Command | Description |
|---------|-------------|
| `ps aux` | **P**rocess **S**tatus — snapshot of all running processes |
| `top` | Real-time interactive process monitor |

---

## 🔢 Understanding `ps aux` Output

```bash
ps aux
```

```
USER       PID  %CPU  %MEM    VSZ   RSS  TTY   STAT  START   TIME  COMMAND
root         1   0.0   0.1  16952  5612  ?     Ss    Mar20   0:05  /sbin/init
devuser   4821   0.0   0.3  25600  8192  pts/0 S+    10:00   0:00  bash
root      5201   0.2   0.8 456000 32000  ?     Ssl   09:00   1:23  /usr/sbin/nginx
mysql     2341  14.3   2.0 1892000 324000 ?    Sl    08:00   5:21  /usr/sbin/mysqld
```

| Column | Description |
|--------|-------------|
| `USER` | Owner of the process |
| `PID` | **Process ID** — unique identifier |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | **Resident Set Size** — actual RAM used (KB) |
| `TTY` | Terminal — `?` means no terminal (daemon) |
| `STAT` | Process state — see table below |
| `START` | When the process started |
| `TIME` | Total CPU time consumed |
| `COMMAND` | The command that launched the process |

### Process State Codes (STAT)

| State | Meaning |
|-------|---------|
| `R` | **Running** — actively using CPU |
| `S` | **Sleeping** — waiting for an event |
| `D` | **Uninterruptible sleep** — waiting for I/O |
| `T` | **Stopped** — suspended |
| `Z` | **Zombie** — finished but not reaped |
| `s` | Session leader |
| `+` | Foreground process group |
| `l` | Multi-threaded |
| `<` | High priority |
| `N` | Low priority (nice) |

---

## 📖 Extended Examples

### Example 1 — List All Processes

```bash
ps aux
```

---

### Example 2 — Show Processes in a Tree Format

```bash
# Show parent-child relationships
ps axjf

# Or using pstree
pstree -p
```

```bash
# pstree output:
systemd(1)─┬─sshd(801)───sshd(4821)───bash(4822)───ps(5201)
           ├─nginx(5100)─┬─nginx(5101)
           │             └─nginx(5102)
           └─mysqld(2341)
```

---

### Example 3 — Custom Output Columns

```bash
# Show specific columns: PID, parent PID, user, CPU, memory, command
ps -eo pid,ppid,user,%cpu,%mem,cmd

# Sort by CPU usage
ps -eo pid,user,%cpu,cmd --sort=-%cpu | head -10

# Sort by memory usage
ps -eo pid,user,%mem,rss,cmd --sort=-%mem | head -10
```

---

### Example 4 — Find a Specific Process

```bash
# Find all nginx processes
ps aux | grep nginx

# Cleaner — avoid matching the grep itself
ps aux | grep "[n]ginx"

# Get just the PID
pgrep nginx

# Get PID and process name
pgrep -la nginx
```

```bash
# pgrep -la output:
5100 nginx: master process /usr/sbin/nginx
5101 nginx: worker process
5102 nginx: worker process
```

---

### Example 5 — Show Processes for a Specific User

```bash
# Show all processes for a specific user
ps -u devuser

# Show all processes for root
ps -u root

# Using aux with grep
ps aux | grep "^devuser"
```

---

### Example 6 — Interactive `top` View

```bash
top
```

```bash
top - 10:35:22 up 3 days, 1:35, 2 users, load average: 0.45, 0.42, 0.38
Tasks: 215 total,   1 running, 214 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 92.8 id,  0.5 wa,  0.0 hi,  0.2 si
MiB Mem : 15,820.0 total,  5,812.0 free,  9,341.0 used,    667.0 buff/cache
MiB Swap:  2,048.0 total,  2,048.0 free,      0.0 used. 6,021.0 avail Mem

  PID USER      PR  NI    VIRT    RES  %CPU  %MEM     TIME+ COMMAND
12345 appuser   20   0 2145000 512000  85.2   3.2  45:12.56 python3
 2341 mysql     20   0 1892000 324000  14.3   2.0  15:21.10 mysqld
```

**Useful `top` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `P` | Sort by **CPU** usage |
| `M` | Sort by **Memory** usage |
| `k` | **Kill** a process (enter PID) |
| `r` | **Renice** a process |
| `q` | **Quit** |
| `1` | Toggle per-**core** CPU view |
| `u` | Filter by **user** |
| `f` | Manage displayed **fields** |

---

### Example 7 — Interactive `htop` (Enhanced top)

```bash
htop
```

```bash
# Install if not available
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL/CentOS
```

| Feature | `top` | `htop` |
|---------|-------|--------|
| Color display | Basic | ✅ Full color |
| Mouse support | ❌ | ✅ |
| Tree view | ❌ | ✅ |
| Per-CPU bars | ❌ | ✅ |
| Scroll | Limited | ✅ |
| Kill without PID | ❌ | ✅ |

---

### Example 8 — Count Total Running Processes

```bash
# Total number of processes
ps aux | wc -l

# Subtract 1 for the header line
echo "Running processes: $(($(ps aux | wc -l) - 1))"

# Using nproc for CPU info
nproc
```

---

### Example 9 — Show Processes with Full Command Line

```bash
# Default ps may truncate long commands
ps aux

# Show full command line (no truncation)
ps auxww

# Show command line for a specific PID
cat /proc/4821/cmdline | tr '\0' ' '
```

---

### Example 10 — Monitor a Specific Process

```bash
# Watch a specific process every 2 seconds
watch -n 2 'ps -p 4821 -o pid,user,%cpu,%mem,cmd'

# Use top for just one process
top -p 4821

# Monitor multiple PIDs
top -p 4821,2341,5201
```

---

### Example 11 — Show Process Start Time and Duration

```bash
# Show PID, start time, elapsed time, and command
ps -eo pid,lstart,etime,cmd | head -10
```

```bash
  PID                  STARTED     ELAPSED CMD
    1 Thu Mar 18 08:00:01 2025 3-02:35:21 /sbin/init
  801 Thu Mar 18 08:00:05 2025 3-02:35:17 /usr/sbin/sshd
 2341 Thu Mar 18 08:00:10 2025 3-02:35:12 /usr/sbin/mysqld
```

---

### Example 12 — Find and Kill a Process

```bash
# Find process ID
pgrep nginx

# Kill gracefully (SIGTERM)
kill -15 5100
# or
kill -SIGTERM 5100

# Kill forcefully (SIGKILL)
kill -9 5100

# Kill by name
pkill nginx

# Kill all processes for a user
pkill -u baduser
```

---

## 🗂️ `ps` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `a` | Show processes for **all users** |
| `u` | **User-oriented** format with extra columns |
| `x` | Include processes **without a terminal** |
| `-e` | Show **every** process |
| `-o` | Specify **custom output** columns |
| `-p PID` | Show a **specific PID** |
| `-u USER` | Show processes for a **specific user** |
| `w` / `ww` | **Wide** output — don't truncate command |
| `f` | **Forest** — show process tree |
| `--sort=FIELD` | **Sort** by a column |

---

## 🔄 Process Monitoring Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `ps aux` | Snapshot | Quick sorted list, scripting |
| `top` | Interactive | Real-time overview |
| `htop` | Interactive | Visual, color-coded, mouse support |
| `pgrep` | Search | Find PIDs by name |
| `pstree` | Tree view | Parent-child relationships |
| `pidstat` | Sampled | Per-process CPU/memory over time |

---

## 🚀 Full Workflow — Find and Manage a Process

```bash
# Step 1: View all running processes
ps aux

# Step 2: Find a specific process
ps aux | grep nginx
pgrep -la nginx

# Step 3: Check resource usage
ps -p 5100 -o pid,user,%cpu,%mem,rss,cmd

# Step 4: Monitor it in real time
top -p 5100

# Step 5: Check what files it has open
sudo lsof -p 5100 | head -10

# Step 6: Check its network connections
sudo ss -tp | grep 5100

# Step 7: Kill if needed (graceful first)
kill -15 5100
sleep 5
kill -9 5100    # Force if still running

# Step 8: Verify it is gone
pgrep nginx || echo "Process stopped"
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `ps aux \| grep process` shows grep itself | Use `ps aux \| grep "[p]rocess"` or `pgrep process` |
| `%CPU` in `ps` shows lifetime average | Use `top` or `pidstat` for current CPU usage |
| Killing parent kills all children | Use `pgrep` to find and inspect the full process tree first |
| `kill` with wrong PID | Always verify PID with `ps -p PID` before killing |
| `ps aux` truncates long commands | Use `ps auxww` for full command display |

---

## 🔑 Key Takeaways

- `ps aux` gives a **complete snapshot** of all running processes — the most used command for process inspection.
- `top` provides **real-time interactive monitoring** — press `P` to sort by CPU, `M` for memory.
- **Process state** (`STAT` column) tells you what each process is doing — `D` state (I/O wait) is often a sign of disk problems.
- `pgrep -la name` finds processes **by name** and returns their PIDs.
- Always use `ps auxww` for **full command line** — default `ps` truncates long commands.
- Kill processes gracefully with `SIGTERM (-15)` first — only use `SIGKILL (-9)` if the process ignores SIGTERM.
- `htop` is the most beginner-friendly process monitor — color-coded, mouse-aware, and interactive.

---

## 📚 Related Concepts

- [GNU ps Manual](https://linux.die.net/man/1/ps)
- [top Manual](https://linux.die.net/man/1/top)

</details>
