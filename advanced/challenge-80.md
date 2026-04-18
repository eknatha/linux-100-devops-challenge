# Challenge 80: Log Analysis for Root Cause Investigation

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Log analysis** is the cornerstone of root cause investigation in Linux. When something goes wrong — a service fails, performance degrades, or security is breached — the logs tell the story. The challenge is knowing which logs to look at, how to filter the noise, and how to correlate events across multiple log sources to find the **exact moment and cause** of the problem.

`grep` is the primary tool for searching log files, but effective log analysis combines `grep`, `awk`, `sed`, `sort`, `uniq`, and `journalctl` to extract meaningful patterns from thousands of log lines.

---

## 🧩 Task

Search system logs for error messages to identify the root cause of a system or application issue.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
grep -i error /var/log/messages
```

### How it works

| Part | Description |
|------|-------------|
| `grep` | Search for a pattern in files |
| `-i` | **Case-insensitive** — matches `error`, `ERROR`, `Error`, etc. |
| `error` | The search pattern to look for |
| `/var/log/messages` | The system log file on **RHEL/CentOS/Fedora** |

> 💡 On **Debian/Ubuntu**, the equivalent file is `/var/log/syslog`:
> ```bash
> grep -i error /var/log/syslog
> ```

---

## 🗂️ Log File Locations by Distribution

| Log File | Distribution | Contents |
|----------|-------------|----------|
| `/var/log/messages` | RHEL / CentOS / Fedora | General system messages |
| `/var/log/syslog` | Debian / Ubuntu | General system messages |
| `/var/log/auth.log` | Debian / Ubuntu | Authentication events |
| `/var/log/secure` | RHEL / CentOS | Authentication events |
| `/var/log/kern.log` | Debian / Ubuntu | Kernel messages |
| `/var/log/dmesg` | All | Boot-time kernel messages |
| `/var/log/boot.log` | All | Boot process log |
| `/var/log/cron` | All | Cron job execution log |
| `/var/log/maillog` | RHEL | Mail server log |
| `/var/log/nginx/error.log` | All | Nginx error log |
| `/var/log/apache2/error.log` | Debian | Apache error log |
| `/var/log/mysql/error.log` | All | MySQL error log |
| `journalctl` | Systemd distros | All-in-one structured journal |

---

## 📖 Extended Examples

### Example 1 — Basic Error Search

```bash
# RHEL/CentOS
grep -i error /var/log/messages

# Debian/Ubuntu
grep -i error /var/log/syslog
```

```bash
Mar 21 10:35:01 hostname kernel: EXT4-fs error (device sda1): ext4_find_entry:1455: inode #2: comm myapp: reading directory lblock 0
Mar 21 10:35:02 hostname myapp[4821]: ERROR: Failed to open config file /etc/myapp/config.yaml
Mar 21 10:35:02 hostname myapp[4821]: ERROR: Cannot connect to database at 127.0.0.1:5432
```

---

### Example 2 — Search Multiple Keywords

```bash
# Search for error OR failure OR critical
grep -iE "error|fail|critical|fatal" /var/log/messages

# Search across multiple log files
grep -i error /var/log/messages /var/log/secure /var/log/cron

