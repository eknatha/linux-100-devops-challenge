# Challenge 63: Use journalctl Effectively

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**`journalctl`** is the primary tool for querying and displaying logs collected by **systemd's journal daemon** (`systemd-journald`). It collects logs from the kernel, systemd units, applications, and more — all in one centralized, indexed, structured store.

While `journalctl -b` focuses on boot sessions (Challenge 62), mastering `journalctl` broadly means knowing how to **slice, filter, follow, and export** logs efficiently for any situation — from quick debugging to production incident analysis.

---

## 🧩 Task

View the most recent 50 log entries from the system journal.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
journalctl -n 50
```

### How it works

| Part | Description |
|------|-------------|
| `journalctl` | Query and display the systemd journal |
| `-n 50` | Show the **last 50 lines** of the journal (most recent entries) |

> `-n` works like `tail -n` — it shows the most recent N entries from the end of the journal. Default is 10 if no number is given.

---

## 📖 Extended Examples

### Example 1 — View Last 50 Log Entries

```bash
journalctl -n 50
```

```bash
Mar 21 10:00:01 hostname sshd[1024]: Accepted publickey for devuser
Mar 21 10:00:05 hostname sudo[1031]: devuser: TTY=pts/0 ; USER=root ; COMMAND=/bin/bash
Mar 21 10:00:12 hostname nginx[1102]: 127.0.0.1 - GET / HTTP/1.1 200
...
(50 lines total)
```

---

### Example 2 — View Last N Lines (Varying Amounts)

```bash
# Last 10 lines (default)
journalctl -n

# Last 20 lines
journalctl -n 20

# Last 100 lines
journalctl -n 100

# Last 500 lines
journalctl -n 500
```

---

### Example 3 — Follow Live Logs in Real Time

```bash
journalctl -f
```

```bash
Mar 21 10:05:01 hostname sshd[2048]: Failed password for root from 192.168.1.10
Mar 21 10:05:03 hostname sshd[2049]: Failed password for root from 192.168.1.10
Mar 21 10:05:05 hostname kernel: iptables: DROP IN=eth0 SRC=192.168.1.10
```

> 💡 `-f` streams new entries in real time — equivalent to `tail -f` for the system journal. Press `Ctrl+C` to stop.

---

### Example 4 — Combine `-n` and `-f` (Follow from Last N Lines)

```bash
journalctl -n 50 -f
```

> Shows the last 50 lines first, then continues streaming new entries as they arrive — the most useful combination for real-time debugging.

---

### Example 5 — Filter by a Specific Service

```bash
# View last 50 lines for SSH
journalctl -u sshd -n 50

# View last 50 lines for nginx
journalctl -u nginx -n 50

# Follow logs for a service in real time
journalctl -u nginx -f

# Multiple services
journalctl -u nginx -u mysql -n 50
```

```bash
Mar 21 10:00:01 hostname sshd[801]: Server listening on 0.0.0.0 port 22
Mar 21 10:01:05 hostname sshd[802]: Accepted publickey for devuser
Mar 21 10:02:10 hostname sshd[803]: Disconnected from devuser port 52413
```

---

### Example 6 — Filter by Priority Level

```bash
# Show only errors (and above) from the last 50 entries
journalctl -n 50 -p err

# Show only warnings and above
journalctl -p warning

# Show a range of priorities (err through info)
journalctl -p 3..6

# Show critical issues only
journalctl -p crit
```

| Priority Name | Number | Severity |
|--------------|--------|----------|
| `emerg` | 0 | System is unusable |
| `alert` | 1 | Immediate action required |
| `crit` | 2 | Critical conditions |
| `err` | 3 | Error conditions |
| `warning` | 4 | Warning conditions |
| `notice` | 5 | Normal significant event |
| `info` | 6 | Informational |
| `debug` | 7 | Debug messages |

---

### Example 7 — Filter by Time Range

```bash
# Logs since a specific date and time
journalctl --since "2025-03-21 09:00:00"

# Logs between two timestamps
journalctl --since "2025-03-21 09:00:00" --until "2025-03-21 10:00:00"

# Logs from the last 30 minutes
journalctl --since "30 min ago"

# Logs from the last 2 hours
journalctl --since "2 hours ago"

# Logs from today
journalctl --since today

# Logs from yesterday
journalctl --since yesterday --until today
```

---

### Example 8 — Filter by Process ID or User

```bash
# Logs from a specific PID
journalctl _PID=1234

# Logs from a specific user (by UID)
journalctl _UID=1001

# Logs from a specific executable
journalctl _EXE=/usr/sbin/sshd

# Logs from the current user
journalctl _UID=$(id -u)
```

---

### Example 9 — Search with a Keyword (grep-style)

```bash
# Built-in grep with -g
journalctl -g "failed"
journalctl -g "error" -n 50

# Classic pipe with grep
journalctl -n 100 | grep -i "denied"

# Case-insensitive search with context
journalctl | grep -i -A 2 "out of memory"
```

---

### Example 10 — Output in Different Formats

```bash
# Default short format
journalctl -n 10 -o short

# Only the message text (clean output)
journalctl -n 10 -o cat

# JSON format (for scripting or log aggregators)
journalctl -n 10 -o json

# JSON pretty-printed
journalctl -n 10 -o json-pretty

# Verbose — all metadata fields
journalctl -n 5 -o verbose

