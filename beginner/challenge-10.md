# Challenge 10: Copy and Move Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Copying and moving files** are two of the most common operations in Linux. Whether you are backing up a configuration file before editing it, organizing files into folders, or deploying application files — `cp` and `mv` are the essential tools.

Key difference:
- **`cp`** — creates a **copy** of the file; the original remains
- **`mv`** — **moves** the file to a new location (or renames it); the original is gone

---

## 🧩 Task

Copy a file to a new location and move (or rename) a file.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Copy a file
cp source.txt destination.txt

# Move (or rename) a file
mv source.txt destination.txt
```

### How it works

| Command | Description |
|---------|-------------|
| `cp source dest` | **C**o**p**y — duplicates the file; original stays |
| `mv source dest` | **M**o**v**e — relocates the file; original is removed |

---

## 🔢 `cp` vs `mv` — Key Difference

```
cp file.txt backup.txt
├── file.txt      ← original still exists
└── backup.txt    ← new copy created

mv file.txt newname.txt
└── newname.txt   ← file.txt is gone, renamed to newname.txt
```

---

## 📖 Extended Examples

### Example 1 — Copy a File

```bash
# Copy file.txt to backup.txt in the same directory
cp file.txt backup.txt

# Verify both exist
ls -la
```

```bash
-rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 backup.txt
-rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 file.txt
```

---

### Example 2 — Copy a File to Another Directory

```bash
# Copy file.txt into the /tmp directory
cp file.txt /tmp/

# Copy and give it a different name in the new location
cp file.txt /tmp/file_backup.txt
```

---

### Example 3 — Copy Multiple Files to a Directory

```bash
# Copy multiple files to a directory
cp file1.txt file2.txt file3.txt /backup/

# Copy all .txt files to a directory
cp *.txt /backup/
```

---

### Example 4 — Copy a Directory Recursively

```bash
# Copy a directory and all its contents
cp -r mydir/ /backup/mydir/

# Copy a directory with verbose output
cp -rv mydir/ /backup/mydir/
```

```bash
# -rv output:
'mydir/' -> '/backup/mydir/'
'mydir/file1.txt' -> '/backup/mydir/file1.txt'
'mydir/subdir/' -> '/backup/mydir/subdir/'
'mydir/subdir/file2.txt' -> '/backup/mydir/subdir/file2.txt'
```

> ⚠️ Without `-r`, `cp` refuses to copy directories — always use `-r` for directories.

---

### Example 5 — Copy with Backup of Existing File

```bash
# If destination exists, keep a backup before overwriting
cp -b file.txt /etc/app/config.txt

# Result: original saved as config.txt~
ls /etc/app/
# config.txt  config.txt~
```

---

### Example 6 — Copy and Preserve Permissions and Timestamps

```bash
# Preserve permissions, timestamps, and ownership
cp -p file.txt backup.txt

# Copy directory preserving all attributes (same as -dR --preserve=all)
cp -a mydir/ /backup/mydir/
```

```bash
# Without -p: copy gets current timestamp
# With -p: copy keeps original timestamp

ls -la
# -rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 file.txt     ← original
# -rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 backup.txt   ← same timestamp with -p
```

---

### Example 7 — Move (Rename) a File

```bash
# Rename file.txt to newname.txt (same directory)
mv file.txt newname.txt

# Verify: file.txt is gone, newname.txt exists
ls -la
```

```bash
-rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 newname.txt
# file.txt no longer exists
```

---

### Example 8 — Move a File to Another Directory

```bash
# Move file.txt into /tmp/
mv file.txt /tmp/

# Move and rename at the same time
mv file.txt /backup/file_backup.txt
```

---

### Example 9 — Move Multiple Files to a Directory

```bash
# Move multiple files to /archive/
mv file1.txt file2.txt file3.txt /archive/

# Move all .log files to /var/log/archive/
mv *.log /var/log/archive/

# Move a directory
mv mydir/ /backup/mydir/
```

---

### Example 10 — Interactive and Verbose Mode

```bash
# Ask before overwriting (-i = interactive)
cp -i file.txt /backup/file.txt
# cp: overwrite '/backup/file.txt'? y

# Show each file being copied (-v = verbose)
cp -v file.txt /backup/
# 'file.txt' -> '/backup/file.txt'

