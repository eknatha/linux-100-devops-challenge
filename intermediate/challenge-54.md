# Challenge 54: Schedule a Script with Cron

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Cron** is a time-based job scheduler built into Unix/Linux systems. It allows you to **automate scripts and commands** to run at fixed times, dates, or intervals — without any manual trigger. The scheduled jobs are called **cron jobs**, and they are managed through a configuration file called the **crontab**.

---

## 🧩 Task

Schedule a script to run automatically every day at midnight.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Open the crontab editor
crontab -e

# Add this line to run the script every day at midnight
0 0 * * * /path/to/script.sh
```

### How it works

```
┌─────────── Minute       (0 - 59)
│ ┌───────── Hour         (0 - 23)
│ │ ┌─────── Day of Month (1 - 31)
│ │ │ ┌───── Month        (1 - 12)
│ │ │ │ ┌─── Day of Week  (0 - 7)  (0 and 7 = Sunday)
│ │ │ │ │
* * * * *  command_to_execute
0 0 * * *  /path/to/script.sh
```

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | `0` | At minute 0 |
| Hour | `0` | At hour 0 (midnight) |
| Day of Month | `*` | Every day of the month |
| Month | `*` | Every month |
| Day of Week | `*` | Every day of the week |

---

## 📖 Extended Examples

### Example 1 — Run Every Day at Midnight

```bash
0 0 * * * /path/to/script.sh
```
> Runs at **00:00** every day.

---

### Example 2 — Run Every Hour

```bash
0 * * * * /path/to/script.sh
```
> Runs at the **start of every hour** (00:00, 01:00, 02:00 ...).

---

### Example 3 — Run Every 15 Minutes

```bash
*/15 * * * * /path/to/script.sh
```
> Runs at **:00, :15, :30, :45** of every hour.

---

### Example 4 — Run Every Monday at 9 AM

```bash
0 9 * * 1 /path/to/script.sh
```
> Runs at **09:00 every Monday**. (`1` = Monday in cron's 0–7 scale)

---

### Example 5 — Run on the 1st of Every Month

```bash
0 8 1 * * /path/to/script.sh
```
> Runs at **08:00 on the 1st of every month**.

---

### Example 6 — Run on Weekdays Only (Mon–Fri)

```bash
0 9 * * 1-5 /path/to/script.sh
```
> Runs at **09:00, Monday through Friday**.

---

### Example 7 — Run Multiple Times a Day

```bash
0 8,12,18 * * * /path/to/script.sh
```
> Runs at **08:00, 12:00, and 18:00** every day.

---

### Example 8 — Run Every 5 Minutes

```bash
*/5 * * * * /path/to/script.sh
```
> Runs **every 5 minutes**, all day, every day.

---

### Example 9 — Run at a Specific Date and Time

```bash
30 14 25 12 * /path/to/script.sh
```
> Runs at **14:30 on December 25th** every year.

---

### Example 10 — Redirect Output to a Log File

```bash
0 0 * * * /path/to/script.sh >> /var/log/myjob.log 2>&1
```
> Runs daily at midnight and **appends both stdout and stderr** to a log file.

---

## ⚡ Cron Special Strings

Instead of the five fields, cron also accepts handy shortcuts:

| Shortcut | Equivalent | Meaning |
|----------|------------|---------|
| `@reboot` | — | Runs once at system startup |
| `@hourly` | `0 * * * *` | Runs once every hour |
| `@daily` | `0 0 * * *` | Runs once every day at midnight |
| `@weekly` | `0 0 * * 0` | Runs once every Sunday at midnight |
| `@monthly` | `0 0 1 * *` | Runs once on the 1st of every month |
| `@yearly` | `0 0 1 1 *` | Runs once on January 1st at midnight |

```bash
# Using special strings
@daily /path/to/backup.sh
@reboot /path/to/startup.sh
```

---

## 🛠️ Managing Crontabs

```bash
# Open and edit your crontab
crontab -e

# List all current cron jobs
crontab -l

# Remove all cron jobs (use with caution!)
crontab -r

# Edit crontab for a specific user (requires root)
sudo crontab -u username -e
```

---

## 📝 Writing a Script for Cron

Always use **absolute paths** inside scripts scheduled with cron, as cron runs with a minimal environment and may not know your `PATH`.

```bash
#!/bin/bash

# Use absolute paths for commands and files
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
LOGFILE="/home/user/logs/backup.log"

echo "[$TIMESTAMP] Backup started" >> "$LOGFILE"
/usr/bin/rsync -av /home/user/data/ /mnt/backup/ >> "$LOGFILE" 2>&1
echo "[$TIMESTAMP] Backup complete" >> "$LOGFILE"
```

Make the script executable before scheduling it:

```bash
chmod +x /path/to/script.sh
```

---

## 🔢 Cron Field Value Reference

| Field | Allowed Values | Special Characters |
|-------|---------------|-------------------|
| Minute | 0 – 59 | `*` `,` `-` `*/n` |
| Hour | 0 – 23 | `*` `,` `-` `*/n` |
| Day of Month | 1 – 31 | `*` `,` `-` `*/n` |
| Month | 1 – 12 | `*` `,` `-` `*/n` |
| Day of Week | 0 – 7 (0 & 7 = Sun) | `*` `,` `-` `*/n` |

| Character | Meaning |
|-----------|---------|
| `*` | Any / every value |
| `,` | List of values — e.g. `1,3,5` |
| `-` | Range of values — e.g. `1-5` |
| `*/n` | Every nth value — e.g. `*/10` |

---

## 🚀 How to Set Up a Cron Job

```bash
# Step 1: Write your script
nano /home/user/myscript.sh

# Step 2: Make it executable
chmod +x /home/user/myscript.sh

# Step 3: Open the crontab editor
crontab -e

# Step 4: Add your cron job (runs every day at midnight)
0 0 * * * /home/user/myscript.sh >> /home/user/myscript.log 2>&1

# Step 5: Save and exit — cron picks it up automatically
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Script not running | Ensure the script has `chmod +x` and uses absolute paths |
| No output or errors visible | Redirect output: `>> /path/to/log 2>&1` |
| Wrong time zone | Check system timezone with `timedatectl` |
| Environment variables missing | Define `PATH` at the top of your crontab or script |
| Cron not installed | Install with `sudo apt install cron` and enable with `sudo systemctl enable cron` |

---

## 🔑 Key Takeaways

- Cron uses a **5-field time expression**: minute, hour, day-of-month, month, day-of-week.
- `*` means **every** — use it for fields you don't want to restrict.
- `*/n` means **every nth unit** — great for intervals like every 5 minutes (`*/5`).
- Always use **absolute paths** in cron jobs — cron's `$PATH` is minimal.
- Always **redirect output** (`>> logfile 2>&1`) to capture errors and debug issues.
- Use `crontab -l` to verify your jobs are registered correctly.

---

## 📚 Related Concepts

- [Crontab Guru — Visual Cron Expression Editor](https://crontab.guru)
- [GNU Cron Manual](https://www.gnu.org/software/mcron/manual/mcron.html)
- [Systemd Timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) — a modern alternative to cron

</details>
