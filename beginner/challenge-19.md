# Challenge 19: Kill a Process in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Killing a process** means sending it a **signal** that tells it to stop. In Linux, processes do not just get abruptly deleted — they receive a signal and react to it. Some signals ask politely (SIGTERM), others force immediate termination (SIGKILL), and others serve different purposes like reloading configuration (SIGHUP).

Understanding how to correctly terminate processes — gracefully first, forcefully when necessary — is a critical Linux skill for every sysadmin and developer.

---

## 🧩 Task

Terminate a running process by its PID or name.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Kill by PID — graceful (SIGTERM)
kill PID

# Kill by PID — forceful (SIGKILL)
kill -9 PID

# Kill by process name
pkill process_name

# Kill all instances by name — forceful
pkill -9 process_name
```

### How it works

| Command | Description |
|---------|-------------|
| `kill PID` | Send **SIGTERM** (signal 15) — ask the process to stop gracefully |
| `kill -9 PID` | Send **SIGKILL** (signal 9) — force immediate termination |
| `pkill name` | Kill processes by **name** instead of PID |
| `killall name` | Kill **all** processes with a matching name |

---

## 🔢 Linux Signals Reference

Every `kill` command sends a **signal** to a process. The most important signals are:

| Signal | Number | Name | Description |
|--------|--------|------|-------------|
| `SIGHUP` | 1 | Hangup | Reload configuration — process stays running |
| `SIGINT` | 2 | Interrupt | Same as `Ctrl+C` — polite stop |
| `SIGQUIT` | 3 | Quit | Quit and produce core dump |
| `SIGKILL` | 9 | Kill | **Force kill** — cannot be caught or ignored |
| `SIGTERM` | 15 | Terminate | **Graceful stop** — default `kill` signal |
| `SIGUSR1` | 10 | User 1 | Application-defined (e.g., nginx log reopen) |
| `SIGUSR2` | 12 | User 2 | Application-defined |
| `SIGSTOP` | 19 | Stop | **Pause** — suspends the process |
| `SIGCONT` | 18 | Continue | **Resume** a stopped process |

> 💡 **Always try SIGTERM first** — it allows the process to clean up, close files, and flush data. Use SIGKILL only if SIGTERM is ignored.

---

## 📖 Extended Examples

### Example 1 — Find a Process ID First

```bash
# Find PID by process name
pgrep nginx

# Find PID with full details
pgrep -la nginx

# Using ps
ps aux | grep nginx

# Using pidof
pidof nginx
```

```bash
# pgrep -la output:
5100 nginx: master process /usr/sbin/nginx
5101 nginx: worker process
5102 nginx: worker process
```

---

### Example 2 — Graceful Kill (SIGTERM — Default)

```bash
# Send SIGTERM — asks the process to stop cleanly
kill 5100

# Equivalent — explicitly specifying signal 15
kill -15 5100
kill -SIGTERM 5100
```

> ✅ SIGTERM gives the process time to:
> - Save data to disk
> - Close open files and connections
> - Clean up temporary files
> - Release locks

---

### Example 3 — Force Kill (SIGKILL)

```bash
# Force kill — cannot be caught or ignored
kill -9 5100
kill -SIGKILL 5100
```

> ⚠️ SIGKILL:
> - Terminates **immediately** — no cleanup
> - The kernel handles it directly
> - May leave **temp files**, **locks**, or **corrupted data** behind
> - Use only when SIGTERM fails

---

### Example 4 — The Safe Kill Pattern

```bash
PID=5100

# Step 1: Try graceful termination
kill -15 $PID

# Step 2: Wait a few seconds
sleep 5

# Step 3: Check if still running
if kill -0 $PID 2>/dev/null; then
  echo "Process still alive — force killing..."
  kill -9 $PID
else
  echo "Process terminated gracefully"
fi
```

---

### Example 5 — Kill by Process Name with `pkill`

```bash
# Kill all nginx processes by name
pkill nginx

# Force kill by name
pkill -9 nginx

# Kill with signal by name
pkill -SIGTERM nginx

# Kill only processes matching a full command pattern
pkill -f "python3 /opt/app/server.py"
```

> 💡 `pkill` is more convenient than `kill` when you don't know the PID — it finds and kills by name.

---

### Example 6 — Kill All Instances with `killall`

```bash
# Kill all processes named nginx
killall nginx

# Force kill all
killall -9 nginx

# Interactive — confirm before killing each
killall -i nginx

# Kill all processes owned by a specific user
pkill -u baduser
```

---

### Example 7 — Reload Configuration (SIGHUP)

```bash
# nginx — reload config without downtime
kill -HUP $(cat /run/nginx.pid)
# or
kill -1 $(pgrep nginx | head -1)

# sshd — reload config
kill -HUP $(pgrep sshd | head -1)

# Using systemctl (easier)
sudo systemctl reload nginx
```

> 💡 SIGHUP (signal 1) tells many daemons to **reload their configuration** without restarting — no downtime.

---

### Example 8 — Pause and Resume a Process

```bash
# Pause (suspend) a process
kill -STOP 5100
# or
kill -19 5100

