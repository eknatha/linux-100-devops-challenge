# Challenge 67: Monitor Authentication Logs in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Authentication logs** record every login attempt, privilege escalation, sudo usage, and authentication event on a Linux system. Regularly monitoring these logs is essential for:

- Detecting **brute-force attacks** and unauthorized access attempts
- Auditing **who logged in, when, and from where**
- Investigating **security incidents** after the fact
- Monitoring **sudo and privilege escalation** activity

The log location varies by distribution — `/var/log/secure` on RHEL-based systems and `/var/log/auth.log` on Debian/Ubuntu.

---

## 🧩 Task

View authentication and login attempt logs to monitor system access.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
cat /var/log/secure
```

### How it works

| Part | Description |
|------|-------------|
| `cat` | Display the full contents of a file |
| `/var/log/secure` | Authentication log on **RHEL / CentOS / Fedora** |

> 💡 On **Debian / Ubuntu**, the equivalent file is `/var/log/auth.log`:
> ```bash
> cat /var/log/auth.log
> ```

---

## 🗂️ Auth Log Location by Distribution

| Distribution | Auth Log File |
|-------------|--------------|
| RHEL / CentOS / Fedora / Rocky | `/var/log/secure` |
| Debian / Ubuntu / Mint | `/var/log/auth.log` |
| Arch Linux | `journalctl` (systemd journal only) |
| openSUSE | `/var/log/messages` |
| Any systemd distro | `journalctl -u sshd` or `journalctl _COMM=sudo` |

---

## 📖 Extended Examples

### Example 1 — View Full Auth Log

```bash
# RHEL/CentOS/Fedora
cat /var/log/secure

# Debian/Ubuntu
cat /var/log/auth.log
```

```bash
Mar 21 10:00:01 hostname sshd[1024]: Accepted password for devuser from 192.168.1.5 port 54312 ssh2
Mar 21 10:01:15 hostname sudo: devuser : TTY=pts/0 ; PWD=/home/devuser ; USER=root ; COMMAND=/bin/bash
Mar 21 10:05:32 hostname sshd[1031]: Failed password for root from 203.0.113.10 port 44213 ssh2
Mar 21 10:05:33 hostname sshd[1031]: Failed password for root from 203.0.113.10 port 44214 ssh2
```

---

### Example 2 — Follow Auth Log in Real Time

```bash
# Watch new login attempts as they happen
tail -f /var/log/secure          # RHEL/CentOS
tail -f /var/log/auth.log        # Debian/Ubuntu
```

> 💡 `-f` streams new log entries live — perfect for monitoring a server while a login is in progress or an attack is ongoing.

---

### Example 3 — Show Only the Last N Lines

```bash
# Last 50 lines
tail -n 50 /var/log/secure

# Last 100 lines
tail -n 100 /var/log/auth.log
```

---

### Example 4 — Filter Failed Login Attempts

```bash
# RHEL/CentOS
grep "Failed password" /var/log/secure

# Debian/Ubuntu
grep "Failed password" /var/log/auth.log
```

```bash
Mar 21 10:05:32 hostname sshd[1031]: Failed password for root from 203.0.113.10 port 44213 ssh2
Mar 21 10:05:35 hostname sshd[1032]: Failed password for root from 203.0.113.10 port 44217 ssh2
Mar 21 10:05:38 hostname sshd[1033]: Failed password for invalid user admin from 203.0.113.10 port 44220 ssh2
```

---

### Example 5 — Filter Successful Logins

```bash
grep "Accepted" /var/log/secure
```

```bash
Mar 21 10:00:01 hostname sshd[1024]: Accepted password for devuser from 192.168.1.5 port 54312 ssh2
Mar 21 10:30:10 hostname sshd[1101]: Accepted publickey for adminuser from 10.0.0.5 port 52100 ssh2
```

---

### Example 6 — Find All Failed Login Attempts with IP Addresses

```bash
grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```

```bash
 127  203.0.113.10
  43  198.51.100.5
  18  203.0.113.88
   5  192.168.1.99
