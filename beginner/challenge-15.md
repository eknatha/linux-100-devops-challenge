# Challenge 15: View File Start and End in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

When working with large files — especially log files, CSV datasets, or configuration files — you rarely need to see the entire content at once. The `head` and `tail` commands let you quickly peek at the **beginning** or **end** of a file without loading it all into memory or scrolling through thousands of lines.

- **`head`** — shows the **first** N lines (default: 10)
- **`tail`** — shows the **last** N lines (default: 10)
- **`tail -f`** — **follows** a file live as new lines are added

---

## 🧩 Task

Display the first and last lines of a file.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# View first 10 lines (default)
head filename.txt

# View last 10 lines (default)
tail filename.txt

# View first N lines
head -n 20 filename.txt

# View last N lines
tail -n 20 filename.txt
```

### How it works

| Command | Description |
|---------|-------------|
| `head` | Display the **first** N lines of a file (default: 10) |
| `tail` | Display the **last** N lines of a file (default: 10) |
| `-n N` | Specify the **number of lines** to display |

---

## 🔢 Understanding the Output

```
File with 100 lines:
┌─────────────────────────────────┐
│ Line 1   ◄── head shows these  │
│ Line 2                          │
│ Line 3                          │
│  ...  (10 lines by default)    │
│ Line 10                         │
├─────────────────────────────────┤
│  ...  (middle lines)           │
├─────────────────────────────────┤
│ Line 91                         │
│ Line 92                         │
│  ...  (10 lines by default)    │
│ Line 100 ◄── tail shows these  │
└─────────────────────────────────┘
```

---

## 📖 Extended Examples

### Example 1 — View First 10 Lines (Default)

```bash
head /var/log/syslog
```

```bash
Mar 21 00:00:01 hostname systemd[1]: Started Daily Cleanup.
Mar 21 00:00:02 hostname cron[1024]: pam_unix(cron:session): session opened
Mar 21 00:05:01 hostname CRON[1456]: (root) CMD (command -v debian-sa1)
Mar 21 00:10:01 hostname CRON[1478]: (root) CMD (test -e /run/systemd/private)
...
(10 lines total)
```

---

### Example 2 — View Last 10 Lines (Default)

```bash
tail /var/log/syslog
```

```bash
Mar 21 10:30:01 hostname sshd[4821]: Accepted publickey for devuser
Mar 21 10:30:02 hostname systemd[1]: Started Session 12 of user devuser.
Mar 21 10:35:01 hostname myapp[5201]: INFO: Processing request
Mar 21 10:35:02 hostname myapp[5201]: ERROR: Database timeout
...
(last 10 lines)
```

---

### Example 3 — Specify Number of Lines

```bash
# View first 5 lines
head -n 5 filename.txt

# View last 20 lines
tail -n 20 filename.txt

# View first 1 line (check file type or header)
head -n 1 data.csv

# View last 1 line (check final status or record)
tail -n 1 data.csv
```

---

### Example 4 — View First and Last Together

```bash
# View first 5 lines
echo "=== First 5 lines ===" && head -n 5 filename.txt

# View last 5 lines
echo "=== Last 5 lines ===" && tail -n 5 filename.txt
```

```bash
=== First 5 lines ===
Line 1
Line 2
Line 3
Line 4
Line 5
=== Last 5 lines ===
Line 96
Line 97
Line 98
Line 99
Line 100
```

---

### Example 5 — Skip First N Lines with `tail +N`

```bash
# Show everything EXCEPT the first line (skip header row)
tail -n +2 data.csv

# Skip first 5 lines
tail -n +6 filename.txt
```

```bash
# data.csv:
Name,Age,City        ← header (skipped)
Alice,30,NYC
Bob,25,LA
Carol,35,Chicago

# tail -n +2 data.csv:
Alice,30,NYC
Bob,25,LA
Carol,35,Chicago
```

> 💡 `tail -n +N` means "start from line N" — perfect for skipping CSV headers in scripts.

---

### Example 6 — Show All Except Last N Lines with `head`

```bash
# Show all lines except the last 5
head -n -5 filename.txt
```

```bash
# If file has 20 lines, shows lines 1–15
```

> 💡 `head -n -N` (negative number) shows everything **except** the last N lines.

---

### Example 7 — View in Bytes Instead of Lines

```bash
# View first 100 bytes
head -c 100 filename.txt

# View last 500 bytes
tail -c 500 filename.txt

# View first 1KB of a file
head -c 1K filename.txt
```

---

### Example 8 — Follow a File Live (`tail -f`)

```bash
# Monitor a log file in real time
tail -f /var/log/syslog

# Monitor nginx error log live
tail -f /var/log/nginx/error.log

# Monitor multiple files at once
tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

```bash
# tail -f output streams new lines as they arrive:
Mar 21 10:35:01 hostname myapp[5201]: INFO: Request received
Mar 21 10:35:02 hostname myapp[5201]: INFO: Processing...
Mar 21 10:35:03 hostname myapp[5201]: ERROR: Timeout        ← appears live
```

