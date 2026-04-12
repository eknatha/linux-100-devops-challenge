# Challenge 59: Monitor Disk Space and Send Alerts

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Disk space exhaustion is one of the most common causes of system failures, service crashes, and data loss. Proactively **monitoring disk usage** and alerting when thresholds are exceeded is a critical sysadmin responsibility. Linux provides powerful built-in tools — `df`, `du`, and `awk` — that make it straightforward to build lightweight, effective disk monitoring scripts.

---

## 🧩 Task

Print an alert for any filesystem whose disk usage exceeds 80%.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
df -h | awk '$5 > 80 {print "Disk Full: " $0}'
```

### How it works

| Part | Description |
|------|-------------|
| `df -h` | Display disk filesystem usage in **human-readable** format (KB, MB, GB) |
| `\|` | Pipe `df` output into `awk` for processing |
| `awk` | Pattern scanning and processing tool |
| `'$5 > 80'` | Condition — check if column 5 (Use%) is greater than 80 |
| `{print "Disk Full: " $0}` | Action — print the alert message with the full matching line |
| `$0` | The entire current line |
| `$5` | The 5th column of `df` output — the **Use%** column |

#### `df -h` Output Columns

```
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   43G    7G    86%  /
tmpfs           1.9G     0  1.9G     0%  /dev/shm
/dev/sdb1       100G   20G   80G    20%  /data
```

| Column | `$` Reference | Content |
|--------|--------------|---------|
| Filesystem | `$1` | Device or filesystem name |
| Size | `$2` | Total size |
| Used | `$3` | Space used |
| Avail | `$4` | Space available |
| Use% | `$5` | Percentage used ← checked in the solution |
| Mounted on | `$6` | Mount point |

---

## 📖 Extended Examples

### Example 1 — Basic Alert for Usage Over 80%

```bash
df -h | awk '$5 > 80 {print "Disk Full: " $0}'
```

```bash
Disk Full: /dev/sda1   50G   43G   7G  86%  /
```

---

### Example 2 — Strip the `%` Sign for Clean Numeric Comparison

The `%` character in `Use%` can cause issues with direct numeric comparison in some `awk` versions. Strip it explicitly:

```bash
df -h | awk 'NR>1 {gsub(/%/, "", $5); if ($5+0 > 80) print "⚠️  Alert: " $6 " is at " $5 "% usage"}'
```

```bash
⚠️  Alert: / is at 86% usage
```

| Part | Description |
|------|-------------|
| `NR>1` | Skip the header row (row number > 1) |
| `gsub(/%/, "", $5)` | Remove `%` from column 5 |
| `$5+0` | Force numeric comparison |
| `$6` | Mount point for a cleaner message |

---

### Example 3 — Custom Threshold Variable

```bash
THRESHOLD=80
df -h | awk -v threshold="$THRESHOLD" 'NR>1 {gsub(/%/,"", $5); if ($5+0 > threshold) print "⚠️  ALERT: "$6" usage is "$5"%"}'
```

```bash
⚠️  ALERT: / usage is 86%
```

> 💡 `-v` passes a shell variable into `awk`. Change `THRESHOLD` to adjust the alert level without modifying the `awk` logic.

---

### Example 4 — Alert with Timestamp

```bash
df -h | awk 'NR>1 {gsub(/%/,"", $5); if ($5+0 > 80) print strftime("[%Y-%m-%d %H:%M:%S]", systime()) " DISK ALERT: "$6" at "$5"%"}'
```

```bash
[2025-03-21 02:00:05] DISK ALERT: / at 86%
```

---

### Example 5 — Check a Specific Partition Only

```bash
df -h /var | awk 'NR>1 {gsub(/%/,"", $5); if ($5+0 > 80) print "ALERT: /var is at "$5"% capacity"}'
```

> Pass a specific path to `df` to monitor only that partition.

---

### Example 6 — Output Alerts to a Log File

```bash
df -h | awk 'NR>1 {gsub(/%/,"", $5); if ($5+0 > 80) print "DISK ALERT: "$6" at "$5"%"}' \
  >> /var/log/disk_alerts.log
```

```bash
# View the log
tail -f /var/log/disk_alerts.log
```

---

### Example 7 — Send an Email Alert

```bash
#!/bin/bash
THRESHOLD=80
ALERT=$(df -h | awk -v t="$THRESHOLD" 'NR>1 {gsub(/%/,"", $5); if ($5+0 > t) print "ALERT: "$6" is at "$5"%"}')

if [ -n "$ALERT" ]; then
  echo "$ALERT" | mail -s "⚠️ Disk Space Alert on $(hostname)" admin@example.com
fi
```

> Requires `mailutils` or `sendmail` to be installed and configured.

---

### Example 8 — Full Disk Monitor Script with Logging and Email

```bash
#!/bin/bash

# ── Configuration ────────────────────────────────────────
THRESHOLD=80
LOGFILE="/var/log/disk_monitor.log"
EMAIL="admin@example.com"
HOSTNAME=$(hostname)
DATE=$(date +"%Y-%m-%d %H:%M:%S")
ALERT_FOUND=0

