# Challenge 85: Handle Full Disk Scenario in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

A **full disk** is a critical incident — when a Linux filesystem reaches 100% capacity, services begin to fail: web servers stop logging, databases cannot write transactions, and users cannot create files or even log in. Resolving a full disk quickly requires a systematic approach: identify what is consuming space, safely remove what is no longer needed, and prevent recurrence.

`/var/log/` is the most common culprit — log files grow continuously unless managed by `logrotate` or regular cleanup.

---

## 🧩 Task

Diagnose a full disk scenario and safely free space by identifying and removing large or old log files.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Find what is consuming space in /var/log
du -sh /var/log/*

# Remove old log files (verify before deleting!)
rm -rf old_logs
```

### How it works

| Part | Description |
|------|-------------|
| `du -sh /var/log/*` | Show the **s**ize **s**ummary of each item in `/var/log/` in **h**uman-readable format |
| `-s` | Summary — one total per argument, not per subdirectory |
| `-h` | Human-readable (KB, MB, GB) |
| `rm -rf old_logs` | **Remove** the specified path **r**ecursively and **f**orcefully |

> ⚠️ Always verify what you are deleting before running `rm -rf`. There is no undo.

---

## 🔢 Understanding `du -sh` Output

```bash
du -sh /var/log/*
```

```
 18G  /var/log/journal
6.0G  /var/log/nginx
3.0G  /var/log/mysql
512M  /var/log/syslog
 40M  /var/log/auth.log
 12M  /var/log/kern.log
4.0K  /var/log/cron
```

> `/var/log/journal` (systemd journal) and `/var/log/nginx` are the biggest consumers — these are the first candidates for cleanup.

---

## 📖 Extended Examples

### Example 1 — Check Disk Usage First

```bash
# Confirm which filesystem is full
df -h

# Find the biggest consumers at root level
du -sh /* 2>/dev/null | sort -hr | head -10

# Drill into /var/log
du -sh /var/log/* 2>/dev/null | sort -hr | head -10
```

```bash
# df -h output showing full disk:
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   50G     0   100%  /
```

---

### Example 2 — Find the Largest Individual Log Files

```bash
# Find files over 100MB in /var/log
find /var/log -type f -size +100M -exec ls -lh {} \;

# Find the top 10 largest files in /var/log
find /var/log -type f -printf '%s %p\n' 2>/dev/null | \
  sort -rn | head -10 | \
  awk '{printf "%.0f MB  %s\n", $1/1048576, $2}'
```

```bash
 6144 MB  /var/log/nginx/access.log
 3072 MB  /var/log/mysql/slow.log
  512 MB  /var/log/syslog
```

---

### Example 3 — Safe Cleanup Methods (Preserve the File, Clear Contents)

For files actively written by running services, **never delete the file directly** — the process holds it open and deletion does not free space until the service is reloaded. Instead, **truncate** the file:

```bash
# Truncate a log file to zero bytes (safest — file remains, content cleared)
sudo truncate -s 0 /var/log/nginx/access.log

# Alternative: redirect empty output (same effect)
sudo > /var/log/nginx/access.log

# Then reload the service so it reopens the log
sudo systemctl reload nginx
```

> 💡 **Truncation** is safer than `rm` for active log files — it preserves the file descriptor so the service continues writing without needing a restart.

---

### Example 4 — Remove Old Rotated Log Files

Log files with `.gz` or numbered extensions are rotated backups — usually safe to remove after a retention period:

```bash
# List rotated logs (compressed and numbered)
ls -lh /var/log/*.gz /var/log/*.1 /var/log/*.2 2>/dev/null

# Remove compressed logs older than 30 days
find /var/log -name "*.gz" -mtime +30 -delete

# Remove numbered rotated logs
find /var/log -name "*.1" -o -name "*.2" -o -name "*.3" | xargs rm -f

# Remove old nginx logs
find /var/log/nginx -name "*.log.*" -mtime +7 -delete
find /var/log/nginx -name "*.gz" -mtime +7 -delete
```

---

### Example 5 — Trim the systemd Journal

The systemd journal (`/var/log/journal/`) often grows to gigabytes over time:

```bash
# Check current journal size
journalctl --disk-usage

# Trim to a maximum size
sudo journalctl --vacuum-size=500M

# Trim to keep only the last 7 days
sudo journalctl --vacuum-time=7d

# Trim to keep only the last 2 weeks
sudo journalctl --vacuum-time=2weeks

# Verify the result
journalctl --disk-usage
```

```bash
# Before:
Archived and active journals take up 18.2G in the file system.

# After --vacuum-size=500M:
Archived and active journals take up 498.2M in the file system.
```

---

### Example 6 — Remove Old Application Logs Safely

```bash
# Identify old log files (not accessed in 30 days)
find /var/log -type f -atime +30 -name "*.log" | head -20

# Review the list first
find /var/log -type f -atime +30 -name "*.log" -exec ls -lh {} \;

# Delete only after confirming
find /var/log -type f -atime +30 -name "*.log" -delete

# Remove a specific application's old logs
rm -f /var/log/myapp/app-2025-01-*.log
rm -f /var/log/myapp/app-2025-02-*.log
```

---

### Example 7 — Check for and Remove Core Dump Files

Core dumps from crashed applications can be enormous:

```bash
# Find core dump files
find / -name "core" -o -name "core.[0-9]*" 2>/dev/null | xargs ls -lh 2>/dev/null

# Find core dumps managed by systemd
coredumpctl list

# Remove old core dumps
sudo journalctl --vacuum-time=1d   # systemd-coredump entries
find / -name "core.[0-9]*" -mtime +7 -delete 2>/dev/null
```

---

### Example 8 — Clean Package Manager Caches

Package managers maintain large caches that can be safely cleared:

```bash
# Debian/Ubuntu — apt cache
sudo apt clean                    # Remove all cached packages
sudo apt autoclean                # Remove only outdated cached packages
sudo apt autoremove               # Remove orphaned packages
du -sh /var/cache/apt/archives/

# RHEL/CentOS/Fedora — dnf/yum cache
sudo dnf clean all
sudo yum clean all
du -sh /var/cache/dnf/

# Docker (if applicable)
docker system prune -f
docker system df
```

---

### Example 9 — Find and Remove Temporary Files

```bash
# Large files in /tmp
find /tmp -type f -size +50M -exec ls -lh {} \;

# Old temp files not accessed in 7 days
find /tmp /var/tmp -type f -atime +7 -delete

# Clear pip cache (Python)
pip cache purge

# Clear npm cache
npm cache clean --force
```

---

### Example 10 — Emergency Space Recovery One-Liner

When a disk hits 100% and services are failing, run this to quickly identify and free the most space:

```bash
# Emergency: find top consumers, trim journal, clear apt cache
echo "=== Top Space Consumers ===" && \
  du -sh /* 2>/dev/null | sort -hr | head -5 && \
  echo "=== Trimming Journal ===" && \
  sudo journalctl --vacuum-size=200M && \
  echo "=== Clearing apt cache ===" && \
  sudo apt clean 2>/dev/null || sudo dnf clean all 2>/dev/null && \
  echo "=== Truncating large nginx logs ===" && \
  sudo truncate -s 0 /var/log/nginx/access.log 2>/dev/null && \
  sudo systemctl reload nginx 2>/dev/null && \
  echo "=== Current disk usage ===" && \
  df -h
```

---

### Example 11 — Set Up logrotate to Prevent Recurrence

The permanent fix is automated log rotation:

```bash
# Check existing logrotate configuration
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Create a custom logrotate rule for an application
sudo nano /etc/logrotate.d/myapp
```

```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily               # Rotate daily
    rotate 7            # Keep 7 rotated files
    compress            # Compress with gzip
    delaycompress       # Don't compress the most recent rotation
    missingok           # Don't error if log file is missing
    notifempty          # Don't rotate if log is empty
    create 640 myapp myapp  # New file permissions and ownership
    postrotate
        systemctl reload myapp 2>/dev/null || true
    endscript
}
```

```bash
# Test the logrotate config
sudo logrotate --debug /etc/logrotate.d/myapp

# Force a rotation now (test)
sudo logrotate --force /etc/logrotate.d/myapp
```

---

### Example 12 — Full Disk Recovery Script

```bash
#!/bin/bash

THRESHOLD=85           # Target: bring usage below this %
LOG="/tmp/disk_recovery_$(date +%Y%m%d_%H%M%S).txt"

echo "============================================" | tee "$LOG"
echo " Full Disk Recovery — $(date)"               | tee -a "$LOG"
echo "============================================" | tee -a "$LOG"

echo -e "\n[Before] Disk Usage:"                   | tee -a "$LOG"
df -h                                               | tee -a "$LOG"

echo -e "\n[1] Trimming systemd journal to 500MB]" | tee -a "$LOG"
sudo journalctl --vacuum-size=500M                  | tee -a "$LOG"

echo -e "\n[2] Removing compressed logs > 30 days]"| tee -a "$LOG"
find /var/log -name "*.gz" -mtime +30 -delete -print | tee -a "$LOG"

echo -e "\n[3] Removing rotated numbered logs]"    | tee -a "$LOG"
find /var/log -name "*.1" -o -name "*.2" -o -name "*.3" 2>/dev/null | \
  xargs rm -f 2>/dev/null && echo "Done"           | tee -a "$LOG"

echo -e "\n[4] Clearing package manager cache]"    | tee -a "$LOG"
sudo apt clean 2>/dev/null || sudo dnf clean all 2>/dev/null
echo "Done"                                         | tee -a "$LOG"

echo -e "\n[5] Clearing /tmp files > 7 days old]"  | tee -a "$LOG"
find /tmp /var/tmp -type f -atime +7 -delete -print 2>/dev/null | tee -a "$LOG"

echo -e "\n[After] Disk Usage:"                    | tee -a "$LOG"
df -h                                               | tee -a "$LOG"

echo -e "\nReport saved: $LOG"
```

---

## 🗂️ Space Recovery Methods — Quick Reference

| Method | Command | Space Freed | Safety |
|--------|---------|-------------|--------|
| Trim journal | `journalctl --vacuum-size=500M` | 1–20 GB | ✅ Safe |
| Remove compressed logs | `find /var/log -name "*.gz" -mtime +30 -delete` | Varies | ✅ Safe |
| Truncate active log | `truncate -s 0 /var/log/app/app.log` | Up to GB | ✅ Safe |
| Clear apt/dnf cache | `apt clean` / `dnf clean all` | 100MB–5GB | ✅ Safe |
| Clear /tmp files | `find /tmp -atime +7 -delete` | Varies | ✅ Usually safe |
| Docker prune | `docker system prune` | 1–50 GB | ⚠️ Removes unused images |
| Remove core dumps | `find / -name "core.*" -delete` | Varies | ✅ Safe if not debugging |
| Force logrotate | `logrotate --force /etc/logrotate.conf` | Varies | ✅ Safe |

---

## 🚀 Full Workflow — Handle a Full Disk Incident

```bash
# Step 1: Confirm the disk is full
df -h

# Step 2: Find what is consuming space
du -sh /* 2>/dev/null | sort -hr | head -10
du -sh /var/log/* 2>/dev/null | sort -hr | head -10

# Step 3: Check for deleted-but-open files (may be holding space)
lsof | grep deleted | awk '{print $7, $1, $2}'

# Step 4: Quick wins — trim journal and clear caches
sudo journalctl --vacuum-size=500M
sudo apt clean 2>/dev/null || sudo dnf clean all 2>/dev/null

# Step 5: Find and remove old rotated logs
find /var/log -name "*.gz" -mtime +30 -delete

# Step 6: Truncate oversized active log files
sudo truncate -s 0 /var/log/nginx/access.log
sudo systemctl reload nginx

# Step 7: Verify space is recovered
df -h

# Step 8: Set up logrotate to prevent recurrence
sudo nano /etc/logrotate.d/myapp

# Step 9: Monitor disk usage going forward
# Add to crontab:
# 0 * * * * df -h | awk '$5 > "85%"' | mail -s "Disk Alert" admin@example.com
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `rm` on active log doesn't free space | Use `truncate -s 0` then reload the service |
| Deleting without checking content first | Always run `ls -lh` or `head` on files before `rm` |
| `rm -rf` on wrong directory | Double-check path with `ls` before running `rm -rf` |
| Disk still full after deletion | Check for deleted-but-open files: `lsof \| grep deleted` |
| No space to even run recovery commands | Use `df -i` to check inodes; use `truncate` instead of `cp` |
| logrotate not configured for custom apps | Add `/etc/logrotate.d/appname` config for each application |

---

## 🔑 Key Takeaways

- `du -sh /var/log/* | sort -hr` immediately shows which logs are consuming the most space.
- For **active log files being written by services**, use `truncate -s 0` + service reload — never `rm`.
- The **systemd journal** is often the biggest hidden consumer — `journalctl --vacuum-size=500M` can free gigabytes instantly.
- If `df` shows full but `du` shows less space, check for **deleted-but-open files** with `lsof | grep deleted`.
- Always **verify before deleting** — run `ls -lh` and `head` on files before `rm`.
- The **permanent fix** is `logrotate` — configure it for every application that writes to `/var/log/`.
- Recover to **below 80% usage** — services begin failing unpredictably as disks approach 100%.

---

## 📚 Related Concepts

- [Challenge 59 — Monitor Disk Space Alert]
- [Challenge 58 — Archive Logs Automatically]
- [Challenge 73 — Disk Space Issue]
- [Challenge 84 — Recover Deleted Files]
- [logrotate Manual](https://linux.die.net/man/8/logrotate)
- [GNU du Manual](https://www.gnu.org/software/coreutils/manual/html_node/du-invocation.html)


</details>