> 💡 `tail -f` is one of the most used Linux commands for **live log monitoring**. Press `Ctrl+C` to stop.

---

### Example 9 — Follow by Filename (`tail -F`)

```bash
# Follow by filename — reconnects if file is rotated
tail -F /var/log/nginx/access.log
```

> 💡 `-F` (capital F) is better than `-f` for log files that get **rotated** — it keeps following even when the file is replaced with a new one.

---

### Example 10 — View Multiple Files

```bash
# View first 5 lines of multiple files
head -n 5 file1.txt file2.txt file3.txt
```

```bash
==> file1.txt <==
Line 1 of file1
Line 2 of file1
Line 3 of file1
Line 4 of file1
Line 5 of file1

==> file2.txt <==
Line 1 of file2
Line 2 of file2
...
```

> When viewing multiple files, `head` and `tail` automatically add `==> filename <==` headers.

---

### Example 11 — Combine with Pipes

```bash
# View first 10 processes sorted by CPU
ps aux --sort=-%cpu | head -n 11

# View last 20 lines of error log, filtered
tail -n 20 /var/log/syslog | grep -i "error"

# Check CSV header and first data row
head -n 2 data.csv

# View last entry in a sorted list
sort filename.txt | tail -n 1

# Get the 5th line of a file
head -n 5 filename.txt | tail -n 1
```

---

### Example 12 — Real-World: Monitor Deployment Logs

```bash
#!/bin/bash
LOG="/var/log/deployment.log"

echo "=== Deployment started ===" >> "$LOG"
./deploy.sh >> "$LOG" 2>&1
echo "=== Deployment finished ===" >> "$LOG"

echo "Last 20 lines of deployment log:"
tail -n 20 "$LOG"
```

---

## 🗂️ `head` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-n N` | Show first **N lines** (default: 10) |
| `-n -N` | Show all except **last N lines** |
| `-c N` | Show first **N bytes** |
| `-q` | **Quiet** — no filename headers when reading multiple files |
| `-v` | **Verbose** — always show filename headers |

---

## 🗂️ `tail` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-n N` | Show last **N lines** (default: 10) |
| `-n +N` | Show all lines **starting from line N** |
| `-c N` | Show last **N bytes** |
| `-f` | **Follow** — stream new lines as they are added |
| `-F` | **Follow by name** — reconnects if file is rotated |
| `-q` | **Quiet** — no filename headers |
| `-v` | **Verbose** — always show filename headers |

---

## 🔄 Common Patterns

| Goal | Command |
|------|---------|
| View file beginning | `head filename.txt` |
| View file end | `tail filename.txt` |
| Custom line count | `head -n 20` / `tail -n 20` |
| Skip CSV header | `tail -n +2 data.csv` |
| All except last N | `head -n -5 filename.txt` |
| Live log monitoring | `tail -f /var/log/app.log` |
| Get specific line | `head -n 5 file \| tail -n 1` |
| First process by CPU | `ps aux --sort=-%cpu \| head -2 \| tail -1` |

---

## 🚀 Full Workflow — Inspect a Large Log File

```bash
# Step 1: Check file size
wc -l /var/log/syslog

# Step 2: View the beginning (oldest entries)
head -n 20 /var/log/syslog

# Step 3: View the end (most recent entries)
tail -n 20 /var/log/syslog

# Step 4: Skip first line if it's a header
tail -n +2 /var/log/syslog | head -n 10

# Step 5: Monitor live as new entries appear
tail -f /var/log/syslog

# Step 6: Filter live output
tail -f /var/log/syslog | grep --color -i "error\|warn"
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Using `tail -f` on rotated log files | Use `tail -F` to follow by filename after rotation |
| `head -n -5` not working on older systems | Some old `head` versions don't support negative numbers |
| Forgetting `-n` gives 10 lines by default | Always specify `-n N` for a precise count |
| `tail -n +2` includes line 2 (1-based) | `+N` means start FROM line N — verify with a test file |

---

## 🔑 Key Takeaways

- `head` shows the **beginning** of a file; `tail` shows the **end** — both default to 10 lines.
- Always use `-n N` to specify an exact line count — default 10 may not be enough.
- `tail -n +N` skips the first N-1 lines — perfect for **removing CSV headers** in scripts.
- `head -n -N` shows everything **except** the last N lines.
- `tail -f` is the essential tool for **real-time log monitoring** — it streams new lines as they arrive.
- Use `tail -F` (capital F) for **log rotation awareness** — it reconnects when the log file is replaced.
- Combine `head` and `tail` with pipes for **extracting specific lines**: `head -n 10 file | tail -n 1` gets line 10.

---

## 📚 Related Concepts

- [GNU head Manual](https://www.gnu.org/software/coreutils/manual/html_node/head-invocation.html)
- [GNU tail Manual](https://www.gnu.org/software/coreutils/manual/html_node/tail-invocation.html)

</details>
