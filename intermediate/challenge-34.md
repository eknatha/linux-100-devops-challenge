# Challenge 34: Configure a Cron Job in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Cron** is Linux's built-in job scheduler — it runs commands or scripts automatically at specified times and intervals without any manual intervention. From running a backup every night to cleaning temp files every hour, cron automates repetitive tasks reliably.

The schedule is defined in a **crontab** (cron table) file using a five-field time expression that specifies the exact minute, hour, day, month, and weekday to run a job.

---

## 🧩 Task

Schedule a command to run automatically every minute using cron.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Open your crontab for editing
crontab -e

# Add this line to run a command every minute
* * * * * /path/to/command
```

### How it works

| Part | Description |
|------|-------------|
| `crontab -e` | Open the current user's crontab in an editor |
| `* * * * *` | Cron time expression — every minute |
| `/path/to/command` | The command or script to run |

---

## 🔢 The Cron Time Expression

```
┌─────────── Minute       (0–59)
│ ┌───────── Hour         (0–23)
│ │ ┌─────── Day of Month (1–31)
│ │ │ ┌───── Month        (1–12)
│ │ │ │ ┌─── Day of Week  (0–7)  [0 and 7 = Sunday]
│ │ │ │ │
* * * * *  command
```

| Field | Values | Special Characters |
|-------|--------|-------------------|
| Minute | 0–59 | `*`, `,`, `-`, `*/n` |
| Hour | 0–23 | `*`, `,`, `-`, `*/n` |
| Day of Month | 1–31 | `*`, `,`, `-`, `*/n` |
| Month | 1–12 | `*`, `,`, `-`, `*/n` |
| Day of Week | 0–7 (0=Sun) | `*`, `,`, `-`, `*/n` |

| Character | Meaning |
|-----------|---------|
| `*` | Every value |
| `,` | List (e.g., `1,3,5`) |
| `-` | Range (e.g., `1-5`) |
| `*/n` | Every nth value (e.g., `*/5` = every 5) |

---

## 📖 Extended Examples

### Example 1 — Run Every Minute

```bash
crontab -e
```

```bash
* * * * * echo "Hello" >> /tmp/cron_test.log
```

> Every minute, every hour, every day — the job runs 1,440 times per day.

---

### Example 2 — Run Every Hour

```bash
0 * * * * /usr/local/bin/hourly_cleanup.sh
```

> Runs at the **start of every hour** (00:00, 01:00, 02:00...).

---

### Example 3 — Run Every Day at Midnight

```bash
0 0 * * * /usr/local/bin/daily_backup.sh
```

---

### Example 4 — Run Every 5 Minutes

```bash
*/5 * * * * /usr/local/bin/health_check.sh
```

> `*/5` means every 5th minute: :00, :05, :10, :15, :20...

---

### Example 5 — Run at a Specific Time

```bash
# Run at 2:30 AM every day
30 2 * * * /usr/local/bin/nightly_report.sh

# Run at 9:00 AM on weekdays (Mon-Fri)
0 9 * * 1-5 /usr/local/bin/morning_report.sh

# Run on the 1st of every month at 6 AM
0 6 1 * * /usr/local/bin/monthly_invoice.sh
```

---

### Example 6 — Run at Multiple Times

```bash
# Run at 8 AM, 12 PM, and 6 PM
0 8,12,18 * * * /usr/local/bin/send_report.sh

# Run every 15 minutes
*/15 * * * * /usr/local/bin/sync_data.sh

# Run every Monday and Wednesday at 9 AM
0 9 * * 1,3 /usr/local/bin/team_report.sh
```

---

### Example 7 — Common Cron Shorthand (`@` Syntax)

```bash
@reboot   /usr/local/bin/startup.sh        # Run once at system boot
@hourly   /usr/local/bin/cleanup.sh        # = 0 * * * *
@daily    /usr/local/bin/backup.sh         # = 0 0 * * *
@weekly   /usr/local/bin/weekly_check.sh   # = 0 0 * * 0
@monthly  /usr/local/bin/monthly_job.sh    # = 0 0 1 * *
@yearly   /usr/local/bin/annual_job.sh     # = 0 0 1 1 *
```

| Shorthand | Equivalent | Description |
|-----------|------------|-------------|
| `@reboot` | — | Once at system startup |
| `@hourly` | `0 * * * *` | Every hour |
| `@daily` | `0 0 * * *` | Every day at midnight |
| `@weekly` | `0 0 * * 0` | Every Sunday at midnight |
| `@monthly` | `0 0 1 * *` | 1st of every month |
| `@yearly` | `0 0 1 1 *` | January 1st every year |

---

### Example 8 — Redirect Output to a Log File

By default, cron sends output via email. Redirect it to a file instead:

```bash
# Redirect both stdout and stderr to a log file
* * * * * /usr/local/bin/script.sh >> /var/log/myjob.log 2>&1

# Redirect to a timestamped entry
* * * * * echo "[$(date)] Job ran" >> /var/log/myjob.log 2>&1

# Discard all output (silent)
* * * * * /usr/local/bin/script.sh > /dev/null 2>&1
```

> 💡 Always redirect output to a log file for cron jobs — you need a record of what happened and any errors.

---

### Example 9 — Manage Crontabs

```bash
# Edit your crontab
crontab -e

# List your current cron jobs
crontab -l

# Remove all your cron jobs (use with caution!)
crontab -r

# Edit another user's crontab (requires root)
sudo crontab -u username -e