# Move with confirmation prompt
mv -i file.txt /backup/file.txt
# mv: overwrite '/backup/file.txt'? y
```

---

### Example 11 — Safe Copy — Only Copy If Source Is Newer

```bash
# Only update the destination if source is newer
cp -u file.txt /backup/file.txt

# Move with no clobber (don't overwrite existing files)
mv -n file.txt /backup/file.txt
```

---

### Example 12 — Real-World: Backup Config Before Editing

```bash
# Before editing a config file — always make a backup!
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak

# Edit the original
sudo nano /etc/nginx/nginx.conf

# If something goes wrong — restore from backup
sudo cp /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf
```

---

## 🗂️ `cp` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-r` / `-R` | **Recursive** — copy directories and their contents |
| `-p` | **Preserve** — keep permissions, timestamps, and ownership |
| `-a` | **Archive** — preserve everything (`-dR --preserve=all`) |
| `-i` | **Interactive** — prompt before overwriting |
| `-n` | **No clobber** — do not overwrite existing files |
| `-u` | **Update** — copy only if source is newer than destination |
| `-v` | **Verbose** — show each file as it is copied |
| `-b` | **Backup** — make a backup of each file that would be overwritten |
| `-f` | **Force** — remove destination if it cannot be opened |

---

## 🗂️ `mv` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-i` | **Interactive** — prompt before overwriting |
| `-n` | **No clobber** — do not overwrite existing files |
| `-u` | **Update** — move only if source is newer than destination |
| `-v` | **Verbose** — show each file as it is moved |
| `-b` | **Backup** — make a backup of each file that would be overwritten |
| `-f` | **Force** — do not prompt before overwriting |

---

## 🔄 `cp` vs `mv` — When to Use Which

| Situation | Command |
|-----------|---------|
| Backup before editing | `cp original original.bak` |
| Deploy files to production | `cp -r build/ /var/www/html/` |
| Preserve timestamps on backup | `cp -p file backup` |
| Rename a file | `mv oldname newname` |
| Organize files into folders | `mv *.jpg photos/` |
| Move application files after install | `mv /tmp/app /opt/app` |
| Safest copy (no overwrite) | `cp -n source dest` |

---

## 🚀 Full Workflow — Safe File Management

```bash
# Step 1: Back up a config file before modifying
sudo cp -p /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Step 2: Verify the backup exists
ls -la /etc/ssh/sshd_config*

# Step 3: Make your changes
sudo nano /etc/ssh/sshd_config

# Step 4: If changes went wrong, restore the backup
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config

# Step 5: Organize project files
mkdir -p /project/{src,docs,backup}
cp -r /tmp/source_code/ /project/src/
mv /tmp/old_docs/ /project/docs/
mv /tmp/archive*.tar.gz /project/backup/

# Step 6: Verify final structure
ls -laR /project/
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `cp dir/ dest/` without `-r` fails | Always use `cp -r` for directories |
| `mv` overwrites destination silently | Use `mv -i` to prompt before overwriting |
| `cp` changes timestamps | Use `cp -p` to preserve original timestamps |
| Forgetting that `mv` removes the original | If you need the original, use `cp` instead |
| `cp *.txt dest/` with no matching files | Check glob expansion with `ls *.txt` first |
| Moving to a non-existent directory | Create the directory first with `mkdir -p` |

---

## 🔑 Key Takeaways

- `cp` **duplicates** a file — the original remains; `mv` **relocates or renames** — the original is removed.
- Always use `cp -r` for copying **directories** — `cp` refuses to copy them without it.
- Use `cp -p` to **preserve timestamps and permissions** — essential for backups.
- Use `cp -a` for a complete **archive copy** of a directory tree.
- Use `mv` to **rename a file** in the same directory — it is the standard rename command in Linux.
- Always make a **backup copy** before editing important configuration files: `cp file file.bak`.
- Use `-i` (interactive) or `-n` (no clobber) to **prevent accidental overwrites**.

---

## 📚 Related Concepts

- [GNU cp Manual](https://www.gnu.org/software/coreutils/manual/html_node/cp-invocation.html)
- [GNU mv Manual](https://www.gnu.org/software/coreutils/manual/html_node/mv-invocation.html)


</details>
