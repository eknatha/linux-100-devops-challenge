# Challenge 41: Find Large Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Finding large files is one of the most common disk management tasks in Linux. When a disk fills up unexpectedly, the first step is to identify which files are consuming the most space. The `find` command combined with `-size` is the most powerful and flexible way to locate files by size — whether you are cleaning up a full disk or performing routine maintenance.

---

## 🧩 Task

Find files larger than 100MB on the system.

---

<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Find files larger than 100MB
find / -type f -size +100M 2>/dev/null

# Find and show file sizes
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Find in a specific directory
find /var/log -type f -size +100M
```

### How it works

| Part | Description |
|------|-------------|
| `find /` | Search **starting from root** `/` |
| `-type f` | Match **regular files** only (not directories) |
| `-size +100M` | Match files **larger than** 100 megabytes |
| `2>/dev/null` | Suppress permission-denied errors |
| `-exec ls -lh {} \;` | Run `ls -lh` on each found file |

---

## 🔢 Size Suffixes and Operators

| Suffix | Meaning |
|--------|---------|
| `c` | Bytes |
| `k` | Kilobytes (1024 bytes) |
| `M` | Megabytes |
| `G` | Gigabytes |

| Operator | Meaning |
|----------|---------|
| `+100M` | **Greater than** 100MB |
| `-100M` | **Less than** 100MB |
| `100M` | **Exactly** 100MB |

---

## 📖 Extended Examples

### Example 1 — Find Files Larger Than 100MB

```bash
find / -type f -size +100M 2>/dev/null
```

```bash
/var/log/journal/abc123/system.journal
/var/lib/mysql/ibdata1
/home/devuser/backup.tar.gz
/var/log/nginx/access.log
```

---

### Example 2 — Show File Sizes with Results

```bash
# Display with human-readable sizes
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

```bash
-rw-r--r-- 1 root     root    18G Mar 21 10:00 /var/log/journal/abc123/system.journal
-rw-r--r-- 1 mysql    mysql  512M Mar 21 09:00 /var/lib/mysql/ibdata1
-rw-r--r-- 1 devuser  devuser 256M Mar 20 22:00 /home/devuser/backup.tar.gz
```

---

### Example 3 — Find in a Specific Directory

```bash
# Search only in /var/log
find /var/log -type f -size +100M

# Search in home directories
find /home -type f -size +100M

# Search in /var/lib (databases, etc.)
find /var/lib -type f -size +100M 2>/dev/null
```

---

### Example 4 — Sort Results by Size (Largest First)

```bash
# Find and sort by size — largest first
find / -type f -size +100M -printf '%s %p\n' 2>/dev/null | \
  sort -rn | \
  head -20 | \
  awk '{printf "%.0f MB  %s\n", $1/1048576, $2}'
```

```bash
18432 MB  /var/log/journal/abc123/system.journal
  512 MB  /var/lib/mysql/ibdata1
  256 MB  /home/devuser/backup.tar.gz
  128 MB  /var/log/nginx/access.log
```

---

### Example 5 — Find Files of Different Size Thresholds

```bash
# Larger than 1GB
find / -type f -size +1G 2>/dev/null

# Between 100MB and 1GB
find / -type f -size +100M -size -1G 2>/dev/null

# Larger than 500MB
find / -type f -size +500M 2>/dev/null

# Exactly 100MB (rarely useful)
find / -type f -size 100M 2>/dev/null
```

---

### Example 6 — Limit Search Depth

```bash
# Only search 3 levels deep (faster)
find / -maxdepth 3 -type f -size +100M 2>/dev/null

# Search only top-level directories
find / -maxdepth 2 -type f -size +100M 2>/dev/null
```

---

### Example 7 — Find Large Files by Type

```bash
# Find large log files
find /var/log -name "*.log" -size +100M

# Find large compressed files
find / -name "*.gz" -size +100M 2>/dev/null

# Find large database files
find /var/lib -name "*.db" -o -name "*.sql" | xargs ls -lh 2>/dev/null

# Find large video files
find /home -name "*.mp4" -o -name "*.mkv" -size +500M 2>/dev/null
```

---

### Example 8 — Find Large Files Modified Recently

```bash
# Large files modified in the last 7 days
find / -type f -size +100M -mtime -7 2>/dev/null

# Large files modified more than 30 days ago (candidates for cleanup)
find /var/log -type f -size +100M -mtime +30
```