# List another user's crontab
sudo crontab -u username -l
```

---

### Example 10 — System-Wide Cron Files

In addition to user crontabs, system-wide cron jobs can be placed in:

```bash
# System crontab (has username field)
sudo nano /etc/crontab

# Drop-in directories (scripts placed here run automatically)
ls /etc/cron.d/        # Custom cron files
ls /etc/cron.hourly/   # Scripts that run hourly
ls /etc/cron.daily/    # Scripts that run daily
ls /etc/cron.weekly/   # Scripts that run weekly
ls /etc/cron.monthly/  # Scripts that run monthly

# Place a script in the daily run directory
sudo cp myscript.sh /etc/cron.daily/
sudo chmod +x /etc/cron.daily/myscript.sh
```

```bash
# /etc/crontab format (includes username):
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || run-parts --report /etc/cron.daily
```

---

### Example 11 — A Complete Cron Job Script

```bash
#!/bin/bash
# /usr/local/bin/disk_check.sh

THRESHOLD=85
LOGFILE="/var/log/disk_check.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Check disk usage
df -h | awk -v t="$THRESHOLD" -v d="$DATE" 'NR>1 {
  gsub(/%/, "", $5)
  if ($5+0 > t)
    print "[" d "] ⚠️  ALERT: " $6 " is at " $5 "% full"
}' | tee -a "$LOGFILE"
```

```bash
# Schedule it to run every 30 minutes
crontab -e
```

```bash
# Add this line:
*/30 * * * * /usr/local/bin/disk_check.sh >> /var/log/disk_check.log 2>&1
```

---

### Example 12 — Debug Cron Jobs

Cron jobs run in a minimal environment — PATH and other variables may differ from your shell:

```bash
# Check what PATH cron uses
* * * * * env >> /tmp/cron_env.log

# Always use full paths in cron scripts
* * * * * /usr/bin/python3 /opt/app/script.py   # ✅ Full path
* * * * * python3 /opt/app/script.py             # ❌ May fail if PATH is wrong

# Set PATH at the top of your crontab
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Check cron logs for errors
grep CRON /var/log/syslog | tail -20        # Debian/Ubuntu
grep CRON /var/log/cron | tail -20          # RHEL/CentOS
journalctl -u cron | tail -20               # systemd
```

---

## 🗂️ Common Cron Schedules — Quick Reference

| Schedule | Expression | Description |
|----------|------------|-------------|
| Every minute | `* * * * *` | 1,440 times per day |
| Every 5 minutes | `*/5 * * * *` | 288 times per day |
| Every 15 minutes | `*/15 * * * *` | 96 times per day |
| Every 30 minutes | `*/30 * * * *` | 48 times per day |
| Every hour | `0 * * * *` | 24 times per day |
| Every day at midnight | `0 0 * * *` | Once per day |
| Every day at 2 AM | `0 2 * * *` | Once per day |
| Every Monday at 9 AM | `0 9 * * 1` | Once per week |
| 1st of month at 6 AM | `0 6 1 * *` | Once per month |
| At system boot | `@reboot` | Once per boot |

---

## 🚀 Full Workflow — Set Up a Monitoring Cron Job

```bash
# Step 1: Create the script
sudo nano /usr/local/bin/health_check.sh

# Step 2: Make it executable
sudo chmod +x /usr/local/bin/health_check.sh

# Step 3: Test it manually first
/usr/local/bin/health_check.sh

# Step 4: Open crontab
crontab -e

# Step 5: Add the schedule (every 5 minutes)
*/5 * * * * /usr/local/bin/health_check.sh >> /var/log/health_check.log 2>&1

# Step 6: Verify it was saved
crontab -l

# Step 7: Wait and check the log
tail -f /var/log/health_check.log

# Step 8: Check cron system logs if not running
grep CRON /var/log/syslog | tail -10
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Script works manually but fails in cron | Use **full paths** — cron has a minimal `PATH` |
| No output — can't tell if it ran | Always redirect: `>> /path/to/log 2>&1` |
| Cron job not triggering | Check with `crontab -l`; verify cron service: `systemctl status cron` |
| Forgetting `chmod +x` on the script | Run `chmod +x /path/to/script.sh` |
| Script has Windows line endings | Fix with `dos2unix script.sh` or `sed -i 's/\r//' script.sh` |
| Email floods from cron output | Add `> /dev/null 2>&1` or `MAILTO=""` at top of crontab |

---

## 🔑 Key Takeaways

- The cron time expression has **five fields**: minute, hour, day-of-month, month, day-of-week.
- `* * * * *` runs **every minute** — the most frequent possible schedule.
- `*/5` means **every 5th** unit — `*/5 * * * *` runs every 5 minutes.
- Always use **full paths** in cron jobs — the `PATH` environment is minimal compared to your shell.
- Always **redirect output** to a log file (`>> /log/file 2>&1`) — cron output is lost otherwise.
- Use **`@shorthand`** (`@daily`, `@reboot`, `@hourly`) for cleaner, more readable schedules.
- Test scripts **manually first** before scheduling — `crontab -l` to verify the job was saved.
- Check **`/var/log/syslog`** or **`journalctl -u cron`** if a cron job is not running as expected.

---

## 📚 Related Concepts

- [GNU crontab Manual](https://linux.die.net/man/5/crontab)
- [Crontab Guru — Visual Cron Editor](https://crontab.guru)

</details>