```

> This one-liner shows which IP addresses have the most failed login attempts — quickly identifies brute-force attackers.

| Part | Description |
|------|-------------|
| `grep "Failed password"` | Filter only failed attempts |
| `awk '{print $(NF-3)}'` | Extract the IP address field (3rd from last) |
| `sort` | Sort IPs for grouping |
| `uniq -c` | Count occurrences of each IP |
| `sort -rn` | Sort by count, highest first |

---

### Example 7 — Check for Invalid User Attempts

```bash
grep "Invalid user" /var/log/secure
```

```bash
Mar 21 10:05:38 hostname sshd[1033]: Invalid user admin from 203.0.113.10 port 44220
Mar 21 10:05:40 hostname sshd[1034]: Invalid user oracle from 203.0.113.10 port 44225
Mar 21 10:05:42 hostname sshd[1035]: Invalid user ubuntu from 203.0.113.10 port 44230
```

> Repeated attempts with common usernames (`admin`, `root`, `ubuntu`, `oracle`) are classic signs of a **credential stuffing** or **dictionary attack**.

---

### Example 8 — Monitor Sudo Usage

```bash
grep "sudo" /var/log/secure
# or
grep "sudo" /var/log/auth.log
```

```bash
Mar 21 10:01:15 hostname sudo: devuser : TTY=pts/0 ; PWD=/home/devuser ; USER=root ; COMMAND=/bin/bash
Mar 21 10:15:42 hostname sudo: devuser : TTY=pts/0 ; PWD=/home/devuser ; USER=root ; COMMAND=/usr/bin/apt update
Mar 21 10:20:01 hostname sudo: baduser : 3 incorrect password attempts ; TTY=pts/1 ; COMMAND=/bin/su
```

---

### Example 9 — Check Authentication Log with `journalctl` (systemd)

```bash
# All authentication events (current boot)
journalctl -b -u sshd

# Failed SSH logins
journalctl -b -u sshd | grep "Failed"

# Successful SSH logins
journalctl -b -u sshd | grep "Accepted"

# sudo activity
journalctl _COMM=sudo

# Authentication events for a specific user
journalctl _COMM=sshd | grep "devuser"

# Follow live SSH logs
journalctl -u sshd -f
```

---

### Example 10 — Count Login Attempts by User

```bash
grep "Failed password" /var/log/secure | awk '{print $9}' | sort | uniq -c | sort -rn | head -10
```

```bash
 127  root
  43  admin
  18  ubuntu
   5  postgres
   3  devuser
```

> Shows which usernames attackers are targeting most — `root` and `admin` are almost always at the top of brute-force lists.

---

### Example 11 — Find Logins Within a Specific Time Range

```bash
# Using awk to filter by date
awk '/Mar 21 10/ {print}' /var/log/secure

# Using grep for a time window
grep "Mar 21 10:0[0-5]" /var/log/secure

# Using journalctl for precise time ranges
journalctl -u sshd --since "2025-03-21 10:00:00" --until "2025-03-21 10:30:00"
```

---

### Example 12 — Check Last Logins with `last` and `lastb`

```bash
# Show recent successful logins
last

# Show recent failed login attempts (requires root)
sudo lastb

# Show last login for a specific user
last devuser

# Show currently logged-in users
who
w
```

```bash
# last output:
devuser  pts/0        192.168.1.5      Thu Mar 21 10:00   still logged in
adminuser pts/1       10.0.0.5         Thu Mar 21 09:45 - 10:30  (00:45)
root     tty1                          Thu Mar 20 22:10 - 22:15  (00:05)

# lastb output (failed attempts):
root     ssh:notty    203.0.113.10     Thu Mar 21 10:05 - 10:05  (00:00)
root     ssh:notty    203.0.113.10     Thu Mar 21 10:05 - 10:05  (00:00)
admin    ssh:notty    203.0.113.10     Thu Mar 21 10:05 - 10:05  (00:00)
```

---

### Example 13 — Full Auth Monitoring Script

```bash
#!/bin/bash

LOG="/var/log/secure"          # Change to /var/log/auth.log on Debian/Ubuntu
THRESHOLD=10                   # Alert if an IP has more than 10 failed attempts
REPORT="/tmp/auth_report_$(date +"%Y-%m-%d").txt"

echo "============================================" > "$REPORT"
echo " Auth Security Report — $(date)"             >> "$REPORT"
echo "============================================" >> "$REPORT"

echo -e "\n[Failed Login Attempts by IP]"          >> "$REPORT"
grep "Failed password" "$LOG" \
  | awk '{print $(NF-3)}' \
  | sort | uniq -c | sort -rn \
  | awk -v t="$THRESHOLD" '$1 >= t {print $1 " attempts — " $2}' \
  >> "$REPORT"

