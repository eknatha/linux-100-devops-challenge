# Challenge 62: Analyze Boot Logs in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

## 📌 Overview

**`journalctl`** is the command-line interface for **systemd's journal** — the centralized logging system on modern Linux distributions. Unlike traditional log files scattered across `/var/log/`, the journal stores structured, indexed logs from the kernel, services, and applications in a binary format, making it fast and powerful to query.

The `-b` flag filters logs to the **current boot session**, making it the go-to command for diagnosing startup issues, failed services, and hardware errors.

---

## 🧩 Task

View and analyze system logs from the current boot.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
journalctl -b
```

### How it works

| Part | Description |
|------|-------------|
| `journalctl` | Query and display systemd journal logs |
| `-b` | Show logs from the **current boot** session only |

> Without `-b`, `journalctl` shows all logs across all past boots — which can be overwhelming.

---

## 📖 Extended Examples

### Example 1 — View Current Boot Logs

```bash
journalctl -b
```

```bash
Mar 21 09:00:01 hostname kernel: Linux version 6.5.0-generic
Mar 21 09:00:01 hostname kernel: BIOS-provided physical RAM map
Mar 21 09:00:02 hostname systemd[1]: Started Journal Service
Mar 21 09:00:03 hostname systemd[1]: Reached target Basic System
...
```

---

### Example 2 — View Logs from a Previous Boot

```bash
# List available boots
journalctl --list-boots
```

```bash
-3  abc123def  Thu 2025-03-18 09:00:01 UTC—Thu 2025-03-18 17:45:22 UTC
-2  def456ghi  Fri 2025-03-19 08:55:10 UTC—Fri 2025-03-19 23:12:05 UTC
-1  ghi789jkl  Sat 2025-03-20 09:01:44 UTC—Sat 2025-03-20 22:30:11 UTC
 0  jkl012mno  Sun 2025-03-21 09:00:01 UTC—present
```

```bash
# View logs from the previous boot (-1)
journalctl -b -1

# View logs from two boots ago (-2)
journalctl -b -2
```

---

### Example 3 — Filter by Priority / Severity Level

```bash
# Show only errors and critical messages from current boot
journalctl -b -p err
```

```bash
Mar 21 09:01:15 hostname kernel: ata1: SRST failed (errno=-16)
Mar 21 09:02:30 hostname NetworkManager[512]: Failed to connect to wifi
```

| Priority | Level | Description |
|----------|-------|-------------|
| `0` | `emerg` | System is unusable |
| `1` | `alert` | Action must be taken immediately |
| `2` | `crit` | Critical conditions |
| `3` | `err` | Error conditions |
| `4` | `warning` | Warning conditions |
| `5` | `notice` | Normal but significant events |
| `6` | `info` | Informational messages |
| `7` | `debug` | Debug-level messages |

```bash
# Show warnings and above (0–4)
journalctl -b -p warning

# Show a range of priorities
journalctl -b -p 3..6
```

---

### Example 4 — Filter by a Specific Service (Unit)

```bash
# View boot logs for SSH service only
journalctl -b -u ssh

# View boot logs for nginx
journalctl -b -u nginx

# View multiple services
journalctl -b -u ssh -u nginx
```

```bash
Mar 21 09:01:05 hostname sshd[801]: Server listening on 0.0.0.0 port 22
Mar 21 09:01:05 hostname sshd[801]: Server listening on :: port 22
```

---

### Example 5 — Follow Live Logs (Like `tail -f`)

```bash
# Follow current boot logs in real time
journalctl -b -f

# Follow logs for a specific service
journalctl -b -u nginx -f
```

> 💡 `-f` streams new log entries as they arrive — useful for monitoring a service while testing or debugging.

---

### Example 6 — Show Only the Last N Lines

```bash
# Show last 50 lines from current boot
journalctl -b -n 50

# Show last 100 lines for a specific service
journalctl -b -u sshd -n 100
```

---

### Example 7 — Filter by Time Range

```bash
# Logs since a specific time today
journalctl -b --since "09:00:00"

# Logs between two timestamps
journalctl -b --since "2025-03-21 09:00:00" --until "2025-03-21 10:00:00"

# Logs from the last 30 minutes
journalctl -b --since "30 min ago"

# Logs from the last hour
journalctl -b --since "1 hour ago"
```

---

### Example 8 — Search for a Keyword

```bash
# Grep-style keyword search in boot logs
journalctl -b | grep -i "error"

# Search for failed services
journalctl -b | grep -i "failed"

# Search for a specific process
journalctl -b | grep -i "sshd"
```

```bash
# Or use journalctl's built-in grep
journalctl -b -g "failed"
```

---

### Example 9 — Show Kernel Messages Only

```bash
# Kernel messages from current boot (equivalent to dmesg)
journalctl -b -k
```

```bash
Mar 21 09:00:01 hostname kernel: Initializing cgroup subsys cpuset
Mar 21 09:00:01 hostname kernel: Linux version 6.5.0-generic
Mar 21 09:00:01 hostname kernel: ACPI: RSDP 0x00000000000F05B0 000024
...
```

> 💡 `-k` (or `--dmesg`) shows only kernel ring buffer messages — equivalent to running `dmesg`.

---

### Example 10 — Output in Different Formats

```bash
# Short format (default)
journalctl -b -o short

