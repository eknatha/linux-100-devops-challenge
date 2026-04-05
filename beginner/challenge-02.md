# Challenge 02: Create and Delete Directories in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

Creating and deleting directories are among the most fundamental Linux operations. Whether you are organizing project files, setting up application directories, or cleaning up old data — `mkdir` and `rmdir`/`rm -rf` are the essential tools.

Understanding the difference between **safe removal** (`rmdir` — only removes empty directories) and **force removal** (`rm -rf` — removes everything recursively) is critical to avoiding accidental data loss.

---

## 🧩 Task

Create a new directory and delete an existing directory.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Create a directory
mkdir mydir

# Delete an empty directory
rmdir mydir

# Delete a directory and all its contents
rm -rf mydir
```

### How it works

| Command | Description |
|---------|-------------|
| `mkdir mydir` | **M**a**k**e **dir**ectory — creates a new empty directory named `mydir` |
| `rmdir mydir` | **R**e**m**ove **dir**ectory — removes an **empty** directory only |
| `rm -rf mydir` | **R**e**m**ove **r**ecursively and **f**orcefully — removes directory and all contents |

---

## 🔢 `mkdir` vs `rmdir` vs `rm -rf`

```
Directory: mydir/
├── file1.txt
├── subdir/
│   └── file2.txt

rmdir mydir    → ❌ Fails — directory is not empty
rm -rf mydir   → ✅ Removes everything including contents
```

---

## 📖 Extended Examples

### Example 1 — Create a Single Directory

```bash
mkdir mydir
```

```bash
# Verify it was created
ls -la | grep mydir
```

```bash
drwxr-xr-x 2 devuser devuser 4096 Mar 21 10:00 mydir
```

---

### Example 2 — Create Multiple Directories at Once

```bash
# Create multiple directories in one command
mkdir dir1 dir2 dir3

# Verify
ls -la
```

```bash
drwxr-xr-x 2 devuser devuser 4096 Mar 21 10:00 dir1
drwxr-xr-x 2 devuser devuser 4096 Mar 21 10:00 dir2
drwxr-xr-x 2 devuser devuser 4096 Mar 21 10:00 dir3
```

---

### Example 3 — Create Nested Directories with `-p`

Without `-p`, `mkdir` fails if a parent directory doesn't exist. The `-p` flag creates the full path:

```bash
# Without -p — fails if /projects doesn't exist
mkdir /projects/myapp/src     # ❌ Error: No such file or directory

# With -p — creates the entire path
mkdir -p /projects/myapp/src  # ✅ Creates all missing parent directories
```

```bash
# Create a complex nested structure in one command
mkdir -p myproject/{src,tests,docs,config}

# Result:
tree myproject/
myproject/
├── config/
├── docs/
├── src/
└── tests/
```

> 💡 The `{src,tests,docs,config}` syntax is **brace expansion** — it creates all listed subdirectories at once.

---

### Example 4 — Create a Directory with Specific Permissions

```bash
# Create directory with custom permissions (e.g., 755)
mkdir -m 755 publicdir

# Create directory readable only by owner (700)
mkdir -m 700 privatedir

# Verify permissions
ls -la
```

```bash
drwxr-xr-x 2 devuser devuser 4096 Mar 21 10:00 publicdir    ← 755
drwx------ 2 devuser devuser 4096 Mar 21 10:00 privatedir   ← 700
```

---

### Example 5 — Verbose Mode — See What's Being Created

```bash
# -v prints each directory as it is created
mkdir -pv /projects/myapp/src/utils
```

```bash
mkdir: created directory '/projects'
mkdir: created directory '/projects/myapp'
mkdir: created directory '/projects/myapp/src'
mkdir: created directory '/projects/myapp/src/utils'
```

---

### Example 6 — Remove an Empty Directory

```bash
rmdir mydir
```

```bash
# If directory is NOT empty, rmdir fails safely:
rmdir mydir
# rmdir: failed to remove 'mydir': Directory not empty
```

> ✅ `rmdir` is the **safe choice** — it refuses to delete non-empty directories, protecting you from accidental data loss.

---

### Example 7 — Remove Multiple Empty Directories

```bash
rmdir dir1 dir2 dir3
```

```bash
# Remove an entire tree of empty directories
rmdir -p a/b/c/d
# Removes d, then c, then b, then a — only if all are empty
```

---

### Example 8 — Remove a Directory and All Its Contents

```bash
# Remove directory and everything inside it
rm -rf mydir
```

> ⚠️ **There is no undo.** `rm -rf` permanently deletes everything. Always double-check the path before running.

```bash
# Safe practice: list before deleting
ls -la mydir/
rm -rf mydir
```

---

### Example 9 — Remove Multiple Directories with `rm -rf`

```bash
# Remove multiple directories at once
rm -rf dir1 dir2 dir3

# Remove all test directories matching a pattern
rm -rf test_*/

