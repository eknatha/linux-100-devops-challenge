# Challenge 16: Check Disk Usage in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Monitoring disk space is a critical daily task in Linux administration. A full disk can crash web servers, corrupt databases, halt logging, and prevent users from logging in. Knowing how to quickly check disk usage — both at the **filesystem level** and the **directory level** — is essential for every sysadmin and DevOps engineer.

Two primary tools:
- **`df`** — **D**isk **F**ree — shows usage at the **filesystem/partition level**
- **`du`** — **D**isk **U**sage — shows usage at the **directory/file level**

---

## 🧩 Task

Check how much disk space is used and available on the system.

---

<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Check filesystem-level disk usage
df -h

# Check directory-level disk usage
du -sh /var/log/
```

### How it works

| Command | Description |
|---------|-------------|
| `df -h` | Show disk **free** space per filesystem in **human-readable** format |
| `du -sh /path` | Show **disk usage** of a directory — **s**ummary, **h**uman-readable |

---

## 🔢 Understanding `df -h` Output

```bash
df -h
```

```
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   43G    7G    86%  /
/dev/sdb1       100G   20G   80G    20%  /data
tmpfs           1.9G     0  1.9G     0%  /dev/shm
/dev/sda2        20G   18G    2G    92%  /var
```

| Column | Description |
|--------|-------------|
| `Filesystem` | The disk partition or device |
| `Size` | Total capacity of the filesystem |
| `Used` | Space currently in use |
| `Avail` | Space still available |
| `Use%` | Percentage used — **watch for > 80%** |
| `Mounted on` | Mount point in the directory tree |

> ⚠️ When `Use%` reaches **100%**, services begin failing — aim to keep it below **80%**.

---

## 🔢 Understanding `du -sh` Output

```bash
du -sh /var/log/*
```

```
 18G  /var/log/journal
6.0G  /var/log/nginx
3.0G  /var/log/mysql
512M  /var/log/syslog
 40M  /var/log/auth.log
```

| Flag | Description |
|------|-------------|
| `-s` | **Summary** — show one total per argument |
| `-h` | **Human-readable** — show KB, MB, GB |

---

## 📖 Extended Examples

### Example 1 — Check All Filesystem Usage

```bash
df -h
```

```bash
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   43G    7G    86%  /
/dev/sdb1       100G   20G   80G    20%  /data
tmpfs           1.9G     0  1.9G     0%  /dev/shm
```

---

### Example 2 — Check a Specific Filesystem or Directory

```bash
# Check disk usage of a specific mount point
df -h /

# Check the filesystem containing /var
df -h /var

# Check disk usage of /home
df -h /home
```

---

### Example 3 — Show Filesystem Types

```bash
df -hT
```

```bash
Filesystem     Type      Size  Used  Avail  Use%  Mounted on
/dev/sda1      ext4       50G   43G    7G    86%  /
/dev/sdb1      xfs       100G   20G   80G    20%  /data
tmpfs          tmpfs     1.9G     0  1.9G     0%  /dev/shm
```

> `-T` adds the **filesystem type** column — useful for identifying ext4, xfs, tmpfs, nfs mounts.

---

### Example 4 — Check Inode Usage

A disk can be "full" even with free space if all **inodes** are exhausted:

```bash
df -i
```

```bash
Filesystem      Inodes   IUsed   IFree  IUse%  Mounted on
/dev/sda1      3276800  3276800      0   100%  /
/dev/sdb1      6553600   120000  6433600    2%  /data
```

> ⚠️ `IUse%` at **100%** means no new files can be created even if disk space is available.

---

### Example 5 — Check Directory Disk Usage

```bash
# Check total size of a single directory
du -sh /var/log/

# Check size of each item in /var/log
du -sh /var/log/*
```

```bash
 18G  /var/log/journal
6.0G  /var/log/nginx
3.0G  /var/log/mysql
512M  /var/log/syslog
```

---

### Example 6 — Sort by Size (Find the Biggest)

```bash
# Find largest directories at root level
du -sh /* 2>/dev/null | sort -hr | head -10
```

```bash
 32G  /var
  8G  /usr
  3G  /home
  1G  /opt
512M  /root
```

```bash
# Find top 10 largest files anywhere
find / -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -10 | \
  awk '{printf "%.0f MB  %s\n", $1/1048576, $2}'
```

---

### Example 7 — Drill Down into Large Directories

```bash
# Step 1: Find the biggest top-level directory
du -sh /* 2>/dev/null | sort -hr | head -5

# Step 2: Drill into /var
du -sh /var/* 2>/dev/null | sort -hr | head -5

# Step 3: Drill into /var/log
du -sh /var/log/* 2>/dev/null | sort -hr | head -5
```

---

### Example 8 — Check Disk Usage of Current Directory

```bash
# Total size of current directory
du -sh .

# Size of each item in current directory
du -sh *

# Recursive breakdown of all subdirectories
du -h --max-depth=1
```

---

### Example 9 — Alert When Disk Is Over a Threshold

```bash
# One-liner: print alert if any filesystem is over 80%
df -h | awk '$5+0 > 80 {print "⚠️  Alert:", $1, "is at", $5, "on", $6}'
```

```bash
⚠️  Alert: /dev/sda1 is at 86% on /
⚠️  Alert: /dev/sda2 is at 92% on /var
```

---

### Example 10 — Check Disk Usage in a Script

```bash
#!/bin/bash
THRESHOLD=80

df -h | awk -v t="$THRESHOLD" 'NR>1 {
  gsub(/%/, "", $5)
  if ($5+0 > t) {
    print "⚠️  DISK ALERT: "$6" is at "$5"% (threshold: "t"%)"
  }
}'
```

---

## 🗂️ `df` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-h` | **Human-readable** (KB, MB, GB) |
| `-H` | Human-readable using powers of 1000 |
| `-T` | Show **filesystem type** |
| `-i` | Show **inode** usage instead of blocks |
| `-t TYPE` | Show only filesystems of a specific type |
| `-x TYPE` | **Exclude** a filesystem type |
| `--total` | Add a **total** row at the bottom |

---

## 🗂️ `du` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-s` | **Summary** — one total per argument |
| `-h` | **Human-readable** sizes |
| `-a` | Show **all** files (not just directories) |
| `--max-depth=N` | Limit recursion to N levels deep |
| `-c` | Show a **grand total** at the end |
| `-x` | **Stay on one filesystem** — don't cross mounts |
| `--exclude=PATTERN` | Exclude files matching a pattern |

---

## 🔄 `df` vs `du` — When to Use Each

| Tool | Scope | Best For |
|------|-------|---------|
| `df` | Filesystem level | Which **partition** is running out of space |
| `du` | Directory/file level | What is **consuming** the space |

> Always start with `df` to find the full filesystem, then use `du` to find what is filling it.

---

## 🚀 Full Workflow — Investigate Full Disk

```bash
# Step 1: Identify the full filesystem
df -h

# Step 2: Find top space consumers at root
du -sh /* 2>/dev/null | sort -hr | head -10

# Step 3: Drill into the largest directory
du -sh /var/* 2>/dev/null | sort -hr | head -10

# Step 4: Find the specific large files
find /var/log -type f -size +100M -exec ls -lh {} \;

# Step 5: Check inode usage too
df -i

# Step 6: Take action (clean logs, trim journal, etc.)
sudo journalctl --vacuum-size=500M

# Step 7: Verify disk space is recovered
df -h
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `df` shows full but `du` shows less | Check for deleted-but-open files: `lsof \| grep deleted` |
| Disk full but no obvious files | Check inode usage: `df -i` |
| `du /*` misses hidden directories | Add hidden dirs: `du -sh /.[!.]* /* 2>/dev/null` |
| Errors in `du` output | Add `2>/dev/null` to suppress permission denied messages |
| `sort -h` not sorting correctly | Ensure GNU sort is installed — macOS `sort` lacks `-h` |

---

## 🔑 Key Takeaways

- `df -h` shows **filesystem-level** usage — which partition or mount point is full.
- `du -sh /path/*` shows **directory-level** usage — what is consuming the space inside a path.
- Always start with `df`, then drill down with `du` to find the culprit.
- Use `sort -hr` with `du` output to sort **largest first** in human-readable format.
- Check **inode usage** with `df -i` — a disk can be "full" even with free space.
- **`Use% > 80%`** is a warning sign — take action before it reaches 100%.

---

## 📚 Related Concepts

- [GNU df Manual](https://www.gnu.org/software/coreutils/manual/html_node/df-invocation.html)
- [GNU du Manual](https://www.gnu.org/software/coreutils/manual/html_node/du-invocation.html)


</details>
