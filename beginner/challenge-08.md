# Challenge 08: Check Logged-In Users in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Knowing **who is currently logged into a Linux system** is a fundamental administration task. It helps you:

- Monitor who is actively using the server
- Identify unexpected or unauthorized sessions
- See what each user is currently doing
- Check how long users have been idle
- Verify your own active sessions before making changes

Linux provides several simple commands — `who`, `w`, and `users` — each offering a different level of detail about active login sessions.

---

## 🧩 Task

Check which users are currently logged into the system and what they are doing.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Simple list of logged-in users
who

# Detailed view with activity information
w

# Just the usernames
users
```

### How it works

| Command | Description |
|---------|-------------|
| `who` | Lists all users currently logged in, their terminal, and login time |
| `w` | Shows logged-in users plus what command they are currently running and idle time |
| `users` | Prints only the usernames of currently logged-in users |

---

## 🔢 Understanding `who` Output

```bash
who
```

```
devuser   pts/0        2025-03-21 10:00 (192.168.1.5)
adminuser pts/1        2025-03-21 09:45 (10.0.0.5)
root      tty1         2025-03-21 08:00
```

| Column | Description |
|--------|-------------|
| `devuser` | Username of the logged-in user |
| `pts/0` | Terminal — `pts` = SSH/remote session, `tty` = local console |
| `2025-03-21 10:00` | Login date and time |
| `(192.168.1.5)` | IP address the user connected from |

---

## 🔢 Understanding `w` Output

```bash
w
```

```
 10:35:22 up 3 days,  1:35,  2 users,  load average: 0.45, 0.42, 0.38
USER     TTY      FROM             LOGIN@   IDLE WHAT
devuser  pts/0    192.168.1.5      10:00    0.00s w
adminuser pts/1   10.0.0.5         09:45    5:10  bash
```

| Column | Description |
|--------|-------------|
| `USER` | Username |
| `TTY` | Terminal (`pts/N` = remote, `tty` = local) |
| `FROM` | Source IP address |
| `LOGIN@` | Login time |
| `IDLE` | How long since the user last ran a command |
| `WHAT` | The command currently being executed |

> 💡 `w` is more useful than `who` for active monitoring — the `IDLE` and `WHAT` columns show whether users are actively working or have walked away.

---

## 📖 Extended Examples

### Example 1 — Simple User List with `who`

```bash
who
```

```bash
devuser   pts/0   2025-03-21 10:00 (192.168.1.5)
adminuser pts/1   2025-03-21 09:45 (10.0.0.5)
```

---

### Example 2 — Detailed View with `w`

```bash
w
```

```bash
 10:35:22 up 3 days,  1:35,  2 users,  load average: 0.45, 0.42, 0.38
USER     TTY      FROM             LOGIN@   IDLE WHAT
devuser  pts/0    192.168.1.5      10:00    0.00s w
adminuser pts/1   10.0.0.5         09:45    5:10  vim /etc/nginx/nginx.conf
```

---

### Example 3 — Just the Usernames with `users`

```bash
users
```

```bash
adminuser devuser
```

> `users` is the simplest output — a single space-separated line of usernames.

---

### Example 4 — Who Am I?

```bash
# Who is the current user?
whoami

# Full details about the current session
who am i
```

```bash
# whoami output:
devuser

# who am i output:
devuser  pts/0  2025-03-21 10:00 (192.168.1.5)
```

---

### Example 5 — Count Currently Logged-In Users

```bash
# Count total logged-in users
who | wc -l

# Or using w (subtract 2 for header lines)
w | tail -n +3 | wc -l
```

```bash
2
```

---

### Example 6 — Show Login History with `last`

```bash
# Recent login history (successful logins)
last

# Last 10 logins
last -n 10

# Login history for a specific user
last devuser
```

```bash
devuser  pts/0  192.168.1.5  Thu Mar 21 10:00   still logged in
adminuser pts/1 10.0.0.5     Thu Mar 21 09:45 - 10:30  (00:45)
root     tty1               Thu Mar 20 22:10 - 22:15  (00:05)

