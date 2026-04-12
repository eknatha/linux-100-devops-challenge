# Challenge 60: Backup Using rsync

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`rsync`** (Remote Sync) is the gold standard tool for file synchronization and backups in Linux. Unlike `cp` or `tar`, `rsync` is **incremental** — it only transfers files that have changed, making subsequent backups extremely fast and bandwidth-efficient. It works seamlessly for both **local** and **remote** backups over SSH.

---

## 🧩 Task

Backup a source directory to a backup location.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
rsync -av /source/ /backup/
```

### How it works

| Flag | Description |
|------|-------------|
| `-a` | **Archive mode** — preserves permissions, timestamps, symlinks, owner, group, and recurses into subdirectories (equivalent to `-rlptgoD`) |
| `-v` | **Verbose** — prints each file as it is transferred |
| `/source/` | Source directory — the trailing `/` means "copy the **contents** of this directory" |
| `/backup/` | Destination directory where files will be synced to |

### Trailing Slash — Critical Distinction

```bash
# Copies CONTENTS of /source/ into /backup/
rsync -av /source/ /backup/

# Copies the /source DIRECTORY ITSELF into /backup/ → creates /backup/source/
rsync -av /source /backup/
```

> ⚠️ The trailing slash on the **source** path is one of the most important details in `rsync` — it controls whether the directory itself or just its contents are copied.

---

## 📖 Extended Examples

### Example 1 — Basic Local Backup

```bash
rsync -av /source/ /backup/
```

```bash
sending incremental file list
./
file1.txt
file2.txt
logs/
logs/app.log

sent 1,024 bytes  received 42 bytes  2,132.00 bytes/sec
total size is 98,304  speedup is 93.41
```

---

### Example 2 — Dry Run (Preview Without Making Changes)

```bash
rsync -av --dry-run /source/ /backup/
```

```bash
sending incremental file list
./
file1.txt
file2.txt

