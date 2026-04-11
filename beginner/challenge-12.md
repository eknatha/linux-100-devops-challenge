# Challenge 12: Redirect Output in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**Output redirection** is one of the most powerful and fundamental concepts in Linux. By default, command output goes to the terminal screen. Redirection lets you send that output somewhere else — to a file, to another command, or to the void.

Understanding redirection unlocks the ability to:
- Save command output to files for later analysis
- Build log files for scripts
- Chain commands together using pipes
- Suppress unwanted output
- Separate normal output from error messages

---

## 🧩 Task

Write the output of a command to a file instead of displaying it on the terminal.

---



<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Write output to a file (overwrite)
echo "Hello, World!" > output.txt

# Append output to a file (keep existing content)
echo "Another line" >> output.txt
```

### How it works

| Operator | Description |
|----------|-------------|
| `>` | **Redirect** output to a file — **overwrites** existing content |
| `>>` | **Append** output to a file — adds to existing content |

---

## 🔢 Understanding Standard Streams

Every Linux process has three standard streams:

```
┌─────────────────────────────────────────┐
│              Linux Process              │
│                                         │
│  stdin  (0) ──► [input from keyboard]  │
│  stdout (1) ──► [normal output]         │
│  stderr (2) ──► [error messages]        │
└─────────────────────────────────────────┘
```

| Stream | Number | Default destination | Description |
|--------|--------|--------------------|-----------------------|
| `stdin` | `0` | Keyboard | Standard input |
| `stdout` | `1` | Terminal screen | Normal output |
| `stderr` | `2` | Terminal screen | Error messages |

---

## 📖 Extended Examples

### Example 1 — Write Output to a File (Overwrite)

```bash
echo "Hello, World!" > output.txt

# View the file
cat output.txt
```

```bash
Hello, World!
```

> ⚠️ `>` **overwrites** the file completely. If `output.txt` already existed, its content is gone.

---

### Example 2 — Append Output to a File

```bash
echo "First line" > output.txt
echo "Second line" >> output.txt
echo "Third line" >> output.txt

cat output.txt
```

```bash
First line
Second line
Third line
```

> 💡 `>>` **adds** to the file without destroying existing content — essential for log files.

---

### Example 3 — Save Command Output to a File

```bash
# Save directory listing to a file
ls -la > filelist.txt

# Save running processes to a file
ps aux > processes.txt

# Save disk usage to a file
df -h > disk_usage.txt

# Save network connections to a file
ss -tlnp > connections.txt
```

---

### Example 4 — Redirect Errors (stderr) to a File

```bash
# Redirect error output (stderr) to a file
ls /nonexistent 2> errors.txt

cat errors.txt
```

```bash
ls: cannot access '/nonexistent': No such file or directory
```

> `2>` redirects **stderr** (file descriptor 2) to a file instead of the terminal.

---

### Example 5 — Redirect Both stdout and stderr

```bash
# Redirect stdout to one file, stderr to another
command > output.txt 2> errors.txt

# Redirect both stdout and stderr to the same file
command > all_output.txt 2>&1

# Shorter syntax (bash 4+)
command &> all_output.txt
```

```bash
# Example: find command with errors
find / -name "*.conf" > found.txt 2>&1
# Normal results → found.txt
# "Permission denied" errors → also found.txt
```

---

### Example 6 — Suppress Output (Send to /dev/null)

```bash
# Suppress normal output
command > /dev/null

# Suppress error messages
command 2> /dev/null

# Suppress everything (both stdout and stderr)
command > /dev/null 2>&1
```

```bash
# Example: run a script silently
./backup.sh > /dev/null 2>&1

# Example: check if a command exists without any output
which nginx > /dev/null 2>&1 && echo "nginx is installed"
```

> 💡 `/dev/null` is a special device that **discards everything** written to it — the "black hole" of Linux.

---

### Example 7 — Pipe Output to Another Command

```bash
# | (pipe) sends stdout of one command to stdin of another
ls -la | grep ".txt"

# Count lines in output
ps aux | wc -l

# Sort and save
ls -la | sort -k5 -rn > sorted_by_size.txt
```

---

### Example 8 — Redirect Input from a File (stdin)

```bash
# Read input from a file instead of keyboard
sort < unsorted.txt

# Use file as input and save output
sort < unsorted.txt > sorted.txt

# Feed a file into a command
mysql -u root -p database_name < backup.sql
```

---

### Example 9 — Here Document (Multi-line Input)

```bash
# Write multiple lines to a file using heredoc
cat > config.txt << EOF
hostname=myserver
port=8080
debug=false
EOF