# Remove old dated directories
rm -rf logs_2024-01/ logs_2024-02/ logs_2024-03/
```

---

### Example 10 — Safely Delete with Confirmation (`-i`)

```bash
# Ask for confirmation before each deletion
rm -ri mydir
```

```bash
rm: descend into directory 'mydir'? y
rm: remove regular file 'mydir/file1.txt'? y
rm: remove directory 'mydir'? y
```

> 💡 Use `-ri` (recursive + interactive) when you want to double-check what is being deleted — especially useful before running bulk deletions.

---

### Example 11 — Create a Temporary Directory

```bash
# Create a unique temporary directory (secure — prevents name collisions)
TMPDIR=$(mktemp -d)
echo "Created: $TMPDIR"

# Use it
cp important_file.txt "$TMPDIR/"

# Clean up when done
rm -rf "$TMPDIR"
```

```bash
Created: /tmp/tmp.aB3xKz8f
```

> 💡 `mktemp -d` creates a **guaranteed-unique** temporary directory — essential for scripts that need a safe working space.

---

### Example 12 — Create a Dated Archive Directory

```bash
# Create a directory with today's date in the name
mkdir "backup_$(date +%Y-%m-%d)"
```

```bash
# Result:
backup_2025-03-21/
```

```bash
# Create a full timestamped directory
mkdir "logs_$(date +%Y%m%d_%H%M%S)"

# Result:
logs_20250321_103522/
```

---

## 🗂️ `mkdir` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-p` | **Parents** — create parent directories as needed; no error if directory already exists |
| `-m MODE` | Set file **mode** (permissions) on creation |
| `-v` | **Verbose** — print each directory as it is created |

## 🗂️ `rmdir` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-p` | Remove **parent** directories too (if they become empty) |
| `-v` | **Verbose** — print each directory as it is removed |

## 🗂️ `rm` Flags for Directories — Quick Reference

| Flag | Description |
|------|-------------|
| `-r` | **Recursive** — required to remove directories |
| `-f` | **Force** — suppress errors and prompts |
| `-i` | **Interactive** — ask before each deletion |
| `-v` | **Verbose** — print each file as it is removed |

---

## 🔄 Choosing the Right Command

| Situation | Command | Safety |
|-----------|---------|--------|
| Create a single directory | `mkdir dirname` | ✅ |
| Create nested directories | `mkdir -p a/b/c` | ✅ |
| Create multiple directories | `mkdir dir1 dir2 dir3` | ✅ |
| Remove empty directory | `rmdir dirname` | ✅ Safest |
| Remove non-empty directory | `rm -rf dirname` | ⚠️ Permanent |
| Remove with confirmation | `rm -ri dirname` | ✅ Safer |
| Remove multiple directories | `rm -rf dir1 dir2` | ⚠️ Permanent |

---

## 🚀 Full Workflow — Set Up a Project Structure

```bash
# Step 1: Create the project root and all subdirectories
mkdir -pv myproject/{src,tests,docs,config,logs,scripts}

# Step 2: Verify the structure
tree myproject/

# Step 3: Set appropriate permissions
chmod 755 myproject/
chmod 700 myproject/config/    # Sensitive configs
chmod 755 myproject/logs/

# Step 4: Create placeholder files
touch myproject/src/.gitkeep
touch myproject/tests/.gitkeep

# Step 5: List the final structure
ls -laR myproject/

# Step 6: Clean up when no longer needed
rm -rf myproject/
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `mkdir a/b/c` fails if `a/b` doesn't exist | Use `mkdir -p a/b/c` to create the full path |
| `rmdir` fails on non-empty directory | Use `rm -rf` only after confirming the contents |
| `rm -rf` with a typo in path | Always `ls` the path first; use tab completion |
| Running `rm -rf /` or `rm -rf ~` | Use `--preserve-root` flag; double-check paths |
| Forgetting permissions on new directories | Use `mkdir -m 755` or `chmod` after creation |

---

## 🔑 Key Takeaways

- `mkdir` creates directories; use `-p` to create the full path including missing parents.
- **Brace expansion** (`mkdir -p project/{src,tests,docs}`) creates multiple subdirectories in one command.
- `rmdir` is the **safe option** — it only removes empty directories and fails gracefully on non-empty ones.
- `rm -rf` is **permanent and irreversible** — always verify the path before using it.
- Use `rm -ri` (interactive) for a safer deletion experience when you need to confirm each file.
- `mktemp -d` creates a **unique, safe temporary directory** for use in scripts.
- Use `mkdir "backup_$(date +%Y-%m-%d)"` to create **automatically dated** directories.

---

## 📚 Related Concepts

- [Challenge 01 — List Hidden Files]
- [GNU mkdir Manual](https://www.gnu.org/software/coreutils/manual/html_node/mkdir-invocation.html)
- [GNU rm Manual](https://www.gnu.org/software/coreutils/manual/html_node/rm-invocation.html)


</details>
