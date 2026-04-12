# Challenge 33: Monitor Logs in Real-Time in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Real-time log monitoring** is one of the most important skills for any Linux administrator or DevOps engineer. When debugging a service failure, testing a deployment, or investigating a security incident — you need to see log entries appear the moment they are written, not after the fact.

The most common real-time log monitoring tools:
- **`tail -f`** — follow a flat log file live
- **`tail -F`** — follow by filename (handles log rotation)
- **`journalctl -f`** — follow the systemd journal (structured, filterable)
- **Combining with `grep`** — filter live output to only show relevant lines

---

## 🧩 Task

Monitor a log file in real-time and see new entries as they are written.

---

<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Follow a log file in real-time
tail -f /var/log/syslog

# Follow the systemd journal in real-time
journalctl -f

# Follow and filter for errors
tail -f /var/log/syslog | grep -i "error"
```

### How it works

| Command | Description |
|---------|-------------|
| `tail -f file` | **Follow** — stream new lines as they are appended |
| `tail -F file` | Follow by **filename** — reconnects after log rotation |
| `journalctl -f` | Stream the **systemd journal** in real-time |
| `\| grep pattern` | **Filter** the live stream to matching lines only |

---

## 🔢 How `tail -f` Works

```
Log file: /var/log/syslog
                │
   Existing content displayed first
                │
                ▼
   Waits for new lines...
                │
   New line written to file ──► Displayed immediately on terminal
                │
   New line written to file ──► Displayed immediately on terminal
                │
   (continues until Ctrl+C)
```

---

## 📖 Extended Examples

### Example 1 — Follow a Log File

```bash
# Follow system log
tail -f /var/log/syslog        # Debian/Ubuntu
tail -f /var/log/messages      # RHEL/CentOS

# Follow authentication log
tail -f /var/log/auth.log      # Debian/Ubuntu
tail -f /var/log/secure        # RHEL/CentOS

# Follow nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

```bash
# Live output as new entries appear:
Mar 21 10:35:01 hostname sshd[4821]: Accepted publickey for devuser
Mar 21 10:35:02 hostname systemd[1]: Started Session 12 of user devuser.
Mar 21 10:35:10 hostname myapp[5201]: INFO: Processing request /api/users
Mar 21 10:35:10 hostname myapp[5201]: ERROR: Database connection timeout  ← appears live
```

Press `Ctrl+C` to stop following.

---

### Example 2 — Follow Multiple Files at Once

```bash
# Watch both access and error logs simultaneously
tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

```bash
# Output shows which file each line comes from:
==> /var/log/nginx/access.log <==
192.168.1.5 - - [21/Mar/2025] "GET /api HTTP/1.1" 200 1234

==> /var/log/nginx/error.log <==
2025/03/21 10:35:12 [error] 5201#: *1 connect() failed (111: Connection refused)
```

---

### Example 3 — Follow with Log Rotation (`tail -F`)

```bash
# -F (capital F) follows by filename — reconnects if file is rotated
tail -F /var/log/nginx/access.log
```

> 💡 Use `tail -F` instead of `tail -f` for production log monitoring. When `logrotate` renames the file and creates a new one, `-F` automatically follows the new file. `-f` would keep reading the old rotated file.

---

### Example 4 — Show Last N Lines Then Follow

```bash
# Show last 50 lines, then continue following
tail -n 50 -f /var/log/syslog

# Show last 100 lines of nginx error log then follow
tail -n 100 -f /var/log/nginx/error.log
```

---

### Example 5 — Filter Live Output with `grep`

```bash
# Show only error lines in real-time
tail -f /var/log/syslog | grep --line-buffered -i "error"

# Show errors and warnings
tail -f /var/log/syslog | grep --line-buffered -iE "error|warn|critical"

# Show lines from a specific service
tail -f /var/log/syslog | grep --line-buffered "nginx"

# Show lines NOT matching a pattern
tail -f /var/log/syslog | grep --line-buffered -v "DEBUG"
```

> 💡 **`--line-buffered`** is essential when piping `tail -f` to `grep` — without it, `grep` buffers output and delays display. Always include it.

---

### Example 6 — Colorize Live Output

```bash
# Highlight matching text in color
tail -f /var/log/syslog | grep --line-buffered --color -iE "error|warn|critical|fail"

# Advanced colorization with awk
tail -f /var/log/syslog | awk '{
  if (/ERROR|CRITICAL/) print "\033[31m" $0 "\033[0m"    # Red
  else if (/WARN/)      print "\033[33m" $0 "\033[0m"    # Yellow
  else if (/INFO/)      print "\033[32m" $0 "\033[0m"    # Green
  else                  print $0
}'
```

---

### Example 7 — Follow the systemd Journal in Real-Time

```bash
# Follow all journal entries
journalctl -f

# Follow a specific service
journalctl -f -u nginx
journalctl -f -u sshd
journalctl -f -u myapp.service

# Follow and show only errors
journalctl -f -p err

# Follow multiple services
journalctl -f -u nginx -u mysql
```

```bash
# journalctl -f -u nginx output:
Mar 21 10:35:01 hostname nginx[5201]: 192.168.1.5 "GET /api" 200
Mar 21 10:35:12 hostname nginx[5201]: [error] connect() failed: Connection refused
```

---

### Example 8 — Follow Kernel Messages

```bash
# Follow kernel ring buffer in real-time
dmesg -w