# JSON format (for parsing or scripting)
journalctl -b -o json

# JSON pretty-printed
journalctl -b -o json-pretty

# Only the message text, no metadata
journalctl -b -o cat

# Full verbose output
journalctl -b -o verbose
```

```bash
# JSON output example
{
  "__REALTIME_TIMESTAMP": "1711012801000000",
  "MESSAGE": "Started OpenSSH server daemon",
  "SYSLOG_IDENTIFIER": "sshd",
  "_HOSTNAME": "myserver",
  "PRIORITY": "6"
}
```

---

### Example 11 — Check for Failed Services at Boot

```bash
# Show all failed units
systemctl --failed

# Show logs for a specific failed service
journalctl -b -u servicename.service -p err
```

```bash
# Quick boot health check one-liner
journalctl -b -p err..crit --no-pager
```

---

### Example 12 — Export Boot Logs to a File

```bash
# Export current boot logs to a text file
journalctl -b --no-pager > boot_logs_$(date +"%Y-%m-%d").txt

# Export errors only
journalctl -b -p err --no-pager > boot_errors_$(date +"%Y-%m-%d").txt

# Export as JSON for analysis
journalctl -b -o json --no-pager > boot_logs_$(date +"%Y-%m-%d").json
```

> 💡 `--no-pager` disables the interactive pager so output flows directly to stdout (and into files or pipes).

---

## 🗂️ `journalctl` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-b [N]` | Show logs from boot `N` (0 = current, -1 = previous) |
| `--list-boots` | List all recorded boot sessions |
| `-u unit` | Filter by systemd unit/service |
| `-p level` | Filter by priority level (emerg–debug or 0–7) |
| `-f` | Follow — stream new entries in real time |
| `-n N` | Show last N lines |
| `-k` | Show kernel messages only |
| `-g pattern` | Filter by grep pattern |
| `--since` | Show logs since a timestamp |
| `--until` | Show logs until a timestamp |
| `-o format` | Output format (short, json, cat, verbose) |
| `--no-pager` | Disable pager for piping or file output |
| `--disk-usage` | Show total journal disk usage |
| `--vacuum-size=` | Reduce journal to a max size |
| `--vacuum-time=` | Delete journal entries older than a time |

---

## 📋 `journalctl` vs Traditional Log Files

| Feature | `journalctl` | `/var/log/*.log` |
|---------|-------------|-----------------|
| Format | Binary (structured) | Plain text |
| Indexing | Fast indexed queries | Sequential `grep` |
| Boot filtering | Built-in (`-b`) | Manual `grep` |
| Priority filtering | Built-in (`-p`) | Manual `grep` |
| Log rotation | Automatic | Via `logrotate` |
| Remote logging | Via `systemd-journal-remote` | Via `rsyslog`/`syslog-ng` |

---

## 🚀 Full Workflow — Diagnose a Boot Issue

```bash
# Step 1: List all boots to identify when the issue occurred
journalctl --list-boots

# Step 2: Check for critical errors in the current boot
journalctl -b -p err --no-pager

# Step 3: Check for failed services
systemctl --failed

# Step 4: Investigate a specific failed service
journalctl -b -u <failed-service> -n 100

# Step 5: Review kernel messages for hardware errors
journalctl -b -k | grep -i "error\|fail\|warn"

# Step 6: Export logs for deeper analysis or sharing
journalctl -b --no-pager > /tmp/boot_report_$(date +"%Y-%m-%d").txt
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Output too long to read | Use `-n 50` for last 50 lines or `-p err` to filter errors only |
| Logs not persisting across reboots | Enable persistent logging: `mkdir -p /var/log/journal && systemctl restart systemd-journald` |
| Journal consuming too much disk | Run `journalctl --vacuum-size=500M` to trim to 500MB |
| `journalctl` not available | System may use `syslog` — check `/var/log/syslog` or `/var/log/messages` instead |
| Piping output but pager interrupts | Always add `--no-pager` when piping or redirecting |
| Time range filtering not working | Ensure system clock is correct — verify with `timedatectl` |

---

## 🔑 Key Takeaways

- `journalctl -b` shows **current boot logs only** — far more focused than viewing all logs.
- Use `-p err` to quickly surface **errors and critical messages** without scrolling through everything.
- Use `-u servicename` to isolate logs for a **specific service** during boot.
- `-f` enables **real-time following** — the systemd equivalent of `tail -f`.
- `--list-boots` reveals all recorded sessions so you can **compare across boots**.
- Add `--no-pager` whenever piping output to a file, `grep`, or another command.
- Combine `journalctl -b -p err` with `systemctl --failed` for a fast boot health check.

---

## 📚 Related Concepts

- [systemd Journal Documentation](https://www.freedesktop.org/software/systemd/man/journalctl.html)
- [dmesg](https://linux.die.net/man/1/dmesg) — kernel ring buffer (pre-systemd alternative for kernel messages)
- [systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html) — manage and inspect systemd services

</details>
