# Challenge 73: Diagnose Disk Space Issues in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

A full disk is one of the most common and disruptive Linux problems — it can silently crash web servers, corrupt databases, halt log collection, and prevent users from logging in. The key to resolving disk space issues quickly is knowing how to **locate what is consuming space** and drill down from the filesystem level to the exact files responsible.

Two tools form the foundation of disk space investigation:
- **`df`** — shows disk usage at the **filesystem level** (what partitions are full)
- **`du`** — shows disk usage at the **directory/file level** (what is actually consuming space)

---

## 🧩 Task

Identify which filesystem is filling up and find the directories or files consuming the most space.

---



<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Step 1: Check which filesystem is full
df -h

# Step 2: Find the top space consumers at the root level
du -sh /* | sort -hr | head
```

### How it works

#### `df -h`

| Part | Description |
|------|-------------|
| `df` | **D**isk **F**ree — reports filesystem-level disk usage |
| `-h` | **Human-readable** output (KB, MB, GB, TB) |

#### `du -sh /* | sort -hr | head`

| Part | Description |
|------|-------------|
| `du` | **D**isk **U**sage — reports directory/file-level size |
| `-s` | **Summary** — one total per argument (not per subdirectory) |
| `-h` | **Human-readable** format |
| `/*` | Scan all top-level directories under root |
| `sort -hr` | Sort **h**uman-readable sizes in **r**everse (largest first) |
| `head` | Show only the top 10 results |

---

## 🔢 Understanding `df -h` Output

```bash
df -h
```

```
Filesystem       Size  Used  Avail  Use%  Mounted on
/dev/sda1         50G   47G    3G    95%  /
/dev/sdb1        100G   20G   80G    20%  /data
tmpfs            1.9G    0     1.9G    0%  /dev/shm
/dev/sda2         20G   18G    2G    92%  /var
```

| Column | Description |
|--------|-------------|
| `Filesystem` | Device or filesystem name |
| `Size` | Total capacity |
| `Used` | Space consumed |
| `Avail` | Space remaining |
| `Use%` | Percentage used — **watch for > 80%** |
| `Mounted on` | Mount point |

> ⚠️ `Use%` at **95%** on `/` is critical — many systems begin failing at 100% and some applications fail even earlier.

---

## 📖 Extended Examples

### Example 1 — Check Filesystem Usage

```bash
df -h
```

```
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   47G    3G    95%  /
/dev/sda2        20G   18G    2G    92%  /var
```

> Both `/` and `/var` are critically full. Start investigating `/var` since it often contains logs.

---

### Example 2 — Find Top Space Consumers at Root Level

```bash
du -sh /* 2>/dev/null | sort -hr | head
```

```bash
32G  /var
 8G  /usr
 3G  /home
 1G  /opt
512M /root
200M /tmp
```

> 💡 `2>/dev/null` suppresses permission-denied errors for directories you cannot read.

---

### Example 3 — Drill Down into a Large Directory

```bash
# Drill into /var
du -sh /var/* 2>/dev/null | sort -hr | head
```

```bash
28G  /var/log
2.1G /var/lib
512M /var/cache
 48M /var/spool
```

```bash
# Drill further into /var/log
du -sh /var/log/* 2>/dev/null | sort -hr | head
```

```bash
18G  /var/log/journal
 6G  /var/log/nginx
 3G  /var/log/mysql
512M /var/log/syslog
 40M /var/log/auth.log
```

---

### Example 4 — Find the Top 20 Largest Files Anywhere on the System

```bash
find / -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -20 | awk '{printf "%.1fMB  %s\n", $1/1048576, $2}'
```

```bash
18432.0MB  /var/log/journal/abc123/system.journal
 6144.0MB  /var/log/nginx/access.log
 2048.0MB  /var/lib/mysql/ibdata1
  512.0MB  /var/log/mysql/slow.log
  256.0MB  /home/devuser/backup.tar.gz
```

---

### Example 5 — Find Large Files in a Specific Directory

```bash
# Large files under /var/log
find /var/log -type f -size +100M -exec ls -lh {} \;
```

```bash
-rw-r--r-- 1 root root  18G Mar 21 10:00 /var/log/journal/abc123/system.journal
-rw-r--r-- 1 www-data www-data 6.0G Mar 21 09:45 /var/log/nginx/access.log
```

```bash
# Find files larger than 1GB anywhere
find / -type f -size +1G 2>/dev/null | xargs ls -lh
```

---

### Example 6 — Check for Deleted Files Still Held Open

A very common trap: a large file is deleted, but a process still has it open — the space is **not freed until the process releases it**. `df` shows the disk as full even though `du` finds nothing!

```bash
# Find deleted files still held open by processes
sudo lsof | grep '(deleted)'
```

```bash
nginx   5201  www-data  3w  REG  8,1  6442450944  /var/log/nginx/access.log (deleted)
```

```bash
# Fix: restart the process holding the deleted file open
sudo systemctl restart nginx

# Or send SIGUSR1 to reopen log files (nginx-specific)
sudo kill -USR1 $(cat /run/nginx.pid)
```

> 💡 This is one of the most frequently missed causes of a "disk full but nothing to delete" situation.

---

### Example 7 — Check for Inode Exhaustion

A disk can be "full" even with free space if all **inodes** (file table entries) are exhausted. This happens with millions of tiny files.

```bash
df -i
```

```bash
Filesystem      Inodes  IUsed   IFree  IUse%  Mounted on
/dev/sda1      3276800  3276800      0   100%  /
/dev/sdb1      6553600   120000  6433600    2%  /data
```

> `IUse%` at **100%** means no new files can be created even if there is free disk space.

```bash
# Find directory with the most files (likely culprit for inode exhaustion)
find / -xdev -printf '%h\n' 2>/dev/null | sort | uniq -c | sort -rn | head -10
```

---

### Example 8 — Identify and Clean Up Log Files

```bash
# Check journal size
journalctl --disk-usage

# Trim journal to 500MB
sudo journalctl --vacuum-size=500M

# Trim journal to keep only last 7 days
sudo journalctl --vacuum-time=7d

# Manually rotate and compress log files
sudo logrotate -f /etc/logrotate.conf

# Truncate a specific oversize log (preserves the file, clears content)
sudo truncate -s 0 /var/log/nginx/access.log

# Remove old rotated logs
sudo find /var/log -name "*.gz" -mtime +30 -delete
sudo find /var/log -name "*.1" -mtime +7 -delete
```

---

### Example 9 — Find and Remove Temporary Files

```bash
# Large files in /tmp
find /tmp -type f -size +50M -exec ls -lh {} \;

# Old temp files (not accessed in 7 days)
find /tmp -type f -atime +7 -delete

# Check for large core dump files
find / -name "core" -o -name "core.*" 2>/dev/null | xargs ls -lh 2>/dev/null

# Remove core dumps
find / -name "core" -o -name "core.[0-9]*" 2>/dev/null | xargs rm -f
```

---

### Example 10 — Check for Large Docker Artifacts (If Applicable)

```bash
# Docker disk usage overview
docker system df

# Detailed breakdown
docker system df -v

# Remove unused containers, images, networks, and build cache
docker system prune

# Remove all unused images (not just dangling)
docker system prune -a
```

```bash
TYPE            TOTAL  ACTIVE  SIZE      RECLAIMABLE
Images           24     3      18.2GB    14.5GB (79%)
Containers        3     2       512MB     0 (0%)
Volumes           8     2      4.1GB     3.2GB (78%)
Build Cache      45     0      2.3GB     2.3GB
```

---

### Example 11 — Interactive Disk Usage with `ncdu`

```bash
# Install ncdu
sudo apt install ncdu     # Debian/Ubuntu
sudo dnf install ncdu     # RHEL/CentOS

# Launch interactive browser
sudo ncdu /
```

> `ncdu` (NCurses Disk Usage) is the most **user-friendly** tool for navigating disk usage interactively. Use arrow keys to navigate, `d` to delete files.

---

### Example 12 — Full Disk Investigation Script

```bash
#!/bin/bash

THRESHOLD=85
LOGFILE="/var/log/disk_investigation.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "============================================" | tee "$LOGFILE"
echo " Disk Investigation Report — $DATE"          | tee -a "$LOGFILE"
echo "============================================" | tee -a "$LOGFILE"

echo -e "\n[1] Filesystem Usage]"                  | tee -a "$LOGFILE"
df -h | awk -v t="$THRESHOLD" 'NR==1 || int($5) > t' | tee -a "$LOGFILE"

echo -e "\n[2] Top Directories at Root]"           | tee -a "$LOGFILE"
du -sh /* 2>/dev/null | sort -hr | head -10        | tee -a "$LOGFILE"

echo -e "\n[3] Top 10 Largest Files on System]"    | tee -a "$LOGFILE"
find / -type f -printf '%s %p\n' 2>/dev/null \
  | sort -rn | head -10 \
  | awk '{printf "%.0f MB  %s\n", $1/1048576, $2}' | tee -a "$LOGFILE"

echo -e "\n[4] Deleted Files Still Held Open]"     | tee -a "$LOGFILE"
sudo lsof 2>/dev/null | grep "(deleted)"            | tee -a "$LOGFILE"

echo -e "\n[5] Inode Usage]"                       | tee -a "$LOGFILE"
df -i | awk 'NR==1 || int($5) > 80'               | tee -a "$LOGFILE"

echo -e "\nReport saved: $LOGFILE"
```

---

## 🗂️ `df` vs `du` — When to Use Each

| Tool | Scope | Best For |
|------|-------|---------|
| `df` | Filesystem level | Identify which **partition/mount** is full |
| `du` | Directory/file level | Identify **what is consuming** the space |

> Always start with `df` to find the full filesystem, then use `du` to drill down into what is filling it.

---

## 🛠️ Disk Space Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `df -h` | Snapshot | Filesystem-level usage overview |
| `du -sh` | Snapshot | Directory-level space consumption |
| `find -size` | Search | Locate specific large files |
| `lsof` + grep deleted | Check | Detect deleted-but-open file space leaks |
| `df -i` | Inodes | Detect inode exhaustion |
| `ncdu` | Interactive | Visual filesystem navigation |
| `journalctl --vacuum` | Cleanup | Trim systemd journal |
| `docker system prune` | Cleanup | Reclaim Docker disk space |

---

## 🚀 Full Workflow — Locate and Fix a Full Disk

```bash
# Step 1: Identify the full filesystem
df -h

# Step 2: Find the biggest top-level directories on the full mount
du -sh /var/* 2>/dev/null | sort -hr | head

# Step 3: Drill deeper into the largest directory
du -sh /var/log/* 2>/dev/null | sort -hr | head

# Step 4: Find the specific large files
find /var/log -type f -size +100M -exec ls -lh {} \;

# Step 5: Check for deleted-but-open files
sudo lsof | grep '(deleted)'

# Step 6: Check inode usage
df -i

# Step 7: Take action
# Clean journal logs
sudo journalctl --vacuum-size=500M

# Truncate oversized logs
sudo truncate -s 0 /var/log/nginx/access.log

# Restart process holding deleted file
sudo systemctl restart nginx

# Step 8: Verify space is recovered
df -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `du` shows free space but `df` shows full | Deleted files held open by processes — check `lsof | grep deleted` |
| Deleted files but disk still full | Restart the process holding them open |
| Disk has space but can't create files | Check inode exhaustion with `df -i` |
| `du /*` misses files in hidden dirs | Add hidden directories: `du -sh /.[!.]* /* 2>/dev/null` |
| Errors in `du` output | Add `2>/dev/null` to suppress permission-denied messages |
| Docker images silently eating space | Run `docker system df` and `docker system prune` |

---

## 🔑 Key Takeaways

- Always start with **`df -h`** to find which filesystem is full, then use **`du`** to find what is filling it.
- **Drill down** progressively: `/*` → `/var/*` → `/var/log/*` until you find the culprit.
- **Deleted files held open** by running processes are invisible to `du` but still consume space — check with `lsof | grep deleted`.
- **Inode exhaustion** (`df -i`) can make a disk "full" even with free blocks — caused by millions of tiny files.
- `sort -hr` is the key to sorting human-readable sizes correctly (G before M before K).
- Use `ncdu` for the fastest interactive investigation when you need to navigate the tree visually.
- Regular **log rotation** and **journal vacuuming** prevent disk space issues from recurring.

---

## 📚 Related Concepts

- [ncdu](https://dev.yorhel.nl/ncdu) — interactive ncurses disk usage analyzer
- [logrotate](https://linux.die.net/man/8/logrotate) — automated log rotation and compression
- [GNU df Manual](https://www.gnu.org/software/coreutils/manual/html_node/df-invocation.html)
- [GNU du Manual](https://www.gnu.org/software/coreutils/manual/html_node/du-invocation.html)

</details>
