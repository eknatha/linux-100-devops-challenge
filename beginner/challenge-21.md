# Challenge 21: Find Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

The `find` command is one of the most powerful tools in Linux — it searches the filesystem for files and directories that match specified criteria. Unlike `grep` (which searches inside files), `find` searches for the **files themselves** based on name, type, size, age, permissions, owner, and more.

Common real-world uses:
- Locate a config file you cannot remember the path to
- Find all log files older than 30 days for cleanup
- Find files with incorrect permissions for security auditing
- Locate large files consuming disk space
- Find recently modified files to track changes

---

## 🧩 Task

Search for files by name, type, size, or other attributes.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Find a file by name
find / -name "filename.txt"

# Find in the current directory
find . -name "filename.txt"

# Case-insensitive search
find / -iname "filename.txt"
```

### How it works

| Part | Description |
|------|-------------|
| `find` | Search command — walks the directory tree |
| `/` | **Where** to search — `/` = entire filesystem |
| `.` | Current directory and subdirectories |
| `-name "pattern"` | Match files by **exact name** (case-sensitive) |
| `-iname "pattern"` | Match files by name (**case-insensitive**) |

---

## 🔢 `find` Command Structure

```
find  [path]  [expression]  [action]
  │      │          │            │
  │      │          │            └── What to do: -print, -delete, -exec
  │      │          └────────────── Criteria: -name, -type, -size, -mtime
  │      └───────────────────────── Where to search: /, ., /home, /var
  └──────────────────────────────── The find command
```

---

## 📖 Extended Examples

### Example 1 — Find by Exact Name

```bash
# Find a specific file anywhere on the system
find / -name "nginx.conf" 2>/dev/null

# Find in a specific directory
find /etc -name "*.conf"

# Find in the current directory
find . -name "*.txt"
```

```bash
# Output:
/etc/nginx/nginx.conf
/etc/nginx/sites-available/default.conf
```

---

### Example 2 — Case-Insensitive Search

```bash
# Matches: README.md, readme.md, Readme.MD, README.MD
find . -iname "readme.md"
```

---

### Example 3 — Find by Type

```bash
# Find only regular files
find /var/log -type f

# Find only directories
find /home -type d

# Find only symbolic links
find /etc -type l

# Find only block devices
find /dev -type b
```

| Type | Description |
|------|-------------|
| `f` | Regular **file** |
| `d` | **Directory** |
| `l` | Symbolic **link** |
| `b` | **Block** device |
| `c` | **Character** device |
| `p` | Named **pipe** |
| `s` | **Socket** |

---

### Example 4 — Find by File Size

```bash
# Find files larger than 100MB
find / -type f -size +100M 2>/dev/null

# Find files smaller than 1KB
find . -type f -size -1k

# Find files exactly 1GB
find / -type f -size 1G

# Find files between 10MB and 100MB
find / -type f -size +10M -size -100M
```

| Size Suffix | Meaning |
|-------------|---------|
| `c` | Bytes |
| `k` | Kilobytes (1024 bytes) |
| `M` | Megabytes |
| `G` | Gigabytes |
| `+N` | **Greater than** N |
| `-N` | **Less than** N |
| `N` | **Exactly** N |

---

### Example 5 — Find by Modification Time

```bash
# Files modified in the last 7 days
find /var/log -type f -mtime -7

# Files modified more than 30 days ago
find /var/log -type f -mtime +30

# Files modified in the last 60 minutes
find . -type f -mmin -60

# Files modified more than 24 hours ago
find . -type f -mmin +1440
```

| Time Flag | Description |
|-----------|-------------|
| `-mtime -N` | Modified **less than** N days ago |
| `-mtime +N` | Modified **more than** N days ago |
| `-mmin -N` | Modified **less than** N minutes ago |
| `-atime N` | **Accessed** N days ago |
| `-ctime N` | **Changed** (inode) N days ago |

---

### Example 6 — Find by Permissions

```bash
# Find world-writable files (security risk)
find / -type f -perm -o+w 2>/dev/null

# Find SUID files (run as owner)
find / -type f -perm /4000 2>/dev/null

# Find SGID files
find / -type f -perm /2000 2>/dev/null

# Find files with exact permission 644
find . -type f -perm 644

# Find files readable by everyone
find /home -type f -perm /o=r
```

---

### Example 7 — Find by Owner

```bash
# Find files owned by a specific user
find /home -user devuser

# Find files owned by a specific group
find /var/www -group www-data

# Find files NOT owned by root
find /etc -not -user root

# Find files owned by UID 1001
find / -uid 1001 2>/dev/null
```

---

### Example 8 — Combine Multiple Conditions

```bash
# Files larger than 100MB AND modified more than 30 days ago
find /var/log -type f -size +100M -mtime +30

# .log files modified in the last 7 days
find /var/log -type f -name "*.log" -mtime -7

# Using OR with -o
find . -name "*.txt" -o -name "*.log"

# Using NOT with -not or !
find /etc -type f -not -name "*.conf"
```

---

### Example 9 — Execute a Command on Found Files (`-exec`)

```bash
# Print details of each found file
find /var/log -name "*.log" -exec ls -lh {} \;

# Delete all .tmp files
find /tmp -type f -name "*.tmp" -exec rm {} \;

# Change permissions on found files
find /var/www -type f -exec chmod 644 {} \;

