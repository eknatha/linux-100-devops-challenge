# Challenge 24: Extract Compressed Files in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Extracting files** is the reverse of compression — it restores the original files from a compressed archive. Linux supports many compression formats, and knowing how to extract each one is essential for installing software, restoring backups, and working with downloaded packages.

The most common archive formats you will encounter:
- **`.tar.gz`** / **`.tgz`** — the most common Linux archive format
- **`.tar.bz2`** — better compression, slower
- **`.tar.xz`** — best compression ratio
- **`.gz`** — single-file gzip compression
- **`.zip`** — cross-platform, common on Windows
- **`.bz2`** — single-file bzip2 compression
- **`.xz`** — single-file xz compression

---

## 🧩 Task

Extract a compressed archive file.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Extract a .tar.gz archive
tar -xzf archive.tar.gz

# Extract a .zip file
unzip archive.zip

# Extract a .gz file (single file)
gunzip filename.txt.gz
```

### How it works

| Command | Description |
|---------|-------------|
| `tar -xzf archive.tar.gz` | **Extract** gzip-compressed tar archive |
| `tar -xjf archive.tar.bz2` | **Extract** bzip2-compressed tar archive |
| `tar -xJf archive.tar.xz` | **Extract** xz-compressed tar archive |
| `unzip archive.zip` | **Extract** a zip archive |
| `gunzip file.gz` | **Decompress** a single gzip file |

---

## 🔢 `tar` Extract Flags

```
tar  -x  z  f  archive.tar.gz
      │  │  │       │
      │  │  │       └── Archive filename
      │  │  └────────── -f = specify filename (must precede the filename)
      │  └───────────── -z = use gzip decompression
      └──────────────── -x = extract
```

| Flag | Description |
|------|-------------|
| `-x` | **Extract** files from archive |
| `-z` | Decompress with **gzip** (`.tar.gz`) |
| `-j` | Decompress with **bzip2** (`.tar.bz2`) |
| `-J` | Decompress with **xz** (`.tar.xz`) |
| `-f FILE` | Specify the **archive filename** |
| `-v` | **Verbose** — list files as they are extracted |
| `-C DIR` | Extract to a **specific directory** |
| `-t` | **List** contents without extracting |

---

## 📖 Extended Examples

### Example 1 — Extract a `.tar.gz` Archive

```bash
# Extract to the current directory
tar -xzf archive.tar.gz

# With verbose output — see each file extracted
tar -xzvf archive.tar.gz
```

```bash
# Verbose output:
myapp/
myapp/src/
myapp/src/main.py
myapp/config/
myapp/config/settings.conf
myapp/README.md
```

---

### Example 2 — Extract to a Specific Directory

```bash
# Extract to /opt/myapp/
tar -xzf archive.tar.gz -C /opt/myapp/

# Extract to a new directory (create it first)
mkdir -p /opt/myapp
tar -xzf archive.tar.gz -C /opt/myapp/

# Extract zip to a specific directory
unzip archive.zip -d /opt/myapp/
```

---

### Example 3 — Extract `.tar.bz2` and `.tar.xz`

```bash
# Extract bzip2-compressed archive
tar -xjf archive.tar.bz2

# Extract xz-compressed archive
tar -xJf archive.tar.xz

# With verbose output
tar -xjvf archive.tar.bz2
tar -xJvf archive.tar.xz
```

---

### Example 4 — List Archive Contents Without Extracting

```bash
# List .tar.gz contents
tar -tzf archive.tar.gz

# List .tar.bz2 contents
tar -tjf archive.tar.bz2

# List .zip contents
unzip -l archive.zip
```

```bash
# tar -tzf output:
myapp/
myapp/src/main.py
myapp/config/settings.conf
myapp/README.md
```

> 💡 Always **list contents first** before extracting — especially with untrusted archives — to see what files will be created and where.

---

### Example 5 — Extract a Single `.gz` File

```bash
# Decompress .gz (replaces .gz with original file)
gunzip filename.txt.gz
# or
gzip -d filename.txt.gz

# Decompress and keep the .gz
gunzip -k filename.txt.gz

# View without decompressing
zcat filename.txt.gz
```

---

### Example 6 — Extract `.bz2` and `.xz` Single Files

```bash
# Decompress .bz2
bunzip2 filename.txt.bz2
# or
bzip2 -d filename.txt.bz2

# Decompress .xz
unxz filename.txt.xz
# or
xz -d filename.txt.xz

# Keep the original compressed file
bunzip2 -k filename.txt.bz2
unxz -k filename.txt.xz
```

---

### Example 7 — Extract `.zip` Files

```bash
# Extract to current directory
unzip archive.zip

# Extract to a specific directory
unzip archive.zip -d /opt/extracted/

# Extract only a specific file from the archive
unzip archive.zip specific_file.txt

# List contents without extracting
unzip -l archive.zip

# Test integrity without extracting
unzip -t archive.zip
```

---

### Example 8 — Auto-Detect and Extract Any Format

```bash
# Using the file command to detect format
file archive.tar.gz
# archive.tar.gz: gzip compressed data

# Smart extract script
extract() {
  case "$1" in
    *.tar.gz|*.tgz)    tar -xzf "$1" ;;
    *.tar.bz2|*.tbz2)  tar -xjf "$1" ;;
    *.tar.xz|*.txz)    tar -xJf "$1" ;;
    *.tar)             tar -xf  "$1" ;;
    *.gz)              gunzip   "$1" ;;
    *.bz2)             bunzip2  "$1" ;;
    *.xz)              unxz     "$1" ;;
    *.zip)             unzip    "$1" ;;
    *.rar)             unrar x  "$1" ;;
    *.7z)              7z x     "$1" ;;
    *)    echo "Cannot extract '$1' — unknown format" ;;
  esac
}

