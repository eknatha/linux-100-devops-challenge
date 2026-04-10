# Challenge 07: Search Text in Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Searching for text** inside files is one of the most essential Linux skills. Whether you are looking for an error in a log file, finding a configuration setting, or locating which file contains a specific function — `grep` (**G**lobal **R**egular **E**xpression **P**rint) is the go-to tool.

`grep` scans files line by line and prints every line that matches a given pattern. It supports simple word searches, case-insensitive matching, regular expressions, recursive directory searches, and much more.

---

## 🧩 Task

Search for a specific word or pattern inside a file.

---



<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
grep "word" filename.txt
```

### How it works

| Part | Description |
|------|-------------|
| `grep` | Search tool — scans each line for a matching pattern |
| `"word"` | The pattern to search for (quoted to handle spaces and special characters) |
| `filename.txt` | The file to search in |

---

## 🔢 Understanding `grep` Output

```bash
grep "error" /var/log/syslog
```

```
Mar 21 10:35:01 hostname myapp[4821]: ERROR: Cannot connect to database
Mar 21 10:35:02 hostname myapp[4821]: ERROR: Retry attempt 1 failed
Mar 21 10:35:03 hostname kernel: EXT4-fs error (device sda1): read failed
```

Each line printed is a **full line from the file** that contains the matching pattern. By default:
- Matching is **case-sensitive** (`error` ≠ `ERROR`)
- The entire matching line is printed, not just the matched word
- Line numbers are not shown (use `-n` to add them)

---

## 📖 Extended Examples

### Example 1 — Basic Word Search

```bash
grep "error" server.log
```

```bash
2025-03-21 10:35:01 ERROR: Cannot connect to database
2025-03-21 10:35:02 ERROR: Retry attempt failed
```

---

### Example 2 — Case-Insensitive Search

```bash
grep -i "error" server.log
```

```bash
# Matches: error, Error, ERROR, ErRoR — all variations
2025-03-21 10:35:01 ERROR: Cannot connect to database
2025-03-21 10:35:02 error: config file not found
2025-03-21 10:35:03 Error: Connection timed out
```

> 💡 `-i` is one of the most commonly used flags — always use it when you are not sure about capitalization.

---

### Example 3 — Show Line Numbers

```bash
grep -n "error" server.log
```

```bash
12:2025-03-21 10:35:01 ERROR: Cannot connect to database
15:2025-03-21 10:35:02 ERROR: Retry attempt failed
23:2025-03-21 10:35:03 ERROR: Timeout reached
```

> Line numbers make it easy to jump directly to the matching line in a text editor (`vim +12 server.log`).

---

### Example 4 — Count Matching Lines

```bash
grep -c "error" server.log
```

```bash
47
```

> `-c` returns only the **count** of matching lines — useful for quickly assessing how many errors exist.

---

### Example 5 — Show Lines That Do NOT Match (Invert)

```bash
# Show all lines that do NOT contain "debug"
grep -v "debug" server.log

# Combine: show errors but exclude warnings
grep -i "error" server.log | grep -v "warning"
```

---

### Example 6 — Show Context Around Matches

```bash
# 3 lines before and after each match
grep -C 3 "ERROR" server.log

# 5 lines after each match
grep -A 5 "ERROR" server.log

# 2 lines before each match
grep -B 2 "ERROR" server.log
```

```bash
# grep -C 3 output:
2025-03-21 10:34:58 INFO: Connecting to database...       ← 3 before
2025-03-21 10:34:59 INFO: Connection attempt 1
2025-03-21 10:35:00 INFO: Retry logic activated
2025-03-21 10:35:01 ERROR: Cannot connect to database     ← match
2025-03-21 10:35:02 INFO: Shutting down connection pool   ← 3 after
2025-03-21 10:35:03 INFO: Cleanup complete
2025-03-21 10:35:04 INFO: Exiting
```

> Context lines reveal what **led up to** and **followed** the error — essential for root cause analysis.

---

### Example 7 — Search Multiple Files

```bash
# Search in all .log files
grep "error" *.log

# Search in specific multiple files
grep "error" app.log access.log system.log

# Show which file each match came from (-l = files only)
grep -l "error" *.log
```

```bash
# grep "error" *.log output (shows filename):
app.log:2025-03-21 10:35:01 ERROR: Cannot connect
access.log:2025-03-21 10:36:12 error: 500 Internal Server Error
system.log:2025-03-21 10:37:01 error: disk full
```

---

### Example 8 — Recursive Search in Directories

```bash
# Search through all files in a directory tree
grep -r "TODO" /home/devuser/project/

# Recursive, case-insensitive, with line numbers
grep -rni "password" /etc/

# Recursive — show only filenames that match
grep -rl "api_key" /var/www/
```

```bash
# grep -r output:
/home/devuser/project/src/main.py:45:    # TODO: add error handling
/home/devuser/project/tests/test_api.py:12:    # TODO: mock database
```

---

### Example 9 — Search Using Regular Expressions

```bash
# Lines starting with "ERROR"
grep "^ERROR" server.log

# Lines ending with "failed"
grep "failed$" server.log

# Match any digit sequence
grep "[0-9]\+" server.log

# Extended regex — match "error" or "fail"
grep -E "error|fail" server.log