# Resume a stopped process
kill -CONT 5100
# or
kill -18 5100
```

> 💡 Stopping a process with `SIGSTOP` is like pressing pause — it freezes without killing. Use SIGCONT to resume.

---

### Example 9 — Kill Processes Using `top` Interactively

```bash
top
```

Inside `top`:
1. Press `k`
2. Enter the PID when prompted
3. Enter the signal number (default: 15 for SIGTERM)

```bash
# top kill prompt:
PID to signal/kill [default pid = 5100]:
Signal/Kill [15]:
```

---

### Example 10 — Kill Processes in `htop`

```bash
htop
```

Inside `htop`:
1. Navigate to the process using arrow keys
2. Press `F9` to open the kill menu
3. Select the signal and press Enter

> `htop` is the most beginner-friendly way to kill a process — visual, interactive, and shows signal names.

---

### Example 11 — Kill All Processes from a Specific Terminal

```bash
# Kill all processes on pts/1
pkill -t pts/1

# Disconnect a user's session
pkill -KILL -t pts/2
```

---

### Example 12 — Script to Safely Stop a Service Process

```bash
#!/bin/bash

SERVICE="myapp"
PIDFILE="/var/run/myapp.pid"
TIMEOUT=10

if [ -f "$PIDFILE" ]; then
  PID=$(cat "$PIDFILE")
  echo "Stopping $SERVICE (PID: $PID)..."

  # Graceful stop
  kill -SIGTERM "$PID" 2>/dev/null

  # Wait up to TIMEOUT seconds
  COUNT=0
  while kill -0 "$PID" 2>/dev/null && [ $COUNT -lt $TIMEOUT ]; do
    sleep 1
    ((COUNT++))
    echo "Waiting... ($COUNT/$TIMEOUT)"
  done

  # Force kill if still alive
  if kill -0 "$PID" 2>/dev/null; then
    echo "⚠️  Force killing $SERVICE..."
    kill -SIGKILL "$PID"
  else
    echo "✅ $SERVICE stopped cleanly"
  fi

  rm -f "$PIDFILE"
else
  echo "No PID file found — $SERVICE may not be running"
fi
```

---

## 🗂️ Kill Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `kill PID` | Send SIGTERM (graceful stop) |
| `kill -9 PID` | Send SIGKILL (force stop) |
| `kill -1 PID` | Send SIGHUP (reload config) |
| `kill -19 PID` | Send SIGSTOP (pause) |
| `kill -18 PID` | Send SIGCONT (resume) |
| `kill -0 PID` | Check if process exists (no signal sent) |
| `pkill name` | Kill by process name |
| `pkill -f pattern` | Kill by full command pattern |
| `pkill -u user` | Kill all processes by user |
| `pkill -t tty` | Kill all processes on a terminal |
| `killall name` | Kill all processes with exact name |
| `killall -i name` | Kill interactively (confirm each) |

---

## 🔄 SIGTERM vs SIGKILL — When to Use Each

| Scenario | Use |
|----------|-----|
| Normal process shutdown | `kill PID` (SIGTERM) |
| Process ignores SIGTERM | `kill -9 PID` (SIGKILL) |
| Reload config without restart | `kill -HUP PID` (SIGHUP) |
| Temporarily pause a process | `kill -STOP PID` |
| Resume a paused process | `kill -CONT PID` |
| Check if process is alive | `kill -0 PID` |

---

## 🚀 Full Workflow — Stop a Runaway Process

```bash
# Step 1: Find the process
ps aux | grep myapp
pgrep -la myapp

# Step 2: Check its resource usage
top -p $(pgrep myapp)

# Step 3: Try graceful termination
kill -15 $(pgrep myapp)

# Step 4: Wait and check
sleep 5
pgrep myapp && echo "Still running" || echo "Stopped"

# Step 5: Force kill if needed
kill -9 $(pgrep myapp)

# Step 6: Verify it is gone
pgrep myapp || echo "Process successfully terminated"

# Step 7: Check for leftover files
ls /var/run/myapp.pid /tmp/myapp* 2>/dev/null
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Using `kill -9` first | Always try SIGTERM first — SIGKILL leaves no chance for cleanup |
| Wrong PID — killing the wrong process | Always verify PID with `ps -p PID` before killing |
| `kill` without `sudo` on root processes | Use `sudo kill PID` for processes owned by root |
| `pkill nginx` kills all nginx including master | Use `kill` with the master PID to control gracefully |
| Process respawns after kill | Check for a process manager (systemd, supervisord) restarting it |
| PID reused after delay | Check PID quickly — Linux recycles PIDs after processes exit |

---

## 🔑 Key Takeaways

- `kill PID` sends **SIGTERM** by default — always the first choice for graceful shutdown.
- `kill -9 PID` sends **SIGKILL** — the process is terminated immediately with no cleanup possible.
- **Never start with `kill -9`** — always try SIGTERM first and give the process time to clean up.
- `pkill name` kills by process name — no need to find the PID manually.
- `kill -HUP PID` is used to **reload configuration** in daemons like nginx, sshd, and rsyslog.
- `kill -0 PID` checks if a process exists without sending any signal — useful in scripts.
- Use `kill -STOP` to **pause** a process and `kill -CONT` to **resume** it.

---

## 📚 Related Concepts

- [GNU kill Manual](https://linux.die.net/man/1/kill)
- [Linux Signal Reference](https://man7.org/linux/man-pages/man7/signal.7.html)

</details>
