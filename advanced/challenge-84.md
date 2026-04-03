# Challenge 84: Recover Deleted Files (Basic) in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

Accidentally deleting a file is one of the most common and stressful Linux mishaps. Unlike Windows, Linux does not have a Recycle Bin — `rm` permanently removes files immediately. However, **recovery is often possible** because Linux keeps a file's data on disk until the last process with an open file descriptor releases it.

This challenge focuses on **basic recovery using `lsof`** — the most reliable method when a process still has the deleted file open. For files with no open handles, more advanced tools like `extundelete` or `testdisk` are needed.

---

## 🧩 Task

Attempt to recover a recently deleted file that may still be held open by a running process.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
lsof | grep deleted
```

### How it works

| Part | Description |
|------|-------------|
| `lsof` | **L**ist **O**pen **F**iles — shows all files currently open by any process |
| `\|` | Pipe output to grep |
| `grep deleted` | Filter to show only files that have been **deleted from the filesystem** but are still held open by a process |

### Why This Works

```
Normal file lifecycle:
  File created → File exists on disk → rm deletes it → Space freed immediately

File held open by a process:
  File created → Process opens it → rm deletes directory entry → 
  File DATA still on disk (held by open file descriptor) →
  Space freed ONLY when process closes the file (or terminates)

Recovery window:
  Between rm and the process closing the file → data is recoverable
```

---

## 🔢 Understanding `lsof` Output for Deleted Files

```bash
lsof | grep deleted
```

```
COMMAND   PID     USER   FD   TYPE DEVICE    SIZE/OFF    NODE NAME
nginx    5201  www-data   3w   REG    8,1  6442450944  123456 /var/log/nginx/access.log (deleted)
python3  4821   appuser   4r   REG    8,1   104857600  234567 /tmp/datafile.csv (deleted)
java    12345      root   7r   REG    8,1   524288000  345678 /opt/app/data.bin (deleted)
```

| Column | Description |
|--------|-------------|
| `COMMAND` | Process holding the file open |
| `PID` | Process ID |
| `USER` | Owner of the process |
| `FD` | File descriptor number — used for recovery |
| `TYPE` | File type (`REG` = regular file) |
| `SIZE/OFF` | Current file size in bytes |
| `NODE` | Inode number |
| `NAME` | Original file path + `(deleted)` marker |

---

## 📖 Extended Examples

### Example 1 — Find All Deleted Files Still Held Open

```bash
lsof | grep deleted
```

---

### Example 2 — Find Deleted Files with PID and Size

```bash
lsof | grep deleted | awk '{print $2, $9, $7, $NF}' | column -t
```

```bash
PID    FD  SIZE        STATUS
5201    3  6442450944  (deleted)
4821    4   104857600  (deleted)
```

---

### Example 3 — Recover a Deleted File via `/proc`

This is the actual recovery step. Every open file descriptor is accessible via `/proc/<PID>/fd/<FD>`:

```bash
# From lsof output, note: PID=4821, FD=4, original file=/tmp/datafile.csv

# Step 1: Confirm the file is accessible via /proc
ls -la /proc/4821/fd/4

# Step 2: Copy the file content back to a new location
cp /proc/4821/fd/4 /tmp/datafile_recovered.csv

# Step 3: Verify the recovered file
ls -lh /tmp/datafile_recovered.csv
wc -l /tmp/datafile_recovered.csv
head /tmp/datafile_recovered.csv
```

```bash
# ls output showing the deleted file is accessible:
lrwxrwxrwx 1 appuser appuser 64 Mar 21 10:00 /proc/4821/fd/4 -> /tmp/datafile.csv (deleted)
```

> 💡 `/proc/<PID>/fd/<FD>` is a **symbolic link to the open file** — even though the directory entry is gone, the data is still accessible through this path.

---

### Example 4 — Recover a Large Log File Deleted While nginx Was Writing

A very common real-world scenario — a log file is deleted to free up space, but nginx is still writing to it and disk space is not freed:

```bash
# Step 1: Find the deleted log in lsof
lsof | grep deleted | grep nginx
```

```bash
nginx  5201 www-data  3w  REG  8,1  6442450944  123456 /var/log/nginx/access.log (deleted)
#       ^^^            ^
#       PID            FD number
```

```bash
# Step 2: Recover the file content
sudo cp /proc/5201/fd/3 /var/log/nginx/access.log.recovered