# Match IP address pattern
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log
```

| Regex Pattern | Matches |
|--------------|---------|
| `^word` | Lines **starting** with `word` |
| `word$` | Lines **ending** with `word` |
| `.` | Any **single character** |
| `*` | Zero or more of the preceding |
| `+` | One or more (use with `-E`) |
| `[abc]` | Any one of: a, b, or c |
| `[0-9]` | Any digit |
| `\|` | OR (use with `-E` as just `\|`) |

---

### Example 10 — Search for Whole Words Only

```bash
# Match only the whole word "log" — not "login" or "catalog"
grep -w "log" filename.txt
```

```bash
# Without -w:
grep "log" filename.txt
# Matches: log, login, catalog, dialog, backlog

# With -w:
grep -w "log" filename.txt
# Matches: log only
```

---

### Example 11 — Highlight Matches in Color

```bash
# Highlight the matching text in color
grep --color "error" server.log

# Most systems have this aliased already
grep --color=auto "error" server.log
```

> Many Linux systems alias `grep` to `grep --color=auto` by default in `~/.bashrc`.

---

### Example 12 — Combine with Pipes for Advanced Filtering

```bash
# Find processes containing "python"
ps aux | grep python

# Filter without showing grep itself
ps aux | grep "[p]ython"

# Find open ports on 80 or 443
ss -tlnp | grep -E ":80|:443"

# Count HTTP 404 errors in nginx log
grep " 404 " /var/log/nginx/access.log | wc -l

# Find the top 5 most common errors
grep -oE "ERROR: [^$]+" server.log | sort | uniq -c | sort -rn | head -5
```

---

## 🗂️ `grep` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-i` | **Case-insensitive** matching |
| `-n` | Show **line numbers** |
| `-c` | **Count** matching lines |
| `-v` | **Invert** — show lines that do NOT match |
| `-l` | Show only **filenames** with matches |
| `-L` | Show only **filenames** without matches |
| `-r` / `-R` | **Recursive** directory search |
| `-w` | Match **whole words** only |
| `-x` | Match **whole lines** only |
| `-E` | **Extended** regex (`+`, `?`, `\|`, `{}`) |
| `-P` | **Perl** regex (advanced patterns) |
| `-F` | **Fixed string** — no regex, literal match |
| `-o` | Print **only** the matching part |
| `-A N` | Show N lines **after** match |
| `-B N` | Show N lines **before** match |
| `-C N` | Show N lines **around** match |
| `--color` | **Highlight** matched text |
| `-m N` | Stop after **N** matches |
| `-q` | **Quiet** — exit 0 if match found, no output |

---

## 🔄 `grep` Variants

| Command | Description |
|---------|-------------|
| `grep` | Standard — basic and extended regex |
| `egrep` | Same as `grep -E` — extended regex |
| `fgrep` | Same as `grep -F` — literal strings (fastest) |
| `zgrep` | Search inside **compressed** (`.gz`) files |
| `rgrep` | Recursive grep (same as `grep -r`) |

---

## 🚀 Full Workflow — Search Logs for a Root Cause

```bash
# Step 1: Quick check — does the error exist?
grep -c "ERROR" /var/log/app.log

# Step 2: Find all error lines with line numbers
grep -in "error" /var/log/app.log

# Step 3: See context around the first error
grep -n "ERROR" /var/log/app.log | head -1
grep -C 5 "ERROR" /var/log/app.log | head -20

# Step 4: Count errors by type
grep -oE "ERROR: [^\n]+" /var/log/app.log | sort | uniq -c | sort -rn

# Step 5: Search across multiple log files
grep -rni "database" /var/log/

# Step 6: Find which files contain the pattern
grep -rl "connection refused" /var/log/

# Step 7: Filter to a time window
grep "Mar 21 10:3" /var/log/app.log | grep -i error

# Step 8: Follow live and filter
tail -f /var/log/app.log | grep --color -i "error\|warn\|fail"
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Case mismatch — missing results | Add `-i` for case-insensitive matching |
| `grep "."` matches everything | `.` is a regex wildcard — use `grep -F "."` for a literal dot |
| `grep` returns exit code 1 with no output | Exit code 1 means no match found — this is normal, not an error |
| Slow search on huge files | Use `grep -F` for literal strings (no regex overhead) |
| `grep pattern *` on empty directory | No files to match — check directory contents first |
| `ps aux \| grep process` shows grep itself | Use `grep "[p]rocess"` — the bracket prevents self-matching |

---

## 🔑 Key Takeaways

- `grep "word" file` is the simplest form — searches for a literal word in a file.
- Always add **`-i`** when unsure about capitalization — `grep -i "error"` matches all cases.
- **`-n`** adds line numbers — useful for jumping directly to the match in an editor.
- **`-C N`** shows context lines around matches — essential for understanding what caused an error.
- **`-r`** searches entire directory trees — use `grep -rni "pattern" /path/` for recursive case-insensitive search with line numbers.
- **`-E`** enables extended regex — allows `|` (OR), `+`, `?`, and `{}` patterns.
- Combine `grep` with pipes — `ps aux | grep python`, `tail -f log | grep error` — for powerful real-time filtering.

---

## 📚 Related Concepts


- [GNU grep Manual](https://www.gnu.org/software/grep/manual/grep.html)
- [Regular Expressions Reference](https://www.gnu.org/software/grep/manual/grep.html#Regular-Expressions)

</details>