sent 120 bytes  received 18 bytes  (dry run)
```

> 💡 Always test with `--dry-run` (or `-n`) before running a real backup to verify what will be transferred.

---

### Example 3 — Show Progress for Large Transfers

```bash
rsync -av --progress /source/ /backup/
```

```bash
file1.txt
    102,400 100%   48.83MB/s    0:00:00 (xfr#1, to-chk=3/5)
database.sql
  1,048,576  100%   12.45MB/s    0:00:00 (xfr#2, to-chk=2/5)
```

---

### Example 4 — Delete Files in Destination That No Longer Exist in Source

```bash
rsync -av --delete /source/ /backup/
```

> `--delete` keeps the destination a **perfect mirror** of the source. Files removed from `/source/` are also removed from `/backup/`.

> ⚠️ Use with caution — deleted files in the destination are **not recoverable** unless you have another backup.

---

### Example 5 — Remote Backup over SSH

```bash
rsync -avz -e ssh /source/ user@remote-server:/backup/
```

| Flag | Description |
|------|-------------|
| `-z` | **Compress** data during transfer — saves bandwidth |
| `-e ssh` | Use SSH as the transport protocol |

```bash
# Using a custom SSH port
rsync -avz -e "ssh -p 2222" /source/ user@remote-server:/backup/
```

---

### Example 6 — Pull Backup FROM a Remote Server

```bash
rsync -avz user@remote-server:/var/www/ /local/backup/www/
```

> `rsync` works in both directions — **push** to a remote or **pull** from a remote.

---

### Example 7 — Exclude Files and Directories

```bash
rsync -av \
  --exclude="*.log" \
  --exclude="*.tmp" \
  --exclude=".git/" \
  --exclude="node_modules/" \
  /source/ /backup/
```

```bash
# Or use an exclude file
rsync -av --exclude-from="/etc/rsync-excludes.txt" /source/ /backup/
```

```bash
# /etc/rsync-excludes.txt
*.log
*.tmp
.git/
node_modules/
__pycache__/
```

---

### Example 8 — Limit Bandwidth Usage

```bash
rsync -avz --bwlimit=5000 /source/ user@remote-server:/backup/
```

> `--bwlimit` sets the maximum transfer rate in **KB/s** — useful to avoid saturating a network connection during business hours.

---

### Example 9 — Incremental Backup with Hardlinks (Space-Efficient Snapshots)

```bash
#!/bin/bash
DATE=$(date +"%Y-%m-%d")
DEST="/backup/$DATE"
LATEST="/backup/latest"

rsync -av --delete \
  --link-dest="$LATEST" \
  /source/ "$DEST/"

# Update the 'latest' symlink to point to today's backup
ln -sfn "$DEST" "$LATEST"
```

> **`--link-dest`** creates **hardlinks** to unchanged files from the previous backup instead of copying them again. Each daily snapshot appears complete but only stores the differences — major space saving.

```bash
# Resulting structure:
/backup/
├── 2025-03-19/   ← full snapshot
├── 2025-03-20/   ← only changed files copied; rest are hardlinks
├── 2025-03-21/   ← only changed files copied; rest are hardlinks
└── latest -> /backup/2025-03-21/
```

---

### Example 10 — Full Automated Backup Script

```bash
#!/bin/bash

# ── Configuration ─────────────────────────────────────────
SOURCE="/var/www/"
DEST="/backup/www"
REMOTE_USER="backupuser"
REMOTE_HOST="backup.example.com"
REMOTE_PATH="/backups/www"
LOG="/var/log/rsync_backup.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# ── Local backup ──────────────────────────────────────────
echo "[$DATE] Starting local backup..." >> "$LOG"

rsync -av --delete \
  --exclude="*.tmp" \
  --exclude=".git/" \
  "$SOURCE" "$DEST/" >> "$LOG" 2>&1

# ── Remote backup ─────────────────────────────────────────
echo "[$DATE] Starting remote backup..." >> "$LOG"

rsync -avz --delete \
  -e "ssh -i /root/.ssh/backup_key" \
  "$SOURCE" "$REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/" >> "$LOG" 2>&1

# ── Result ───────────────────────────────────────────────
if [ $? -eq 0 ]; then
  echo "[$DATE] ✅ Backup completed successfully." >> "$LOG"
else
  echo "[$DATE] ❌ Backup failed!" >> "$LOG"
  exit 1
fi
```

---

### Example 11 — Schedule with Cron (Daily at 2 AM)

```bash
crontab -e
```

```bash
# Daily backup at 2:00 AM
0 2 * * * /usr/local/bin/backup.sh >> /var/log/rsync_backup.log 2>&1
```

---

## 🗂️ `rsync` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-a` | Archive mode (preserves everything, recurses) |
| `-v` | Verbose output |
| `-z` | Compress data during transfer |
| `-n` / `--dry-run` | Simulate the transfer without making changes |
| `--progress` | Show per-file transfer progress |
| `--delete` | Remove files in destination not present in source |
| `--exclude` | Exclude files matching a pattern |
| `--exclude-from` | Read exclusion patterns from a file |
| `--link-dest` | Create hardlinks to unchanged files from a reference dir |
| `--bwlimit=KB` | Limit transfer speed in KB/s |
| `-e ssh` | Use SSH as transport |
| `--checksum` | Compare files by checksum instead of timestamp/size |
| `--stats` | Print detailed transfer statistics |
| `-P` | Combines `--progress` and `--partial` (resume interrupted transfers) |

---

## 🔄 `rsync` vs Other Backup Tools

| Tool | Incremental | Remote | Compression | Hardlink Snapshots | Best For |
|------|------------|--------|-------------|-------------------|----------|
| `rsync` | ✅ | ✅ | ✅ | ✅ | Flexible local/remote sync |
| `tar` | ❌ | ❌ | ✅ | ❌ | One-time archives |
| `cp` | ❌ | ❌ | ❌ | ❌ | Simple local copies |
| `scp` | ❌ | ✅ | ❌ | ❌ | One-time remote copy |

---

## 🚀 Full Workflow — Set Up Automated Backups

```bash
# Step 1: Test with a dry run
rsync -av --dry-run /source/ /backup/

# Step 2: Run a real backup
rsync -av --delete /source/ /backup/

# Step 3: Verify the backup
ls -lh /backup/
diff -rq /source/ /backup/

# Step 4: Create the backup script
sudo nano /usr/local/bin/backup.sh
sudo chmod +x /usr/local/bin/backup.sh

# Step 5: Schedule with cron
crontab -e
# Add: 0 2 * * * /usr/local/bin/backup.sh

# Step 6: Monitor logs
tail -f /var/log/rsync_backup.log
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Missing trailing slash on source | `rsync -av /source/ /dest/` copies contents; without `/` it nests the dir |
| Using `--delete` without testing first | Always do a `--dry-run` before running with `--delete` |
| Forgetting `-z` for remote backups | Add `-z` to compress data and save bandwidth over SSH |
| SSH key not set up for cron | Use a dedicated SSH key pair for automated backups |
| Backup fills destination disk | Monitor destination disk separately; use `--bwlimit` and `--exclude` |
| No verification after backup | Run `diff -rq /source/ /backup/` or check rsync exit code `$?` |

---

## 🔑 Key Takeaways

- `rsync -av` is the standard incantation — `-a` preserves everything, `-v` shows progress.
- The **trailing slash** on the source path is critical — with it, contents are copied; without it, the directory itself is nested inside the destination.
- Always test with `--dry-run` before running a backup that uses `--delete`.
- Add `-z` and `-e ssh` for efficient, secure **remote backups**.
- Use `--link-dest` for **space-efficient daily snapshots** that look complete but share unchanged files.
- Combine with **cron** for automated, hands-free scheduled backups.

---

## 📚 Related Concepts

- [rsync Manual](https://linux.die.net/man/1/rsync)
- [Restic](https://restic.net/) — modern encrypted backup tool built on similar principles

</details>
