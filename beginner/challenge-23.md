# Challenge 23: Compress Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**File compression** reduces file size by encoding data more efficiently — saving disk space, reducing backup time, and speeding up file transfers. Linux provides several compression tools, each with different speed vs. compression ratio trade-offs.

The most common tools:
- **`gzip`** — fast, widely used, produces `.gz` files
- **`bzip2`** — slower but better compression, produces `.bz2` files
- **`xz`** — best compression ratio, slowest, produces `.xz` files
- **`zip`** — cross-platform format compatible with Windows
- **`tar`** — bundles multiple files, then compresses (`.tar.gz`, `.tar.bz2`)

---

## 🧩 Task

Compress a file to reduce its size.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Compress with gzip (most common)
gzip filename.txt

# Compress and keep the original
gzip -k filename.txt

# Compress a directory (bundle first, then compress)
tar -czf archive.tar.gz directory/
```

### How it works

| Command | Description |
|---------|-------------|
| `gzip filename.txt` | Compress file — replaces it with `filename.txt.gz` |
| `gzip -k filename.txt` | Compress and **keep** the original |
| `tar -czf archive.tar.gz dir/` | Bundle a directory and compress with gzip |

---

## 🔢 Compression Formats Comparison

| Format | Extension | Tool | Speed | Ratio | Cross-platform |
|--------|-----------|------|-------|-------|----------------|
| gzip | `.gz` | `gzip` | ⚡ Fast | Medium | ✅ |
| bzip2 | `.bz2` | `bzip2` | Medium | Good | ✅ |
| xz | `.xz` | `xz` | 🐢 Slow | Best | ✅ |
| zip | `.zip` | `zip` | Fast | Medium | ✅ Windows |
| lz4 | `.lz4` | `lz4` | ⚡⚡ Fastest | Low | Limited |

---

## 📖 Extended Examples

### Example 1 — Compress with `gzip`

```bash
# Compress a file (original is replaced by .gz)
gzip filename.txt

# Verify
ls -lh filename.txt.gz
```

```bash
# Before:
-rw-r--r-- 1 devuser devuser  10M Mar 21 10:00 filename.txt

# After:
-rw-r--r-- 1 devuser devuser 2.1M Mar 21 10:00 filename.txt.gz
```

> ⚠️ By default, `gzip` **replaces** the original file. Use `-k` to keep it.

---

### Example 2 — Keep the Original with `-k`

```bash
gzip -k filename.txt

# Both files exist
ls -lh filename.txt filename.txt.gz
```

```bash
-rw-r--r-- 1 devuser devuser  10M Mar 21 10:00 filename.txt
-rw-r--r-- 1 devuser devuser 2.1M Mar 21 10:00 filename.txt.gz
```

---

### Example 3 — Decompress a `.gz` File

```bash
# Decompress (replaces .gz with original)
gzip -d filename.txt.gz

# Alternative command
gunzip filename.txt.gz

# Decompress and keep the .gz
gzip -dk filename.txt.gz
```

---

### Example 4 — Set Compression Level

```bash
# Fastest compression (level 1 — largest output)
gzip -1 filename.txt

# Best compression (level 9 — smallest output, slowest)
gzip -9 filename.txt

# Default is level 6 (balanced)
gzip filename.txt
```

| Level | Speed | Size |
|-------|-------|------|
| `-1` | ⚡ Fastest | Largest |
| `-6` | Medium | Medium (default) |
| `-9` | 🐢 Slowest | Smallest |

---

### Example 5 — Compress with `bzip2`

```bash
# Compress (higher compression ratio than gzip)
bzip2 filename.txt
# Creates: filename.txt.bz2

# Keep original
bzip2 -k filename.txt

# Decompress
bzip2 -d filename.txt.bz2
# or
bunzip2 filename.txt.bz2
```

---

### Example 6 — Compress with `xz` (Best Ratio)

```bash
# Best compression ratio
xz filename.txt
# Creates: filename.txt.xz

# Keep original
xz -k filename.txt

# Decompress
xz -d filename.txt.xz
# or
unxz filename.txt.xz
```

---

### Example 7 — Create a `.zip` Archive (Cross-Platform)

```bash
# Compress a single file
zip archive.zip filename.txt

# Compress multiple files
zip archive.zip file1.txt file2.txt file3.txt

# Compress an entire directory recursively
zip -r archive.zip mydir/

# Decompress
unzip archive.zip

# List zip contents without extracting
unzip -l archive.zip
```

---

### Example 8 — Bundle and Compress with `tar`

`tar` alone just bundles files (no compression). Combine with compression flags:

```bash
# Bundle + gzip compress
tar -czf archive.tar.gz directory/

# Bundle + bzip2 compress
tar -cjf archive.tar.bz2 directory/

# Bundle + xz compress
tar -cJf archive.tar.xz directory/

# Decompress and extract
tar -xzf archive.tar.gz         # gzip
tar -xjf archive.tar.bz2        # bzip2
tar -xJf archive.tar.xz         # xz