wtmp begins Mon Mar 18 08:00:00 2025
```

---

### Example 7 — Show Failed Login Attempts with `lastb`

```bash
sudo lastb | head -10
```

```bash
root  ssh:notty  203.0.113.99  Thu Mar 21 03:10 - 03:10  (00:00)
admin ssh:notty  203.0.113.99  Thu Mar 21 03:11 - 03:11  (00:00)
```

> `lastb` reads `/var/log/btmp` — the record of failed login attempts. A high volume from one IP may indicate a brute-force attack.

---

### Example 8 — Check Last Login Per User with `lastlog`

```bash
lastlog
```

```bash
Username         Port     From               Latest
root             tty1                        Thu Mar 20 22:10:05 2025
devuser          pts/0    192.168.1.5        Thu Mar 21 10:00:12 2025
adminuser        pts/1    10.0.0.5           Thu Mar 21 09:45:00 2025
mysql                                        **Never logged in**
nobody                                       **Never logged in**
```

> ⚠️ Service accounts like `mysql` or `www-data` should show **Never logged in**. If they appear with a login time, investigate immediately.

---

### Example 9 — See User Activity in Real Time with `watch`

```bash
# Refresh who output every 2 seconds
watch -n 2 who

# Refresh w output every 5 seconds
watch -n 5 w
```

> `watch` reruns the command at a regular interval and updates the display — useful for monitoring login activity continuously.

---

### Example 10 — Find Users by Terminal or IP

```bash
# Find sessions from a specific IP
who | grep "192.168.1.5"

# Find all SSH sessions (pts terminals)
who | grep "pts"

# Find all local console sessions (tty)
who | grep "tty"

# Show only the IP addresses of connected users
who | awk '{print $NF}' | tr -d '()'
```

```bash
192.168.1.5
10.0.0.5
```

---

## 🗂️ Commands Comparison

| Command | Shows | Use Case |
|---------|-------|----------|
| `users` | Usernames only | Quick check of who is on |
| `who` | User, terminal, login time, IP | Standard session overview |
| `w` | All of `who` + idle time + current command | Active monitoring |
| `whoami` | Current user only | Confirm your own identity |
| `who am i` | Your own session details | Check your own session info |
| `last` | Login history (successful) | Audit past logins |
| `lastb` | Login history (failed) | Security audit |
| `lastlog` | Last login per user | Per-user login audit |

---

## 🗂️ Terminal Types Reference

| Terminal | Meaning |
|----------|---------|
| `pts/0`, `pts/1`, ... | **Pseudo terminal** — SSH or remote session |
| `tty1`, `tty2`, ... | **Physical terminal** — local console login |
| `:0`, `:1` | **Graphical session** — desktop/GUI login |

---

## 🚀 Full Workflow — Monitor Active Sessions

```bash
# Step 1: Quick check of who is logged in
who

# Step 2: See what they are doing and idle times
w

# Step 3: Count active sessions
who | wc -l

# Step 4: Check for sessions from unexpected IPs
who | grep -v "192.168\|10.0.0"

# Step 5: Review recent login history
last -n 20

# Step 6: Check for failed login attempts
sudo lastb | head -20

# Step 7: Verify service accounts have never logged in
lastlog | grep -v "Never logged in" | grep -E "mysql|www-data|nobody"

# Step 8: Monitor continuously
watch -n 5 w
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `who` shows no output | No users are logged in — normal for headless servers with no active sessions |
| Confusing `pts` and `tty` | `pts` = SSH/remote; `tty` = local physical console |
| `lastb` requires root | Use `sudo lastb` |
| `lastlog` shows service accounts | Service accounts should never log in — investigate if they show a login time |
| `w` IDLE shows `0.00s` for your own session | That is normal — the `w` command itself resets your idle timer |

---

## 🔑 Key Takeaways

- `who` shows **who is logged in** — terminal, login time, and source IP.
- `w` adds **idle time and current command** — the most informative tool for active monitoring.
- `users` gives the **simplest output** — just a list of usernames.
- `last` shows **login history**; `lastb` shows **failed attempts** — both are essential for security auditing.
- `lastlog` shows the **last login per user** — service accounts (`mysql`, `www-data`) should always show **Never logged in**.
- Combine with `grep` and `watch` for filtered real-time monitoring.

---

## 📚 Related Concepts

- [GNU who Manual](https://www.gnu.org/software/coreutils/manual/html_node/who-invocation.html)
- [last Manual](https://linux.die.net/man/1/last)

</details>