echo -e "\n[Invalid Usernames Tried]"              >> "$REPORT"
grep "Invalid user" "$LOG" \
  | awk '{print $8}' \
  | sort | uniq -c | sort -rn | head -10           >> "$REPORT"

echo -e "\n[Successful Logins]"                    >> "$REPORT"
grep "Accepted" "$LOG" \
  | awk '{print $1" "$2" "$3" — user:"$9" from:"$11}' \
  >> "$REPORT"

echo -e "\n[Sudo Activity]"                        >> "$REPORT"
grep "sudo" "$LOG"                                 >> "$REPORT"

cat "$REPORT"
echo -e "\nReport saved to: $REPORT"
```

---

## 🔍 Common Log Patterns — Quick Reference

| Pattern | `grep` Filter | Meaning |
|---------|--------------|---------|
| Failed login | `"Failed password"` | Wrong password used |
| Invalid user | `"Invalid user"` | Username doesn't exist on system |
| Accepted login | `"Accepted password"` | Successful password login |
| Key login | `"Accepted publickey"` | Successful key-based login |
| Sudo command | `"sudo"` | Privilege escalation event |
| Session opened | `"session opened"` | Login session started |
| Session closed | `"session closed"` | Login session ended |
| Disconnected | `"Disconnected"` | Connection closed |
| Max auth exceeded | `"maximum authentication"` | Too many failed attempts |

---

## 🛠️ Useful Auth Monitoring Commands

| Command | Description |
|---------|-------------|
| `last` | Recent successful logins |
| `lastb` | Recent failed login attempts |
| `lastlog` | Last login per user |
| `who` | Currently logged-in users |
| `w` | Logged-in users with activity |
| `id username` | User UID, GID, and groups |
| `finger username` | User info (if installed) |
| `faillock --user username` | Show failed auth count (RHEL) |
| `pam_tally2 --user username` | Show failed auth count (older systems) |

---

## 🚀 Full Workflow — Investigate a Suspected Brute-Force Attack

```bash
# Step 1: Check for high volumes of failed attempts
grep "Failed password" /var/log/secure | wc -l

# Step 2: Identify the attacking IP(s)
grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -5

# Step 3: See what usernames were targeted
grep "Failed password" /var/log/secure | awk '{print $9}' | sort | uniq -c | sort -rn | head -10

# Step 4: Check if any attempts succeeded
grep "Accepted" /var/log/secure

# Step 5: See current active sessions
who
w

# Step 6: Block the attacker's IP with firewall
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="203.0.113.10" reject' --permanent
sudo firewall-cmd --reload

# Step 7: (Optional) Install fail2ban to automate IP blocking
sudo dnf install fail2ban
sudo systemctl enable --now fail2ban
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Wrong log file path | Use `/var/log/secure` on RHEL, `/var/log/auth.log` on Debian/Ubuntu |
| `lastb` shows nothing | Needs root: `sudo lastb` — also requires `/var/log/btmp` to exist |
| Logs rotated and missing older entries | Check rotated files: `zcat /var/log/secure.1` or `/var/log/secure-*.gz` |
| `journalctl` preferred over log files | On pure systemd distros (Arch), use `journalctl -u sshd` instead of flat files |
| High number of failed attempts but no block | Install `fail2ban` for automatic IP banning after threshold is exceeded |

---

## 🔑 Key Takeaways

- `/var/log/secure` (RHEL) and `/var/log/auth.log` (Debian/Ubuntu) are the primary authentication log files.
- `grep "Failed password"` + `awk` + `uniq -c` is the fastest way to identify brute-force attackers.
- `last` shows successful logins; `sudo lastb` shows failed attempts — use both together.
- Use `journalctl -u sshd` on modern systemd systems as a more powerful alternative.
- Combine auth log monitoring with **firewalld rich rules** or **fail2ban** to automatically block attackers.
- Check for **`Invalid user`** entries — they reveal which usernames attackers are guessing.

---

## 📚 Related Concepts

- [fail2ban](https://www.fail2ban.org/) — automatically ban IPs with repeated failed auth attempts
- [auditd](https://linux.die.net/man/8/auditd) — advanced Linux audit framework for detailed security logging

</details>
