# Challenge 09: Print Current Working Directory in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

The **working directory** is the directory you are currently "inside" in the terminal. Every command you run operates relative to this location — when you open a file, list directory contents, or create a new file, Linux looks in the current working directory unless you specify an absolute path.

The `pwd` (**P**rint **W**orking **D**irectory) command instantly tells you exactly where you are in the filesystem — essential for orientation and navigation.

---

## 🧩 Task

Print the full path of the current working directory.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
pwd
```

### How it works

| Part | Description |
|------|-------------|
| `pwd` | **P**rint **W**orking **D**irectory — outputs the full absolute path of the current directory |

---

## 🔢 Understanding `pwd` Output

```bash
pwd
```

```
/home/devuser/projects/myapp
```

This is an **absolute path** — it starts from the root `/` and shows every directory down to your current location:

```
/home/devuser/projects/myapp
│    │         │         │
│    │         │         └── Current directory: myapp
│    │         └──────────── Parent: projects
│    └────────────────────── Parent: devuser (home dir)
└─────────────────────────── Root of the filesystem
```

---

## 📖 Extended Examples

### Example 1 — Basic Usage

```bash
pwd
```

```bash
/home/devuser
```

---

### Example 2 — After Navigating Directories

```bash
# Start in home
pwd
# /home/devuser

# Navigate into a directory
cd /var/log

# See where you are now
pwd
# /var/log

# Navigate deeper
cd nginx

# Check again
pwd
# /var/log/nginx
```

---

### Example 3 — Show Logical vs Physical Path (`-L` and `-P`)

```bash
# Default — shows logical path (follows shell variable $PWD, respects symlinks by name)
pwd -L

# Physical — resolves all symlinks to show the real path
pwd -P
```

```bash
# Example with a symlink:
# /var/www/html → symlink pointing to → /data/webroot/html

cd /var/www/html

pwd -L    # /var/www/html    ← shows the symlink path
pwd -P    # /data/webroot/html  ← shows the real path
```

> 💡 Use `pwd -P` when you need to know the **actual physical location** on disk, especially when symlinks are involved.

---

### Example 4 — Use `pwd` in a Script

```bash
#!/bin/bash

# Save current directory to return to later
ORIGINAL_DIR=$(pwd)
echo "Starting in: $ORIGINAL_DIR"

# Change to another directory and do work
cd /tmp
echo "Working in: $(pwd)"

# Return to original directory
cd "$ORIGINAL_DIR"
echo "Back to: $(pwd)"
```

```bash
Starting in: /home/devuser/scripts
Working in: /tmp
Back to: /home/devuser/scripts
```

---

### Example 5 — Use `$PWD` Environment Variable

Linux automatically maintains the `$PWD` environment variable — it always contains the current directory path:

```bash
# Print using the environment variable directly
echo $PWD

# Use in a string
echo "You are in: $PWD"

# Use in a script to construct file paths
LOG_FILE="$PWD/output.log"
echo "Log will be saved to: $LOG_FILE"
```

```bash
/home/devuser
You are in: /home/devuser
Log will be saved to: /home/devuser/output.log
```

---

### Example 6 — Use `pwd` with `cd` Navigation

```bash
# Common navigation pattern
cd ~          # Go to home directory
pwd           # /home/devuser

cd ..         # Go up one level
pwd           # /home

cd /etc       # Jump to /etc
pwd           # /etc

cd -          # Return to previous directory
pwd           # /home/devuser

cd /var/log/nginx
pwd           # /var/log/nginx
```

| `cd` Shortcut | Meaning |
|---------------|---------|
| `cd ~` | Go to home directory |
| `cd ..` | Go up one level |
| `cd -` | Go to previous directory |
| `cd /` | Go to root |
| `cd /path` | Go to absolute path |

---

### Example 7 — Combine `pwd` with Other Commands

```bash
# Create a file in the current directory using $PWD
touch "$PWD/newfile.txt"

# Display the current directory in a custom prompt style
echo "[$(pwd)] $"

# Use pwd output in a log entry
echo "$(date): Running backup from $(pwd)" >> /var/log/backup.log

# Copy something to the current directory
cp /tmp/report.csv "$(pwd)/"
```

---

### Example 8 — Check Working Directory of Another Process

```bash
# See the current working directory of any running process
ls -la /proc/<PID>/cwd

# Example: check nginx's working directory
ls -la /proc/$(pgrep nginx | head -1)/cwd
```

```bash
lrwxrwxrwx 1 root root 0 Mar 21 10:00 /proc/5201/cwd -> /
```

> Every process has a current working directory stored in `/proc/<PID>/cwd` as a symlink.

---

## 🗂️ `pwd` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-L` | **Logical** path — follows symlink names (default behavior) |
| `-P` | **Physical** path — resolves all symlinks to actual directory |

---

## 🔄 `pwd` vs `$PWD`

| Method | Description | Use Case |
|--------|-------------|----------|
| `pwd` | Command — prints to stdout | Interactive terminal use |
| `echo $PWD` | Shell variable — current directory | Scripting, string interpolation |
| `$(pwd)` | Command substitution — captures output | Embedding in commands or strings |

---

## 🚀 Full Workflow — Navigate and Track Your Location

```bash
# Step 1: Check where you start
pwd

# Step 2: Navigate to a working directory
cd /var/www/myapp

# Step 3: Confirm your location
pwd

# Step 4: Save your location in a variable
WORKDIR=$(pwd)
echo "Working directory: $WORKDIR"

# Step 5: Do your work
ls -la
cat config.yaml

# Step 6: Move to another location temporarily
cd /tmp

# Step 7: Return to your saved location
cd "$WORKDIR"
pwd

# Step 8: Alternatively, use cd - to go back to previous
cd /etc
pwd     # /etc
cd -
pwd     # /var/www/myapp
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Forgetting where you are after multiple `cd` commands | Run `pwd` to orient yourself |
| Symlink path confusion | Use `pwd -P` to see the real physical path |
| Scripts failing because relative paths break after `cd` | Save `$(pwd)` before `cd` and use absolute paths in scripts |
| `$PWD` not updating | `$PWD` is automatically updated by the shell — no manual update needed |
| Running a command in the wrong directory | Always `pwd` before running destructive or location-sensitive commands |

---

## 🔑 Key Takeaways

- `pwd` prints the **full absolute path** of your current location in the filesystem.
- The path always starts with `/` (root) and shows every directory down to where you are.
- `$PWD` is an **environment variable** automatically maintained by the shell — equivalent to `pwd` but usable inline in strings.
- `pwd -P` resolves **symlinks** to show the real physical location on disk.
- Always run `pwd` to **orient yourself** after multiple directory changes — especially before running commands that depend on location.
- In scripts, save `$(pwd)` to a variable at the start if you need to **return to the original directory** later.

---

## 📚 Related Concepts

- [GNU pwd Manual](https://www.gnu.org/software/coreutils/manual/html_node/pwd-invocation.html)
- [Linux Filesystem Hierarchy](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)


</details>