# Copy found files to a backup location
find /home/devuser -name "*.conf" -exec cp {} /backup/ \;
```

> 💡 `{}` is replaced by the found filename; `\;` ends the `-exec` command.

```bash
# More efficient: use + instead of \; to batch process
find /tmp -name "*.tmp" -exec rm {} +
```

---

### Example 10 — Delete Found Files (`-delete`)

```bash
# Delete files older than 30 days in /tmp
find /tmp -type f -mtime +30 -delete

# Delete empty directories
find . -type d -empty -delete

# Delete compressed log files older than 60 days
find /var/log -name "*.gz" -mtime +60 -delete
```

> ⚠️ Always test with `-print` before using `-delete`:
> ```bash
> find /tmp -type f -mtime +30 -print    # Preview first
> find /tmp -type f -mtime +30 -delete   # Then delete
> ```

---

### Example 11 — Find and Display Results Clearly

```bash
# Show long listing for each found file
find /var/log -name "*.log" -ls

# Show only the filename (no path)
find /etc -name "*.conf" -printf "%f\n"

# Show file size and path
find /var/log -type f -printf "%s bytes  %p\n" | sort -rn | head -10
```

---

### Example 12 — Real-World Use Cases

```bash
# Find the nginx config file
find /etc -name "nginx.conf" 2>/dev/null

# Find all SSH authorized_keys files
find /home -name "authorized_keys" 2>/dev/null

# Find recently modified config files
find /etc -type f -mtime -1 -ls

# Find files with incorrect permissions in /var/www
find /var/www -type f -perm /o+w -ls

# Find all .env files (may contain secrets)
find / -name ".env" -type f 2>/dev/null

# Find large files consuming disk space
find / -type f -size +500M 2>/dev/null | xargs ls -lh
```

---

## 🗂️ `find` Options — Quick Reference

| Option | Description |
|--------|-------------|
| `-name "pattern"` | Match by filename (case-sensitive) |
| `-iname "pattern"` | Match by filename (case-insensitive) |
| `-type f/d/l` | Match by type: file, dir, symlink |
| `-size +N/-N/N` | Match by size |
| `-mtime +N/-N` | Match by modification time (days) |
| `-mmin +N/-N` | Match by modification time (minutes) |
| `-perm MODE` | Match by permissions |
| `-user NAME` | Match by owner username |
| `-group NAME` | Match by group name |
| `-uid N` | Match by user ID |
| `-not` / `!` | Negate the condition |
| `-o` | OR — match either condition |
| `-exec CMD {} \;` | Run command on each result |
| `-exec CMD {} +` | Run command on all results (batched) |
| `-delete` | Delete found files |
| `-ls` | Long listing of found files |
| `-print` | Print found paths (default) |
| `-printf FORMAT` | Custom print format |
| `-maxdepth N` | Limit search to N directory levels |
| `-mindepth N` | Skip first N directory levels |

---

## 🔄 `find` vs `locate` — When to Use Each

| Feature | `find` | `locate` |
|---------|--------|----------|
| Speed | Slower (real-time scan) | Fast (uses database) |
| Up to date | ✅ Always current | ❌ Database may be stale |
| Criteria | Rich (size, time, perms) | Name only |
| Requires index | ❌ | ✅ (`updatedb`) |
| Best for | Precise, complex searches | Quick name lookups |

```bash
# Update locate database
sudo updatedb

# Quick name search
locate nginx.conf
```

---

## 🚀 Full Workflow — Find and Clean Old Log Files

```bash
# Step 1: Find large log files
find /var/log -type f -size +100M -ls

# Step 2: Find old compressed logs
find /var/log -name "*.gz" -mtime +30 -ls

# Step 3: Preview what would be deleted
find /var/log -name "*.gz" -mtime +30 -print

# Step 4: Count them
find /var/log -name "*.gz" -mtime +30 | wc -l

# Step 5: Delete after confirming
find /var/log -name "*.gz" -mtime +30 -delete

# Step 6: Verify disk space freed
df -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Permission denied errors flooding output | Add `2>/dev/null` to suppress |
| `-exec rm {} \;` too slow on many files | Use `-exec rm {} +` or `-delete` |
| `-delete` removes without confirmation | Always use `-print` first to preview |
| `find /` without `2>/dev/null` | Add `2>/dev/null` to hide permission errors |
| `-name` with path separator | `-name` matches filename only — use `-path` for full path matching |
| `-mtime 0` means today | `-mtime 0` matches files modified in last 24 hours (0 days ago) |

---

## 🔑 Key Takeaways

- `find . -name "pattern"` searches the current directory recursively for files matching the name.
- Always add `2>/dev/null` when searching from `/` to suppress permission-denied errors.
- `-type f` restricts to files; `-type d` restricts to directories — always specify the type.
- `-mtime +30` means "more than 30 days ago"; `-mtime -7` means "within the last 7 days".
- `-exec CMD {} \;` runs a command on every found file — use `+` instead of `\;` for better performance.
- **Always preview with `-print` before using `-delete`** — there is no undo.
- Use `locate` for quick name searches; use `find` for precise, criteria-based searches.

---

## 📚 Related Concepts

- [GNU find Manual](https://www.gnu.org/software/findutils/manual/html_mono/find.html)
- [locate / mlocate](https://linux.die.net/man/1/locate)

</details>