# Use it:
extract archive.tar.gz
extract myfile.zip
```

> 💡 Add this function to `~/.bashrc` to use `extract` as a universal extract command.

---

### Example 9 — Extract Only Specific Files from an Archive

```bash
# Extract only one file
tar -xzf archive.tar.gz myapp/config/settings.conf

# Extract only files matching a pattern
tar -xzf archive.tar.gz --wildcards "*.conf"

# Extract only a subdirectory
tar -xzf archive.tar.gz myapp/config/

# From a zip — extract specific files
unzip archive.zip "*.conf"
```

---

### Example 10 — Verify Archive Integrity Before Extracting

```bash
# Test gzip file integrity
gzip -t archive.tar.gz && echo "✅ Archive is valid"

# Test zip integrity
unzip -t archive.zip

# Test bzip2 integrity
bzip2 -t archive.tar.bz2

# List tar contents (also validates the archive)
tar -tzf archive.tar.gz > /dev/null && echo "✅ Archive is readable"
```

---

### Example 11 — Extract and Install a Software Package

A common real-world pattern — downloading and installing software from a tarball:

```bash
# Step 1: Download the archive
wget https://example.com/software-1.0.tar.gz

# Step 2: List contents first
tar -tzf software-1.0.tar.gz | head -10

# Step 3: Extract to /opt
sudo tar -xzf software-1.0.tar.gz -C /opt/

# Step 4: Navigate to extracted directory
cd /opt/software-1.0/

# Step 5: Install
./configure && make && sudo make install
```

---

### Example 12 — Strip Leading Directory When Extracting

Archives often contain a top-level directory like `myapp-1.0/`. Use `--strip-components` to remove it:

```bash
# Archive structure:
# myapp-1.0/
# myapp-1.0/src/
# myapp-1.0/README.md

# Without --strip-components: creates myapp-1.0/ subdirectory
tar -xzf myapp-1.0.tar.gz

# With --strip-components=1: extracts directly to current directory
tar -xzf myapp-1.0.tar.gz --strip-components=1
# src/
# README.md
```

---

## 🗂️ Format to Extraction Command — Quick Reference

| Format | Extension | Extraction Command |
|--------|-----------|-------------------|
| gzip tar | `.tar.gz` / `.tgz` | `tar -xzf file.tar.gz` |
| bzip2 tar | `.tar.bz2` / `.tbz2` | `tar -xjf file.tar.bz2` |
| xz tar | `.tar.xz` / `.txz` | `tar -xJf file.tar.xz` |
| tar only | `.tar` | `tar -xf file.tar` |
| gzip single | `.gz` | `gunzip file.gz` |
| bzip2 single | `.bz2` | `bunzip2 file.bz2` |
| xz single | `.xz` | `unxz file.xz` |
| zip | `.zip` | `unzip file.zip` |
| rar | `.rar` | `unrar x file.rar` |
| 7-zip | `.7z` | `7z x file.7z` |

---

## 🚀 Full Workflow — Download, Verify, and Extract

```bash
# Step 1: Download the archive
wget https://example.com/myapp-1.0.tar.gz

# Step 2: Check the file type and size
file myapp-1.0.tar.gz
ls -lh myapp-1.0.tar.gz

# Step 3: Verify integrity
gzip -t myapp-1.0.tar.gz && echo "✅ Valid"

# Step 4: Preview contents before extracting
tar -tzf myapp-1.0.tar.gz | head -20

# Step 5: Extract to a specific location
tar -xzf myapp-1.0.tar.gz -C /opt/

# Step 6: Verify extraction
ls -la /opt/myapp-1.0/

# Step 7: Clean up the archive (optional)
rm myapp-1.0.tar.gz
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Extracting to wrong directory | Always use `-C /target/dir/` or `cd` first |
| Archive creates many files in current dir | List with `-t` first to check structure |
| `tar: Unexpected EOF` error | Archive is corrupted or incomplete — re-download |
| `unzip` not installed | `sudo apt install unzip` or `sudo dnf install unzip` |
| `.rar` not supported by default | Install `unrar`: `sudo apt install unrar` |
| Forgetting `-z`, `-j`, or `-J` | Use `file archive.tar.gz` to detect format |

---

## 🔑 Key Takeaways

- `tar -xzf archive.tar.gz` extracts a `.tar.gz` archive — the most common format in Linux.
- Always use **`-C /destination/`** to extract to a specific directory instead of the current one.
- Use **`tar -tzf archive.tar.gz`** to **list contents before extracting** — avoids surprises with file locations.
- Use **`--strip-components=1`** to remove the top-level directory from archives that contain one.
- `gunzip` / `bunzip2` / `unxz` handle **single-file** decompression; `tar` handles **bundled archives**.
- Always **verify archive integrity** with `gzip -t` or `unzip -t` before extracting important backups.
- The universal `extract()` function in `~/.bashrc` handles any format automatically.

---

## 📚 Related Concepts


- [GNU tar Manual](https://www.gnu.org/software/tar/manual/)
- [GNU gzip Manual](https://www.gnu.org/software/gzip/manual/)

</details>