---

### Example 9 — Use `du` to Find Large Directories First, Then Drill Down

```bash
# Step 1: Find the biggest top-level directories
du -sh /* 2>/dev/null | sort -hr | head -10

# Step 2: Drill into the largest
du -sh /var/* 2>/dev/null | sort -hr | head -10

# Step 3: Find specific large files inside
find /var/log -type f -size +100M -exec ls -lh {} \;
```

---

### Example 10 — Interactive Search with `ncdu`

```bash
# Install ncdu (NCurses Disk Usage)
sudo apt install ncdu    # Debian/Ubuntu
sudo dnf install ncdu    # RHEL

# Launch interactive browser
sudo ncdu /

# Navigate with arrow keys, press d to delete
```

> `ncdu` is the most **user-friendly** tool for finding large files — visual, interactive, and shows everything ranked by size.

---

### Example 11 — Find and Delete Old Large Files

```bash
# Preview what would be deleted (safe — no deletion)
find /var/log -type f -size +100M -mtime +30 -print

# Delete after confirming the list
find /var/log -type f -size +100M -mtime +30 -delete

# Or use rm with confirmation
find /var/log -type f -size +100M -mtime +30 -exec rm -i {} \;
```

> ⚠️ Always preview with `-print` before using `-delete`.

---

## 🗂️ `find` Size Options — Quick Reference

| Command | Description |
|---------|-------------|
| `find / -size +100M` | Files larger than 100MB |
| `find / -size +1G` | Files larger than 1GB |
| `find / -size -1k` | Files smaller than 1KB |
| `find / -size +100M -size -1G` | Files between 100MB and 1GB |
| `find / -size +100M -ls` | Large files with long listing |
| `find / -size +100M -printf '%s %p\n'` | Size in bytes + path |
| `find / -size +100M -delete` | Delete large files (careful!) |

---

## 🔄 `find` vs `du` for Finding Large Items

| Tool | Best For |
|------|----------|
| `find -size` | Finding specific **files** above a size threshold |
| `du -sh * \| sort -hr` | Finding large **directories** |
| `ncdu` | Interactive visual exploration |

> Use `du` first to find large directories, then `find` to locate the specific large files inside.

---

## 🚀 Full Workflow — Clean Up Disk Space

```bash
# Step 1: Confirm the disk is getting full
df -h

# Step 2: Find large directories
du -sh /* 2>/dev/null | sort -hr | head -10

# Step 3: Find specific large files
find /var -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Step 4: Sort by size — largest first
find /var -type f -size +100M -printf '%s %p\n' 2>/dev/null | sort -rn | head -20

# Step 5: Investigate each large file
# Is it a log? Truncate or rotate.
# Is it a backup? Move to cold storage or delete.
# Is it a database file? Optimize or archive.

# Step 6: Clean up (example: old compressed logs)
find /var/log -name "*.gz" -size +100M -mtime +30 -delete

# Step 7: Verify space was freed
df -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Permission errors flooding output | Add `2>/dev/null` to suppress |
| `find / -delete` too dangerous | Always preview with `-print` first |
| Confusing `-size +100M` and `-size 100M` | `+` means greater than; without `+` means exactly |
| `M` is megabytes but `m` is not valid | Use uppercase: `k`, `M`, `G` |
| `-exec ls -lh {} \;` is slow on many files | Use `-printf '%s %p\n'` for fast size output |
| Deleting active log files | Use `truncate -s 0` instead of `rm` for active logs |

---

## 🔑 Key Takeaways

- `find / -type f -size +100M` finds all files **larger than 100MB** starting from root.
- Always add `2>/dev/null` when searching from `/` to suppress permission-denied errors.
- Use `-printf '%s %p\n' | sort -rn` to **sort results by size** largest first.
- **Preview before deleting** — use `-print` or `-ls` before switching to `-delete`.
- Use `du -sh` to find **large directories** first, then `find` to drill into specific files.
- `-size +100M -size -1G` combines two size conditions — files **between** two sizes.
- `ncdu` is the fastest **interactive** way to find and clean up large files.

---

## 📚 Related Concepts

- [GNU find Manual](https://www.gnu.org/software/findutils/manual/html_mono/find.html)

</details>