# Short ISO timestamps
journalctl -n 10 -o short-iso
```

```bash
# cat output — clean messages only
Started OpenSSH server daemon.
Accepted publickey for devuser from 192.168.1.5 port 54321 ssh2
session opened for user devuser by (uid=0)
```

---

### Example 11 — View Logs Without Pager (for Piping or Scripts)

```bash
# Disable interactive pager
journalctl -n 50 --no-pager

# Pipe into grep
journalctl -n 200 --no-pager | grep "error"

# Save to file
journalctl -n 500 --no-pager > /tmp/recent_logs.txt

# Save errors to file with timestamp in filename
journalctl -p err --no-pager > /tmp/errors_$(date +"%Y-%m-%d_%H-%M").txt
```

---

### Example 12 — Check Journal Disk Usage and Manage Size

```bash
# Check how much disk the journal is using
journalctl --disk-usage
```

```bash
Archived and active journals take up 1.2G in the file system.
```

```bash
# Trim journal to a maximum size
sudo journalctl --vacuum-size=500M

# Delete journal entries older than 7 days
sudo journalctl --vacuum-time=7d

# Delete entries older than 2 weeks
sudo journalctl --vacuum-time=2weeks
```

---

### Example 13 — Configure Persistent Journal Logging

By default on some distros, the journal is stored in memory and lost on reboot. Enable persistence:

```bash
# Create the persistent journal directory
sudo mkdir -p /var/log/journal

# Restart the journal daemon
sudo systemctl restart systemd-journald

# Verify persistence is active
journalctl --disk-usage
```

Alternatively, set it in the config:

```bash
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
Storage=persistent        # auto | volatile | persistent | none
SystemMaxUse=500M         # Max disk space for system journal
RuntimeMaxUse=100M        # Max RAM for in-memory journal
MaxRetentionSec=30day     # Keep logs for 30 days
```

```bash
sudo systemctl restart systemd-journald
```

---

## 🗂️ `journalctl` — Complete Flag Reference

| Flag | Description |
|------|-------------|
| `-n N` | Show last N lines (default: 10) |
| `-f` | Follow — stream new entries live |
| `-u unit` | Filter by systemd unit/service |
| `-p level` | Filter by priority (0–7 or name) |
| `-b [N]` | Show logs from boot N (0 = current) |
| `--since` | Show logs from a time/date |
| `--until` | Show logs up to a time/date |
| `-g pattern` | Filter by grep pattern |
| `-k` | Kernel messages only (`dmesg` equivalent) |
| `-o format` | Output format: short, json, cat, verbose |
| `--no-pager` | Disable pager (for pipes and scripts) |
| `--disk-usage` | Show journal disk consumption |
| `--vacuum-size=` | Shrink journal to specified size |
| `--vacuum-time=` | Remove journal entries older than time |
| `--list-boots` | List all recorded boot sessions |
| `_PID=` | Filter by process ID |
| `_UID=` | Filter by user ID |
| `_EXE=` | Filter by executable path |

---

## 🔄 `journalctl` vs `tail` — Quick Comparison

| Feature | `journalctl` | `tail -f /var/log/syslog` |
|---------|-------------|--------------------------|
| Structured data | ✅ | ❌ |
| Filter by service | ✅ `-u` | ❌ Manual grep |
| Filter by priority | ✅ `-p` | ❌ Manual grep |
| Boot-scoped logs | ✅ `-b` | ❌ |
| JSON output | ✅ `-o json` | ❌ |
| Real-time follow | ✅ `-f` | ✅ |
| Works on all distros | ❌ systemd only | ✅ |

---

## 🚀 Full Workflow — Investigate a Service Issue

```bash
# Step 1: Check recent logs for any anomalies
journalctl -n 100 --no-pager

# Step 2: Filter to only errors
journalctl -n 100 -p err --no-pager

# Step 3: Focus on the suspected service
journalctl -u nginx -n 50

# Step 4: Follow in real time while reproducing the issue
journalctl -u nginx -f

# Step 5: Check logs around the time the issue was reported
journalctl -u nginx --since "2025-03-21 09:00:00" --until "2025-03-21 09:30:00"

# Step 6: Export relevant logs for sharing or further analysis
journalctl -u nginx --since "2025-03-21 09:00:00" --no-pager > /tmp/nginx_issue.txt
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Output fills terminal | Use `-n 50` to limit or `-p err` to filter |
| Piping fails due to pager | Always add `--no-pager` when piping or redirecting |
| Logs missing after reboot | Enable persistent storage: `mkdir -p /var/log/journal` |
| Journal consuming too much disk | Use `journalctl --vacuum-size=500M` or set `SystemMaxUse` in config |
| `-g` flag not available | Older `journalctl` versions lack `-g` — pipe to `grep` instead |
| Timestamps showing UTC instead of local | Set `localtime` in `/etc/systemd/journald.conf` or use `--output-fields=` |

---

## 🔑 Key Takeaways

- `journalctl -n 50` is the quickest way to see the **50 most recent log entries**.
- Combine with `-f` to **follow live** from those last 50 entries onward.
- Use `-u servicename` to scope logs to a **single service** — dramatically reduces noise.
- Use `-p err` to instantly surface **errors and critical messages** across all services.
- Use `--since` / `--until` for **time-scoped investigations** around a known incident window.
- Always add `--no-pager` when piping output to files, `grep`, or scripts.
- Manage journal disk usage proactively with `--vacuum-size` and `--vacuum-time`.

---

## 📚 Related Concepts

- [systemd Journal Man Page](https://www.freedesktop.org/software/systemd/man/journalctl.html)
- [journald.conf Reference](https://www.freedesktop.org/software/systemd/man/journald.conf.html)
- [systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html) — manage services alongside their logs

</details>