# Follow kernel messages with timestamps
dmesg -wT

# Filter kernel messages for errors
dmesg -w | grep -i "error\|fail\|warn"
```

> 💡 `dmesg -w` is the equivalent of `tail -f` for kernel messages — invaluable when debugging hardware issues, driver errors, or OOM events.

---

### Example 9 — Monitor Multiple Services at Once with `multitail`

```bash
# Install multitail
sudo apt install multitail    # Debian/Ubuntu
sudo dnf install multitail    # RHEL

# Watch two files in split panes
multitail /var/log/nginx/access.log /var/log/nginx/error.log

# Watch with color and merge
multitail -i /var/log/syslog -i /var/log/auth.log --mergeall
```

> `multitail` splits the terminal into panes — each showing a different log file simultaneously.

---

### Example 10 — Live Log Monitoring Script

```bash
#!/bin/bash

# Monitor multiple logs and alert on errors
LOGS=(
  "/var/log/syslog"
  "/var/log/nginx/error.log"
  "/var/log/auth.log"
)
PATTERN="error|fail|critical|denied|refused"
ALERT_LOG="/var/log/realtime_alerts.log"

echo "Starting live monitoring... (Ctrl+C to stop)"

tail -F "${LOGS[@]}" 2>/dev/null | \
  grep --line-buffered -iE "$PATTERN" | \
  while read -r line; do
    TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    echo "[$TIMESTAMP] $line" | tee -a "$ALERT_LOG"
  done
```

---

### Example 11 — `watch` for Periodic Log Snippets

```bash
# Refresh last 10 lines of error log every 2 seconds
watch -n 2 'tail -10 /var/log/nginx/error.log'

# Watch error count
watch -n 5 'grep -c "ERROR" /var/log/myapp.log'

# Watch for new errors since a specific time
watch -n 10 'journalctl -u nginx --since "5 min ago" | grep -i error'
```

---

### Example 12 — Follow Logs While Testing a Change

```bash
# Terminal 1: Start following the log
tail -F /var/log/nginx/error.log

# Terminal 2: Make your change and watch the effect
sudo systemctl reload nginx
curl http://localhost/api/test
```

> 💡 The classic debugging pattern — one terminal follows the log, another triggers actions.

---

## 🗂️ Real-Time Log Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `tail -f file` | Follow a log file |
| `tail -F file` | Follow by name (survives rotation) |
| `tail -n 50 -f file` | Show last 50 lines then follow |
| `tail -f file \| grep -i "error"` | Follow + filter |
| `tail -f file \| grep --line-buffered pattern` | Buffering-safe filter |
| `journalctl -f` | Follow all journal entries |
| `journalctl -f -u service` | Follow one service |
| `journalctl -f -p err` | Follow errors only |
| `dmesg -w` | Follow kernel messages |
| `multitail file1 file2` | Follow multiple files in panes |

---

## 🔄 `tail -f` vs `journalctl -f` — When to Use Each

| Feature | `tail -f` | `journalctl -f` |
|---------|-----------|-----------------|
| Works with flat files | ✅ | ❌ |
| Works with systemd services | Limited | ✅ |
| Filter by priority | ❌ | ✅ (`-p err`) |
| Filter by service | ❌ | ✅ (`-u service`) |
| Handles log rotation | `-F` flag | ✅ Automatic |
| Available on all distros | ✅ | Systemd only |

---

## 🚀 Full Workflow — Debug a Service While It Runs

```bash
# Step 1: Open two terminals

# Terminal 1: Start monitoring the service log
journalctl -f -u myapp.service
# or
tail -F /var/log/myapp/app.log

# Terminal 2: Trigger the action you want to test
sudo systemctl restart myapp
curl -X POST http://localhost/api/test

# Watch Terminal 1 for errors as they appear live

# Step 2: Add filtering if too noisy
journalctl -f -u myapp.service | grep --line-buffered -iE "error|warn|fail"

# Step 3: Save interesting output
journalctl -f -u myapp.service | tee /tmp/debug_session.log

# Step 4: Stop following when done
# Ctrl+C
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `tail -f` stops working after log rotation | Use `tail -F` instead |
| `grep` delays output when piping from `tail -f` | Always add `--line-buffered` to `grep` |
| `journalctl -f` shows too many services | Add `-u service_name` to filter |
| `dmesg -w` not working on older systems | Try `watch -n 1 dmesg \| tail -20` as an alternative |
| Too many lines to read | Add `grep --line-buffered` filter or `-p err` for errors only |

---

## 🔑 Key Takeaways

- `tail -f` streams new lines as they are written — the most common real-time log tool.
- Use **`tail -F`** (capital F) in production — it automatically follows the new file after log rotation.
- **`--line-buffered`** is essential when piping `tail -f` to `grep` — prevents delayed output.
- `journalctl -f -u service` is the best tool for systemd services — built-in filtering by unit and priority.
- `dmesg -w` follows kernel ring buffer messages — use it for hardware and driver debugging.
- The **two-terminal pattern** — one follows the log, the other triggers actions — is the standard debug workflow.
- Use **`tee`** to simultaneously display and save output: `tail -f log | tee captured.log`.

---

## 📚 Related Concepts

- [GNU tail Manual](https://www.gnu.org/software/coreutils/manual/html_node/tail-invocation.html)
- [journalctl Manual](https://www.freedesktop.org/software/systemd/man/journalctl.html)

</details>