# Step 3: Verify recovery
ls -lh /var/log/nginx/access.log.recovered

# Step 4: Free the disk space properly — reload nginx so it reopens the log file
sudo systemctl reload nginx    # nginx reopens logs on SIGUSR1
# OR
sudo kill -USR1 $(cat /run/nginx.pid)

# Step 5: Verify disk space is now freed
df -h
```

---

### Example 5 — Find Deleted Files Consuming Disk Space

The classic "disk full but nothing to delete" problem — deleted files held open by processes:

```bash
# Find deleted files and their sizes, sorted by size
lsof | grep deleted | awk '{print $7, $2, $1, $NF}' | sort -rn | head -10
```

```bash
6442450944  5201  nginx   /var/log/nginx/access.log (deleted)
524288000  12345  java    /opt/app/data.bin (deleted)
104857600   4821  python3 /tmp/datafile.csv (deleted)
```

> 💡 If `df -h` shows a disk as nearly full but `du -sh /*` shows much less space, **deleted-but-open files** are almost certainly the cause.

---

### Example 6 — Recover Deleted Files for a Specific Process

```bash
# Find all deleted files for a specific service (e.g., mysql)
lsof -p $(pgrep mysql) | grep deleted

# List all open file descriptors for a PID
ls -la /proc/4821/fd/

# Recover all deleted files for a process
PID=4821
for fd in $(ls /proc/$PID/fd/); do
  target=$(readlink /proc/$PID/fd/$fd 2>/dev/null)
  if echo "$target" | grep -q "(deleted)"; then
    filename=$(basename "$target" | sed 's/ (deleted)//')
    echo "Recovering: $target → /tmp/recovered_${filename}"
    cp /proc/$PID/fd/$fd /tmp/recovered_${filename} 2>/dev/null && echo "✅ Recovered" || echo "❌ Failed"
  fi
done
```

---

### Example 7 — Check Inode Status Directly

```bash
# Find the inode of a deleted file from lsof output (NODE column)
# Example: NODE = 123456

# Check if the inode is still referenced
sudo debugfs -R "stat <123456>" /dev/sda1 2>/dev/null

# List all unlinked (deleted but open) inodes
sudo debugfs -R "ls -l" /dev/sda1 2>/dev/null | grep "^0 "
```

---

### Example 8 — When `lsof` Shows Nothing — Advanced Recovery with `extundelete`

If no process has the file open, `lsof` will not help. For ext3/ext4 filesystems:

```bash
# Install extundelete
sudo apt install extundelete    # Debian/Ubuntu
sudo dnf install extundelete    # RHEL

# IMPORTANT: Unmount or remount read-only to prevent overwriting data
sudo mount -o remount,ro /dev/sda1

# Recover a specific file by path
sudo extundelete /dev/sda1 --restore-file path/to/deleted/file.txt

# Recover all deleted files
sudo extundelete /dev/sda1 --restore-all

# Recovered files appear in ./RECOVERED_FILES/
ls -la RECOVERED_FILES/
```

> ⚠️ Stop writing to the filesystem immediately after accidental deletion — every new write risks overwriting the deleted file's data blocks.

---

### Example 9 — TestDisk for Filesystem-Level Recovery

```bash
# Install testdisk
sudo apt install testdisk    # Debian/Ubuntu
sudo dnf install testdisk    # RHEL

# Launch testdisk (interactive)
sudo testdisk /dev/sda

# Or use photorec for individual file recovery (included with testdisk)
sudo photorec /dev/sda
```

> `testdisk` works by scanning the raw disk for filesystem structures and file signatures — it can recover files even from formatted or corrupted partitions.

---

### Example 10 — Prevent Future Accidental Deletions

```bash
# Option 1: Alias rm to ask for confirmation
alias rm='rm -i'
echo "alias rm='rm -i'" >> ~/.bashrc

# Option 2: Use trash-cli instead of rm
sudo apt install trash-cli
alias rm='trash-put'

# Option 3: Use safe-rm (blocks /etc, /usr, etc.)
sudo apt install safe-rm

# Option 4: Take backups before bulk deletions
cp -r /important/dir /backup/dir_$(date +%Y%m%d) && rm -rf /important/dir

# Option 5: Use --dry-run or -n with deletion commands
rsync --dry-run --delete /source/ /dest/
```

---

## 🗂️ Recovery Methods — When to Use Each

| Situation | Best Method |
|-----------|-------------|
| File deleted, process still has it open | `lsof \| grep deleted` → `cp /proc/PID/fd/FD` |
| Log file deleted, service still running | `lsof \| grep deleted` → recover → reload service |
| Disk full due to deleted-but-open file | `lsof \| grep deleted` → reload the process holding it |
| File deleted, no open handles, ext3/ext4 | `extundelete` — act fast, minimize writes |
| File deleted, no open handles, any FS | `testdisk` / `photorec` — forensic scan |
| Entire partition accidentally formatted | `testdisk` partition recovery |
| Regular important files | **Backups** — `rsync`, `tar`, cloud storage |

---

## ⚠️ Important Warnings

```
⚠️  STOP WRITING TO THE FILESYSTEM IMMEDIATELY after accidental deletion
    Every write risks overwriting the deleted file's data blocks

⚠️  The /proc/PID/fd recovery method ONLY works while the process
    that has the file open is STILL RUNNING — if it closes, data is gone

⚠️  Recover to a DIFFERENT filesystem than the one you're recovering from
    Recovering to the same disk can overwrite data you're trying to save

⚠️  lsof recovery has a time window — it ends the moment the process
    closes the file descriptor or the process terminates
```

---

## 🚀 Full Workflow — Recover a Deleted File

```bash
# Step 1: Don't panic — stop writing to the affected filesystem
sync

# Step 2: Check if any process has the file open
lsof | grep deleted

# Step 3: Identify the PID and FD from the output
# Example: nginx PID=5201, FD=3

# Step 4: Verify the file is accessible
ls -la /proc/5201/fd/3

# Step 5: Recover to a safe location (different filesystem if possible)
cp /proc/5201/fd/3 /backup/recovered_access.log

# Step 6: Verify the recovered file
ls -lh /backup/recovered_access.log
md5sum /backup/recovered_access.log   # Note checksum for integrity

# Step 7: Free disk space if the deleted file was filling the disk
# Reload/restart the service so it closes and reopens its log files
sudo systemctl reload nginx

# Step 8: Verify disk space is freed
df -h

# Step 9: If no process had it open — use extundelete (unmount first)
sudo umount /dev/sda1
sudo extundelete /dev/sda1 --restore-all

# Step 10: Set up backups to prevent future data loss
crontab -e
# 0 2 * * * rsync -av /important/ /backup/
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `lsof` shows nothing but disk is full | Check if the process already closed the file — space freed next |
| Recovering to same disk risks overwrite | Always recover to a **different** filesystem or mount point |
| `cp /proc/PID/fd/FD` fails with permission | Use `sudo` — process may be owned by root or another user |
| Process terminated before recovery | Data is gone via `lsof` method — try `extundelete` |
| `extundelete` finds nothing | Filesystem may use journaling that overwrote data — try `photorec` |
| Recovery file is empty | File descriptor may point to a pipe or socket, not a regular file |

---

## 🔑 Key Takeaways

- `lsof | grep deleted` finds files that have been **deleted from the directory but whose data is still on disk** because a process has them open.
- Recovery is done by **copying from `/proc/<PID>/fd/<FD>`** — a kernel-provided path to the open file.
- This **only works while the process is still running** — the recovery window closes the moment the process terminates or closes the file.
- If the disk appears full but `du` shows less space, **deleted-but-open files** are almost certainly the cause — reload the holding process to free the space.
- For files with **no open handles**, use `extundelete` (ext3/ext4) or `testdisk` — and stop writing to the filesystem immediately.
- The best recovery strategy is **prevention** — regular backups with `rsync`, `tar`, or cloud storage.

---

## 📚 Related Concepts

- [Challenge 58 — Archive Logs Automatically]
- [Challenge 60 — Backup Using rsync]
- [Challenge 73 — Disk Space Issue]
- [extundelete](http://extundelete.sourceforge.net/) — ext3/ext4 file recovery tool
- [testdisk / photorec](https://www.cgsecurity.org/wiki/TestDisk) — powerful open-source data recovery
- [GNU lsof Manual](https://linux.die.net/man/8/lsof)


</details>