# ── Check disk usage ─────────────────────────────────────
while IFS= read -r line; do
  USAGE=$(echo "$line" | awk '{gsub(/%/,"", $5); print $5+0}')
  MOUNT=$(echo "$line"  | awk '{print $6}')

  if [ "$USAGE" -gt "$THRESHOLD" ]; then
    MESSAGE="[$DATE] ⚠️  ALERT: $MOUNT is at ${USAGE}% on $HOSTNAME"
    echo "$MESSAGE" | tee -a "$LOGFILE"
    ALERT_FOUND=1
  fi
done < <(df -h | tail -n +2)

# ── Send email if any alert was triggered ─────────────────
if [ "$ALERT_FOUND" -eq 1 ]; then
  tail -20 "$LOGFILE" | mail -s "Disk Alert: $HOSTNAME" "$EMAIL"
fi
```

---

### Example 9 — Find the Largest Directories Contributing to Disk Usage

```bash
# Top 10 largest directories under /var/log
du -sh /var/log/* 2>/dev/null | sort -rh | head -10
```

```bash
512M  /var/log/journal
120M  /var/log/syslog
 48M  /var/log/auth.log
 12M  /var/log/kern.log
```

| Part | Description |
|------|-------------|
| `du -sh` | Disk usage — **s**ummary, **h**uman-readable |
| `sort -rh` | Sort in **r**everse **h**uman-readable order (largest first) |
| `head -10` | Show top 10 results |

---

### Example 10 — Automate with Cron (Every Hour)

```bash
crontab -e
```

```bash
# Run disk monitor every hour
0 * * * * /usr/local/bin/disk_monitor.sh >> /var/log/disk_monitor.log 2>&1
```

---

## 🗂️ `df` vs `du` — When to Use Which

| Tool | Full Name | Use For |
|------|-----------|---------|
| `df` | Disk Free | Check **filesystem-level** usage and free space |
| `du` | Disk Usage | Drill down into **directory/file-level** usage |

```bash
# df — filesystem overview
df -h

# du — find what is consuming space
du -sh /var/log/*
du -sh /* 2>/dev/null | sort -rh | head -15
```

---

## 🔢 `df` Useful Flags

| Flag | Description |
|------|-------------|
| `-h` | Human-readable sizes (KB, MB, GB) |
| `-H` | Human-readable using powers of 1000 instead of 1024 |
| `-T` | Show filesystem type (ext4, xfs, tmpfs) |
| `-i` | Show inode usage instead of block usage |
| `--total` | Add a total line at the bottom |
| `-x tmpfs` | Exclude a specific filesystem type |

```bash
# Show filesystem types
df -hT

# Check inode usage (full inodes = no new files even with free space)
df -i

# Exclude tmpfs from output
df -h -x tmpfs
```

---

## 🚀 Full Workflow — Set Up Automated Disk Monitoring

```bash
# Step 1: Test the one-liner manually
df -h | awk '$5 > 80 {print "Disk Full: " $0}'

# Step 2: Create the monitor script
sudo nano /usr/local/bin/disk_monitor.sh

# Step 3: Make it executable
sudo chmod +x /usr/local/bin/disk_monitor.sh

# Step 4: Test the script
/usr/local/bin/disk_monitor.sh

# Step 5: Schedule with cron (every hour)
crontab -e
# Add: 0 * * * * /usr/local/bin/disk_monitor.sh >> /var/log/disk_monitor.log 2>&1

# Step 6: Monitor the log
tail -f /var/log/disk_monitor.log
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `$5 > 80` matching `Use%` header | Add `NR>1` to skip the header row |
| `%` sign causing incorrect numeric comparison | Use `gsub(/%/,"", $5)` to strip it before comparing |
| Alert fires but disk is actually fine | Check if `tmpfs` or loop devices are triggering it — use `-x tmpfs` |
| Disk full but `df` shows space available | Check inode exhaustion with `df -i` |
| Script runs in cron but no email arrives | Verify `mailutils` is installed and SMTP is configured |
| Large log files filling disk silently | Set up `logrotate` alongside disk monitoring |

---

## 🔑 Key Takeaways

- `df -h` shows **filesystem-level** disk usage; `du` drills into directories.
- `awk '$5 > 80'` filters rows where the **Use%** column exceeds the threshold.
- Always use `NR>1` to **skip the header row** and `gsub(/%/,"", $5)` to enable clean numeric comparison.
- Pass thresholds as `-v` variables in `awk` to keep scripts flexible and reusable.
- **Inode exhaustion** (`df -i`) can fill a disk even when block space appears free.
- Combine the script with **cron + email** for fully automated, hands-free monitoring.

---

## 📚 Related Concepts

- [GNU awk Manual](https://www.gnu.org/software/gawk/manual/)
- [logrotate](https://linux.die.net/man/8/logrotate) — prevent logs from consuming all disk space
- [ncdu](https://dev.yorhel.nl/ncdu) — interactive disk usage analyzer for the terminal

</details>