cat config.txt
```

```bash
hostname=myserver
port=8080
debug=false
```

> 💡 `<< EOF` is a **heredoc** — it feeds multiple lines as input until it sees `EOF` on a line by itself.

---

### Example 10 — Append Command Output to a Log File

```bash
#!/bin/bash
LOGFILE="/var/log/myapp.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Append timestamped entries to log
echo "[$DATE] Backup started" >> "$LOGFILE"
tar -czf /backup/data.tar.gz /data/ >> "$LOGFILE" 2>&1
echo "[$DATE] Backup completed" >> "$LOGFILE"
```

```bash
# Log file output:
[2025-03-21 10:00:01] Backup started
[2025-03-21 10:00:45] Backup completed
```

---

### Example 11 — tee — Output to Screen AND File Simultaneously

```bash
# tee writes to both stdout (screen) and a file at the same time
ls -la | tee filelist.txt

# Append mode with tee
command | tee -a logfile.txt
```

```bash
# tee output — shows on screen AND saves to file:
total 48
drwxr-xr-x 5 devuser devuser 4096 Mar 21 10:00 .
-rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 file1.txt
```

> 💡 `tee` is perfect when you want to **see the output AND save it** at the same time.

---

## 🗂️ Redirection Operators — Quick Reference

| Operator | Description |
|----------|-------------|
| `>` | Redirect stdout to file — **overwrites** |
| `>>` | Redirect stdout to file — **appends** |
| `2>` | Redirect stderr to file — overwrites |
| `2>>` | Redirect stderr to file — appends |
| `2>&1` | Redirect stderr to the **same place as stdout** |
| `&>` | Redirect **both** stdout and stderr to file |
| `<` | Redirect **stdin** from a file |
| `<<EOF` | **Heredoc** — multi-line stdin |
| `|` | **Pipe** — send stdout to next command's stdin |
| `>/dev/null` | **Discard** stdout |
| `2>/dev/null` | **Discard** stderr |

---

## 🔄 Common Redirection Patterns

| Goal | Command |
|------|---------|
| Save output to file | `command > file.txt` |
| Add output to file | `command >> file.txt` |
| Save errors to file | `command 2> errors.txt` |
| Save all output + errors | `command > file.txt 2>&1` |
| Discard all output | `command > /dev/null 2>&1` |
| Show output + save to file | `command \| tee file.txt` |
| Read input from file | `command < input.txt` |

---

## 🚀 Full Workflow — Build a Script with Logging

```bash
#!/bin/bash

LOG="/var/log/deployment.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Create or clear the log file
echo "=== Deployment Log ===" > "$LOG"
echo "Started: $DATE" >> "$LOG"

# Run commands and capture output
echo "Pulling latest code..." | tee -a "$LOG"
git pull origin main >> "$LOG" 2>&1

echo "Installing dependencies..." | tee -a "$LOG"
npm install >> "$LOG" 2>&1

echo "Restarting service..." | tee -a "$LOG"
sudo systemctl restart myapp >> "$LOG" 2>&1

echo "Done: $(date)" >> "$LOG"
echo "Deployment complete. See $LOG for details."
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Using `>` instead of `>>` erases the file | Always use `>>` for log files |
| Forgetting `2>&1` — errors not captured | Use `command > file.txt 2>&1` to capture everything |
| `> file.txt` truncates file even if command fails | Use `>>` or check exit code before redirecting |
| Piping stderr through pipe | Use `2>&1 \|` to pipe both streams |
| `tee` not available | Install with `sudo apt install coreutils` |

---

## 🔑 Key Takeaways

- `>` **overwrites** a file; `>>` **appends** to it — choosing the wrong one can destroy data.
- **stdout** is normal output (descriptor 1); **stderr** is error output (descriptor 2).
- `2>&1` merges stderr into stdout — use it to capture everything in one file.
- `/dev/null` is the discard device — send unwanted output there to suppress it.
- `tee` lets you **see output on screen AND save it** to a file simultaneously.
- **Pipes (`|`)** chain commands — the stdout of one becomes the stdin of the next.
- Use `>>` in scripts to **build log files** with timestamped entries.

---

## 📚 Related Concepts

- [GNU Bash Redirection](https://www.gnu.org/software/bash/manual/bash.html#Redirections)
- [tee Manual](https://linux.die.net/man/1/tee)

</details>