# List contents without extracting
tar -tzf archive.tar.gz
```

| `tar` Flag | Description |
|------------|-------------|
| `-c` | **Create** archive |
| `-x` | **Extract** archive |
| `-t` | **List** contents |
| `-z` | Use **gzip** compression |
| `-j` | Use **bzip2** compression |
| `-J` | Use **xz** compression |
| `-f file` | Specify archive **filename** |
| `-v` | **Verbose** output |

---

### Example 9 — Compress Multiple Files at Once

```bash
# Compress all .log files individually
gzip *.log

# Result: app.log.gz, error.log.gz, access.log.gz

# Bundle all .log files into one compressed archive
tar -czf logs.tar.gz *.log

# Compress all files in a directory
gzip /var/log/*.log
```

---

### Example 10 — View Compression Ratio

```bash
# Show compression ratio and stats
gzip -v filename.txt
```

```bash
filename.txt:     79.4% -- replaced with filename.txt.gz
```

```bash
# Check sizes before and after
ls -lh filename.txt
gzip -k filename.txt
ls -lh filename.txt.gz
```

---

### Example 11 — Read Compressed Files Without Decompressing

```bash
# View a .gz file without decompressing
zcat filename.txt.gz

# Page through a .gz file
zless filename.txt.gz

# Search inside a .gz file
zgrep "error" filename.txt.gz

# View first 10 lines of .gz file
zcat filename.txt.gz | head -10
```

---

### Example 12 — Create a Dated Backup Archive

```bash
#!/bin/bash
# Create a compressed backup with today's date

SOURCE="/var/www/myapp"
BACKUP_DIR="/backup"
DATE=$(date +"%Y-%m-%d")
ARCHIVE="$BACKUP_DIR/myapp_$DATE.tar.gz"

# Create the compressed archive
tar -czf "$ARCHIVE" "$SOURCE"

# Verify
ls -lh "$ARCHIVE"
echo "Backup created: $ARCHIVE"
```

```bash
# Output:
-rw-r--r-- 1 root root 45M Mar 21 10:00 /backup/myapp_2025-03-21.tar.gz
Backup created: /backup/myapp_2025-03-21.tar.gz
```

---

## 🗂️ Compression Commands — Quick Reference

### `gzip`

| Command | Description |
|---------|-------------|
| `gzip file` | Compress (replaces original) |
| `gzip -k file` | Compress, keep original |
| `gzip -d file.gz` | Decompress |
| `gzip -9 file` | Best compression |
| `gzip -1 file` | Fastest compression |
| `gzip -v file` | Verbose — show compression ratio |
| `gunzip file.gz` | Decompress (alias) |

### `tar`

| Command | Description |
|---------|-------------|
| `tar -czf out.tar.gz dir/` | Bundle + gzip compress |
| `tar -cjf out.tar.bz2 dir/` | Bundle + bzip2 compress |
| `tar -cJf out.tar.xz dir/` | Bundle + xz compress |
| `tar -xzf file.tar.gz` | Extract gzip archive |
| `tar -tzf file.tar.gz` | List contents |
| `tar -xzf file.tar.gz -C /dest/` | Extract to specific directory |

### `zip`

| Command | Description |
|---------|-------------|
| `zip archive.zip file` | Compress a file |
| `zip -r archive.zip dir/` | Compress a directory |
| `unzip archive.zip` | Decompress |
| `unzip -l archive.zip` | List contents |
| `unzip archive.zip -d /dest/` | Extract to specific directory |

---

## 🚀 Full Workflow — Compress and Verify a Backup

```bash
# Step 1: Check original size
du -sh /var/www/myapp/

# Step 2: Create a compressed archive
tar -czf /backup/myapp_$(date +%Y-%m-%d).tar.gz /var/www/myapp/

# Step 3: Verify the archive was created
ls -lh /backup/myapp_*.tar.gz

# Step 4: Test archive integrity
gzip -t /backup/myapp_*.tar.gz && echo "✅ Archive is valid"

# Step 5: List archive contents
tar -tzf /backup/myapp_*.tar.gz | head -20

# Step 6: Compare sizes
echo "Original: $(du -sh /var/www/myapp/ | cut -f1)"
echo "Compressed: $(ls -lh /backup/myapp_*.tar.gz | awk '{print $5}')"
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `gzip` deletes the original | Use `-k` to keep original |
| `tar -czf` with wrong flag order | `-f` must come last before the filename |
| Compressing already-compressed files | Little to no savings — avoid double-compressing `.jpg`, `.mp4` |
| No space left during compression | Ensure enough free disk for both original and compressed |
| `zip` doesn't preserve Linux permissions | Use `tar` for Linux backups — `zip` loses permissions |

---

## 🔑 Key Takeaways

- `gzip` is the most common compression tool — fast and widely supported.
- `gzip` **replaces** the original file by default — use `-k` to keep the original.
- `tar -czf archive.tar.gz directory/` is the standard way to compress an **entire directory**.
- Use `bzip2` or `xz` for **better compression** when speed is less important.
- Use `zip` for **cross-platform archives** that need to be shared with Windows users.
- **Never compress already-compressed files** (`.jpg`, `.mp4`, `.zip`) — it wastes time with no benefit.
- Always **verify** archives after creation with `gzip -t` or `tar -t`.

---

## 📚 Related Concepts


- [GNU gzip Manual](https://www.gnu.org/software/gzip/manual/)
- [GNU tar Manual](https://www.gnu.org/software/tar/manual/)

</details>
