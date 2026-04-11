# Challenge 13: Count Words, Lines, and Characters in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

The `wc` (**W**ord **C**ount) command is a simple but powerful tool for counting the number of **lines**, **words**, and **characters** in a file or input stream. It is widely used in scripts, log analysis, and data processing pipelines to quickly measure file size, count records, or verify expected output.

Common real-world uses:
- Count how many log entries exist in a log file
- Count lines of code in a source file
- Verify the number of records in a CSV file
- Count how many processes are running
- Measure the size of command output

---

## 🧩 Task

Count the number of lines, words, and characters in a file.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Count lines, words, and characters (all at once)
wc filename.txt

# Count lines only
wc -l filename.txt

# Count words only
wc -w filename.txt

# Count characters only
wc -c filename.txt
```

### How it works

| Flag | Description |
|------|-------------|
| `wc` (no flags) | Shows **lines**, **words**, and **bytes** (in that order) |
| `-l` | Count **lines** only |
| `-w` | Count **words** only |
| `-c` | Count **bytes** (characters for ASCII text) |
| `-m` | Count **characters** (handles multi-byte/Unicode correctly) |

---

## 🔢 Understanding `wc` Output

```bash
wc filename.txt
```

```
  42  187  1024  filename.txt
  │    │    │      └── filename
  │    │    └───────── bytes (characters)
  │    └────────────── words
  └─────────────────── lines
```

> 💡 The output order is always: **lines → words → bytes → filename**

---

## 📖 Extended Examples

### Example 1 — Count Everything at Once

```bash
wc filename.txt
```

```bash
  42  187  1024  filename.txt
```

> 42 lines, 187 words, 1024 bytes in `filename.txt`

---

### Example 2 — Count Lines Only

```bash
wc -l filename.txt
```

```bash
42 filename.txt
```

> `-l` is the most commonly used flag — counting lines is the most frequent use case.

---

### Example 3 — Count Words Only

```bash
wc -w filename.txt
```

```bash
187 filename.txt
```

---

### Example 4 — Count Characters (Bytes)

```bash
# Count bytes (same as characters for ASCII)
wc -c filename.txt

# Count characters (correct for Unicode/multi-byte)
wc -m filename.txt
```

```bash
1024 filename.txt
```

> 💡 Use `-m` instead of `-c` when working with files that contain Unicode characters (non-English text, emojis) — `-c` counts bytes, `-m` counts actual characters.

---

### Example 5 — Count Multiple Files at Once

```bash
wc file1.txt file2.txt file3.txt
```

```bash
  42  187  1024  file1.txt
  18   74   512  file2.txt
  91  320  2048  file3.txt
 151  581  3584  total
```

> `wc` automatically adds a **total** row when processing multiple files.

---

### Example 6 — Count Lines in All Files of a Type

```bash
# Count lines in all .log files
wc -l *.log

# Count lines in all Python files
wc -l *.py

# Count all lines across all text files
wc -l *.txt | tail -1   # Show only the total
```

---

### Example 7 — Count Lines of Piped Output

```bash
# Count running processes
ps aux | wc -l

# Count files in a directory
ls /etc | wc -l

# Count lines containing "error" in a log
grep -i "error" /var/log/syslog | wc -l

# Count open file descriptors
lsof | wc -l
```

```bash
# ps aux | wc -l output:
215
# 215 processes (including header line)
```

---

### Example 8 — Count Words in a Specific String

```bash
# Count words in a string using echo
echo "The quick brown fox jumps over the lazy dog" | wc -w
```

```bash
9
```

---

### Example 9 — Find the Longest Line in a File

```bash
# Find the length of the longest line
wc -L filename.txt
```

```bash
128 filename.txt
```

> `-L` prints the length of the **longest line** in the file — useful for checking code style compliance or column width.

---

### Example 10 — Use `wc` in Scripts for Validation

```bash
#!/bin/bash

FILE="data.csv"
EXPECTED_LINES=100

# Count lines in the file
ACTUAL_LINES=$(wc -l < "$FILE")

if [ "$ACTUAL_LINES" -eq "$EXPECTED_LINES" ]; then
  echo "✅ File has expected $EXPECTED_LINES lines"
else
  echo "❌ Expected $EXPECTED_LINES lines but found $ACTUAL_LINES"
  exit 1
fi
```

```bash
✅ File has expected 100 lines
```

> 💡 `wc -l < file` (with `<` input redirection) returns **only the number** without the filename — perfect for capturing in a variable.

---

### Example 11 — Count Empty Lines in a File

```bash
# Count empty lines
grep -c "^$" filename.txt

# Count non-empty lines
grep -c "." filename.txt

# Count lines with content using wc and grep
grep -v "^$" filename.txt | wc -l
```

---

### Example 12 — Count Lines, Words, and Characters of Command Output

```bash
# Count characters in today's date
date | wc -c

# Count words in system info
uname -a | wc -w

# Measure size of a command's output in bytes
ls -la /etc | wc -c
```

---

## 🗂️ `wc` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-l` | Count **lines** |
| `-w` | Count **words** |
| `-c` | Count **bytes** |
| `-m` | Count **characters** (Unicode-aware) |
| `-L` | Print length of **longest line** |

---

## 🔄 Common `wc` Patterns

| Goal | Command |
|------|---------|
| Count lines in a file | `wc -l file.txt` |
| Count lines in command output | `command \| wc -l` |
| Count only the number (no filename) | `wc -l < file.txt` |
| Count files in directory | `ls /path \| wc -l` |
| Count processes | `ps aux \| wc -l` |
| Count log errors | `grep "error" log.txt \| wc -l` |
| Count all files recursively | `find /path -type f \| wc -l` |
| Get total lines across files | `wc -l *.txt \| tail -1` |

---

## 🚀 Full Workflow — Analyze a Log File

```bash
# Step 1: Total number of log entries
wc -l /var/log/syslog

# Step 2: Count error entries
grep -i "error" /var/log/syslog | wc -l

# Step 3: Count warning entries
grep -i "warn" /var/log/syslog | wc -l

# Step 4: Count entries from today
grep "$(date '+%b %e')" /var/log/syslog | wc -l

# Step 5: Compare multiple log files
wc -l /var/log/*.log

# Step 6: Total size of all logs
wc -c /var/log/*.log | tail -1

# Step 7: Find the log file with the most lines
wc -l /var/log/*.log | sort -rn | head -3
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `wc -l file` includes filename in output | Use `wc -l < file` to get only the number |
| `-c` counts bytes not characters for Unicode | Use `-m` for correct character count with Unicode |
| Counting includes header lines | Subtract 1: `$(ps aux \| wc -l) - 1` |
| `wc` with no file reads from stdin indefinitely | Always specify a file or pipe input to it |
| Last line not counted if no newline at end | `wc -l` counts newlines — a file without final newline may show N-1 |

---

## 🔑 Key Takeaways

- `wc filename.txt` shows **lines, words, and bytes** in that order.
- `-l` is the most used flag — it counts **lines** and works perfectly with pipes.
- Use `wc -l < file` (with input redirection) to get **only the number** without the filename — essential for scripts.
- `wc` works with **pipes** — `ps aux | wc -l` counts running processes; `grep "error" log | wc -l` counts error lines.
- Use `-m` instead of `-c` for **Unicode-aware** character counting.
- `-L` finds the **longest line length** — useful for code formatting checks.
- Always pipe to `tail -1` when running `wc` on multiple files to see just the **total**.

---

## 📚 Related Concepts

- [GNU wc Manual](https://www.gnu.org/software/coreutils/manual/html_node/wc-invocation.html)

</details>