# Search all log files in /var/log
grep -i error /var/log/*.log 2>/dev/null
```

---

### Example 3 — Show Context Around Matches

```bash
# Show 3 lines before and after each match (-B before, -A after)
grep -i error -B 3 -A 3 /var/log/messages

# Show 5 lines around the match
grep -i error -C 5 /var/log/messages
```

```bash
Mar 21 10:34:58 hostname myapp[4821]: Loading configuration...
Mar 21 10:34:59 hostname myapp[4821]: Connecting to database...
Mar 21 10:35:00 hostname myapp[4821]: Retrying connection (attempt 3/3)...
Mar 21 10:35:01 hostname myapp[4821]: ERROR: Cannot connect to database  ← match
Mar 21 10:35:01 hostname myapp[4821]: Shutting down...
Mar 21 10:35:02 hostname systemd[1]: myapp.service: Failed with result 'exit-code'
Mar 21 10:35:02 hostname systemd[1]: Failed to start My Application Service.
```

> 💡 Context lines are essential — the error itself often gives little information, but the **lines before and after** reveal the sequence of events.

---

### Example 4 — Filter Errors by Time Window

```bash
# Errors from a specific date
grep "Mar 21" /var/log/messages | grep -i error

# Errors from a specific hour
grep "Mar 21 10:" /var/log/messages | grep -i error

# Errors between two times
awk '/Mar 21 10:30/,/Mar 21 10:45/' /var/log/messages | grep -i error

# Using journalctl for precise time filtering
journalctl --since "2025-03-21 10:30:00" --until "2025-03-21 10:45:00" -p err
```

---

### Example 5 — Count Error Frequency by Type

```bash
# Count occurrences of each unique error message
grep -i error /var/log/messages | awk '{print $6}' | sort | uniq -c | sort -rn | head -10
```

```bash
  45  myapp[4821]:
  23  kernel:
  12  sshd[1024]:
   8  nginx[5201]:
```

```bash
# Count errors per hour to find when the problem started
grep -i error /var/log/messages | awk '{print $1" "$2" "$3}' | cut -d: -f1 | sort | uniq -c
```

```bash
   2  Mar 21 08
   3  Mar 21 09
  48  Mar 21 10    ← spike at 10:00
   5  Mar 21 11
```

---

### Example 6 — Search in Compressed and Rotated Logs

Logs are often rotated and compressed. Use `zgrep` to search without decompressing:

```bash
# Search in current and all rotated log files
grep -i error /var/log/messages*

# Search in compressed rotated logs
zgrep -i error /var/log/messages.*.gz

# Search across all — current, rotated, and compressed
grep -i error /var/log/messages /var/log/messages.1 2>/dev/null
zgrep -i error /var/log/messages.*.gz 2>/dev/null
```

---

### Example 7 — Correlate Errors Across Multiple Log Files

```bash
#!/bin/bash
TIMESTAMP="Mar 21 10:3"

echo "=== System Messages ==="
grep "$TIMESTAMP" /var/log/messages | grep -i "error\|fail"

echo "=== Auth Events ==="
grep "$TIMESTAMP" /var/log/secure | grep -i "fail\|invalid"

echo "=== Nginx Errors ==="
grep "$TIMESTAMP" /var/log/nginx/error.log 2>/dev/null

echo "=== MySQL Errors ==="
grep "$TIMESTAMP" /var/log/mysql/error.log 2>/dev/null
```

> 💡 Correlating timestamps across multiple log files is the key to **root cause analysis** — the first error in any log file often triggers cascading failures in others.

---

### Example 8 — Extract Unique IP Addresses from Logs

```bash
# Find all IPs generating errors in nginx
grep "error" /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

```bash
 127  203.0.113.10
  45  198.51.100.5
  23  192.168.1.99
```

```bash
# Find failed SSH login IPs
grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10
```

---

### Example 9 — Monitor Logs in Real Time

```bash
# Follow a log file live
tail -f /var/log/messages

# Follow and filter for errors
tail -f /var/log/messages | grep --line-buffered -i "error\|fail\|critical"

# Follow multiple files at once
tail -f /var/log/messages /var/log/nginx/error.log /var/log/myapp/error.log

# Use journalctl to follow all logs with priority filtering
journalctl -f -p err
```

---

### Example 10 — Use `awk` for Advanced Log Parsing

```bash
# Extract timestamp and error message from structured log
awk '/ERROR/ {print $1, $2, $3, substr($0, index($0,$6))}' /var/log/messages | head -20

# Count HTTP 5xx errors in nginx access log
awk '$9 ~ /^5/ {count++} END {print "5xx errors:", count}' /var/log/nginx/access.log

# Find the slowest requests (> 1 second response time)
awk '$NF > 1.0 {print $0}' /var/log/nginx/access.log | sort -k$NF -rn | head -10

# Find average response time
awk '{sum += $NF; count++} END {printf "Avg: %.3fs over %d requests\n", sum/count, count}' /var/log/nginx/access.log
```

---

### Example 11 — Use `sed` for Log Cleaning and Extraction

```bash
# Remove timestamp from each line for grouping similar errors
sed 's/^[A-Z][a-z]* [0-9]* [0-9]*:[0-9]*:[0-9]* [^ ]* //' /var/log/messages | grep -i error | sort | uniq -c | sort -rn | head -10

# Extract just the error messages (remove hostname and PID)
grep -i error /var/log/messages | sed 's/.*\]: //'

# Anonymize IPs in logs before sharing
sed -E 's/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/x.x.x.x/g' /var/log/nginx/access.log
```

---

### Example 12 — Full Root Cause Analysis Script

```bash
#!/bin/bash

# ── Configuration ────────────────────────────────────────
LOG_FILE="${1:-/var/log/messages}"    # Default to messages, allow override
SINCE="${2:-$(date '+%b %e' )}"       # Default to today
OUTPUT="/tmp/rca_$(date +%Y%m%d_%H%M%S).txt"

echo "============================================" | tee "$OUTPUT"
echo " Root Cause Analysis Report"                 | tee -a "$OUTPUT"
echo " Log: $LOG_FILE | Since: $SINCE"             | tee -a "$OUTPUT"
echo " Generated: $(date)"                         | tee -a "$OUTPUT"
echo "============================================" | tee -a "$OUTPUT"

# ── 1. Error count over time ──────────────────────────────
echo -e "\n[1] Error Frequency by Hour]"           | tee -a "$OUTPUT"
grep -i "error\|fail\|critical" "$LOG_FILE" 2>/dev/null \
  | grep "$SINCE" \
  | awk '{print $3}' | cut -d: -f1 \
  | sort | uniq -c                                  | tee -a "$OUTPUT"

# ── 2. Top error sources ──────────────────────────────────
echo -e "\n[2] Top Error Sources (Process/Service)]" | tee -a "$OUTPUT"
grep -i "error" "$LOG_FILE" 2>/dev/null \
  | grep "$SINCE" \
  | awk '{print $5}' | sort | uniq -c | sort -rn | head -10 | tee -a "$OUTPUT"

# ── 3. Unique error messages ──────────────────────────────
echo -e "\n[3] Most Common Error Messages]"         | tee -a "$OUTPUT"
grep -i "error" "$LOG_FILE" 2>/dev/null \
  | grep "$SINCE" \
  | sed 's/^[A-Z][a-z]* [0-9 ]* [0-9:]*  *[^ ]* //' \
  | sort | uniq -c | sort -rn | head -10            | tee -a "$OUTPUT"

# ── 4. Critical and fatal messages ───────────────────────
echo -e "\n[4] Critical / Fatal Messages]"          | tee -a "$OUTPUT"
grep -iE "critical|fatal|panic|oom|segfault" "$LOG_FILE" 2>/dev/null \
  | grep "$SINCE" | tail -20                        | tee -a "$OUTPUT"

# ── 5. Journal errors (systemd) ──────────────────────────
echo -e "\n[5] Systemd Journal Errors (today)]"     | tee -a "$OUTPUT"
journalctl --since today -p err --no-pager 2>/dev/null | tail -20 | tee -a "$OUTPUT"

# ── 6. Failed services ────────────────────────────────────
echo -e "\n[6] Currently Failed Services]"          | tee -a "$OUTPUT"
systemctl --failed --no-legend 2>/dev/null          | tee -a "$OUTPUT"

echo -e "\nReport saved: $OUTPUT"
```

---

## 🔍 grep Options — Quick Reference

| Flag | Description |
|------|-------------|
| `-i` | Case-insensitive matching |
| `-E` | Extended regex — enables `|` (OR), `+`, `?` |
| `-v` | Invert match — show lines NOT matching |
| `-c` | Count matching lines |
| `-n` | Show line numbers |
| `-l` | Show only filenames with matches |
| `-r` | Recursive — search directory tree |
| `-A N` | Show N lines **after** match |
| `-B N` | Show N lines **before** match |
| `-C N` | Show N lines **around** match (before + after) |
| `--color` | Highlight matching text |
| `--no-filename` | Suppress filename in output |
| `-m N` | Stop after N matches |
| `-o` | Print only the matching part |
| `--line-buffered` | Flush output line by line (needed for `tail -f` piping) |

---

## 🔄 Combining Tools for Log Analysis

```bash
# Pattern: tail → grep → color highlight
tail -f /var/log/messages | grep --color -iE "error|warn|critical"

# Pattern: grep → awk → sort → uniq
grep -i error /var/log/messages | awk '{print $5}' | sort | uniq -c | sort -rn

# Pattern: journalctl → grep → count
journalctl --since today | grep -i "fail" | wc -l

# Pattern: multi-file grep with filename
grep -l -i "error" /var/log/*.log

# Pattern: time-filtered awk → grep
awk '/10:30/,/10:45/' /var/log/messages | grep -i error
```

---

## 🚀 Full Workflow — Root Cause Analysis for an Incident

```bash
# Step 1: Find the approximate time the issue started
# (Check monitoring, user reports, alert timestamps)

# Step 2: Filter logs to that time window
grep "Mar 21 10:3" /var/log/messages | grep -iE "error|fail|critical"

# Step 3: Check all relevant log sources
grep "Mar 21 10:3" /var/log/secure /var/log/cron /var/log/nginx/error.log 2>/dev/null | grep -i error

# Step 4: Look at the error in context
grep -i "ERROR: Cannot connect" /var/log/messages -B 5 -A 5

# Step 5: Count error frequency to find the spike
grep -i error /var/log/messages | awk '{print $3}' | cut -d: -f1 | sort | uniq -c

# Step 6: Check if there was an OOM event
grep -i "out of memory\|oom\|killed" /var/log/messages

# Step 7: Check if a service was restarted around that time
grep "Started\|Stopped\|Failed" /var/log/messages | grep "Mar 21 10:3"

# Step 8: Check for hardware/kernel errors
dmesg -T | grep -i "error\|fail\|warn" | grep "Mar 21 10:3"

# Step 9: Correlate with systemd journal
journalctl --since "2025-03-21 10:30:00" --until "2025-03-21 10:45:00" -p warning

# Step 10: Document the timeline and root cause
# Write a post-mortem with: what happened, why, how to prevent recurrence
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Searching only the current log file | Check rotated files: `/var/log/messages*` and `.gz` with `zgrep` |
| High noise — too many false positives | Use `-E "error" -v "level=error"` to refine or narrow to specific service |
| Missing the root cause by looking at symptoms | Always search for the **first** error chronologically — not the most recent |
| `grep` too slow on large log files | Use `journalctl` with `--since`/`--until` for fast indexed queries |
| Correlation failed — timestamps not aligned | Check server NTP/timezone: `timedatectl` |
| Logs rotated away | Set up `logrotate` retention or increase journal size in `journald.conf` |

---

## 🔑 Key Takeaways

- `grep -i error /var/log/messages` is the starting point — always add **time filtering** and **context lines** (`-C 5`) to get the full picture.
- The **first chronological error** is usually the root cause — later errors are often cascading consequences.
- Use `grep -iE "error|fail|critical"` to cast a wider net and catch all severity keywords.
- **Correlate across multiple log files** at the same timestamp — the root cause is often in a different log than the symptom.
- `uniq -c | sort -rn` on error messages reveals which errors are most frequent — a sudden spike at a specific time marks the incident start.
- Use `journalctl` for indexed, structured queries instead of raw `grep` on flat files when possible.
- Always search **rotated and compressed** logs (`zgrep`) — incidents often start before the current log file covers.

---

## 📚 Related Concepts

- [logrotate](https://linux.die.net/man/8/logrotate) — automated log rotation and retention
- [GoAccess](https://goaccess.io/) — real-time web log analyzer
- [ELK Stack](https://www.elastic.co/elastic-stack) — Elasticsearch + Logstash + Kibana for large-scale log analysis


</details>
