# Challenge 04: Change File Permissions in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**File permissions** in Linux control who can read, write, and execute files. Every file and directory has three sets of permissions assigned to three categories of users — the **owner**, the **group**, and **others**. Understanding and managing permissions is fundamental to Linux security and system administration.

The `chmod` (**ch**ange **mod**e) command is used to modify these permissions using either **numeric (octal)** or **symbolic** notation.

---

## 🧩 Task

Give read, write, and execute permissions to the file owner.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Using numeric (octal) notation
chmod 700 filename

# Using symbolic notation
chmod u+rwx filename
```

### How it works

| Part | Description |
|------|-------------|
| `chmod` | **Ch**ange file **mod**e (permissions) |
| `700` | Owner gets **rwx** (7), group gets nothing (0), others get nothing (0) |
| `u+rwx` | Add **r**ead, **w**rite, e**x**ecute permissions for the **u**ser (owner) |
| `filename` | The target file to modify |

---

## 🔢 Understanding Linux Permissions

### The Permission Model

```
-rwxrw-r--
│└┤└┤└┤
│ │  │  └── Others:  r-- = read only
│ │  └───── Group:   rw- = read + write
│ └──────── Owner:   rwx = read + write + execute
└────────── File type: - = file, d = directory, l = symlink
```

### Permission Bits and Values

| Permission | Symbol | Octal Value |
|------------|--------|-------------|
| Read | `r` | `4` |
| Write | `w` | `2` |
| Execute | `x` | `1` |
| No permission | `-` | `0` |

Add the values together for each category:

```
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
--- = 0 + 0 + 0 = 0
```

### Three Digit Octal — Owner, Group, Others

```
chmod  7  5  5  filename
       │  │  └── Others: r-x (4+1=5)
       │  └───── Group:  r-x (4+1=5)
       └──────── Owner:  rwx (4+2+1=7)
```

---

## 📖 Extended Examples

### Example 1 — Give Owner Full Permissions (rwx)

```bash
chmod 700 filename
# or
chmod u+rwx filename
```

```bash
# Verify
ls -la filename
```

```bash
-rwx------ 1 devuser devuser 1024 Mar 21 10:00 filename
```

> Owner has full read, write, execute. Group and others have no access.

---

### Example 2 — Common Permission Combinations

```bash
# Owner: rwx | Group: r-x | Others: r-x (standard for executables)
chmod 755 script.sh

# Owner: rw- | Group: r-- | Others: r-- (standard for files)
chmod 644 document.txt

# Owner: rwx | Group: --- | Others: --- (private scripts)
chmod 700 private.sh

# Owner: rw- | Group: --- | Others: --- (private files)
chmod 600 secret.key

# Owner: rw- | Group: rw- | Others: r-- (shared group file)
chmod 664 shared.txt
```

---

### Example 3 — Symbolic Mode (Add Permissions)

```bash
# Add execute to owner only
chmod u+x script.sh

# Add read and write to owner
chmod u+rw file.txt

# Add execute to everyone
chmod +x script.sh

# Add write to group
chmod g+w shared.txt

# Add read to others
chmod o+r public.txt
```

---

### Example 4 — Symbolic Mode (Remove Permissions)

```bash
# Remove write from group
chmod g-w file.txt

# Remove all permissions from others
chmod o-rwx private.txt

# Remove execute from everyone
chmod a-x file.txt

# Remove write from group and others
chmod go-w file.txt
```

---

### Example 5 — Symbolic Mode (Set Exact Permissions)

```bash
# Set owner to rwx, group to r-x, others to r--
chmod u=rwx,g=rx,o=r file.txt

# Set everyone to read-only
chmod a=r file.txt

# Set owner to rw, remove all from group and others
chmod u=rw,go= file.txt
```

---

### Example 6 — Symbolic Notation Reference

| Symbol | Meaning |
|--------|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u + g + o) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exactly (overwrite existing) |
| `r` | Read |
| `w` | Write |
| `x` | Execute |

---

### Example 7 — Change Permissions Recursively

```bash
# Apply permissions to a directory and all its contents
chmod -R 755 mydir/

# Set files to 644 and directories to 755 (best practice)
find mydir/ -type f -exec chmod 644 {} \;
find mydir/ -type d -exec chmod 755 {} \;
```

> 💡 `chmod -R 755` makes all files executable too, which is usually undesirable. Use `find` to set files and directories separately.

---

### Example 8 — Check Permissions Before and After

```bash
# Before
ls -la myfile.txt
# -rw-r--r-- 1 devuser devuser 1024 Mar 21 10:00 myfile.txt

# Change
chmod 700 myfile.txt

# After
ls -la myfile.txt
# -rwx------ 1 devuser devuser 1024 Mar 21 10:00 myfile.txt
```

```bash
# Check numeric permissions using stat
stat -c "%a %n" myfile.txt
# 700 myfile.txt
```

---

### Example 9 — Make a Script Executable

A very common real-world use case — making a shell script runnable:

```bash
# Create a script
echo '#!/bin/bash' > hello.sh
echo 'echo "Hello, World!"' >> hello.sh

# Before — cannot execute
ls -la hello.sh
# -rw-r--r-- 1 devuser devuser 32 Mar 21 10:00 hello.sh

./hello.sh
# bash: ./hello.sh: Permission denied

# Add execute permission for owner
chmod u+x hello.sh
# or
chmod 755 hello.sh

# Now it works
./hello.sh
# Hello, World!
```

---

### Example 10 — Secure a Private Key File

SSH private keys must have strict permissions — SSH refuses to use keys with too-permissive permissions:

```bash
# Wrong permissions — SSH will refuse this key
ls -la ~/.ssh/id_ed25519
# -rw-r--r-- 1 devuser devuser 411 Mar 21 10:00 id_ed25519

# Fix: owner read/write only
chmod 600 ~/.ssh/id_ed25519

# Verify
ls -la ~/.ssh/id_ed25519
# -rw------- 1 devuser devuser 411 Mar 21 10:00 id_ed25519
```

---

## 🗂️ Common Permission Reference

| Octal | Symbolic | Use Case |
|-------|----------|----------|
| `700` | `rwx------` | Private scripts, SSH keys directories |
| `755` | `rwxr-xr-x` | Public scripts, web directories |
| `644` | `rw-r--r--` | Regular files, config files, web assets |
| `600` | `rw-------` | Private files, SSH keys, `.env` files |
| `777` | `rwxrwxrwx` | ⚠️ Avoid — insecure, full access for all |
| `400` | `r--------` | Read-only backups, certificates |
| `664` | `rw-rw-r--` | Shared group files |
| `750` | `rwxr-x---` | Group-accessible scripts |

---

## 🚀 Full Workflow — Set Up Permissions for a Project

```bash
# Step 1: Create a project structure
mkdir -p myproject/{src,config,scripts,logs}

# Step 2: Set directory permissions
chmod 755 myproject/
chmod 755 myproject/src/
chmod 700 myproject/config/      # Sensitive — owner only
chmod 755 myproject/scripts/
chmod 755 myproject/logs/

# Step 3: Create files and set permissions
touch myproject/config/app.conf
chmod 600 myproject/config/app.conf   # Private config

touch myproject/scripts/deploy.sh
chmod 755 myproject/scripts/deploy.sh # Executable by all

touch myproject/src/main.py
chmod 644 myproject/src/main.py       # Standard file

# Step 4: Verify everything
ls -laR myproject/
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `chmod 777` for convenience | Use the **least permissive** setting — start with `755`/`644` |
| `chmod -R 755` making all files executable | Use `find -type f` and `find -type d` to set separately |
| Forgetting execute on directories | Directories need `x` to be entered — `644` on a directory blocks `cd` |
| Wrong octal order | Always: **Owner** → **Group** → **Others** (left to right) |
| Changing permissions on wrong file | Double-check filename with `ls` before running `chmod` |

---

## 🔑 Key Takeaways

- Permissions have three categories: **owner (u)**, **group (g)**, **others (o)**.
- Each category has three bits: **read (4)**, **write (2)**, **execute (1)** — add them for the octal value.
- `chmod 700` gives the owner full access and blocks everyone else.
- Use **symbolic mode** (`u+x`, `go-w`) for relative changes without overwriting all bits.
- Use **octal mode** (`chmod 755`) when you want to set all permissions at once precisely.
- `ls -la` shows current permissions; `stat -c "%a"` shows the numeric value.
- Always use the **least permissive** setting that still allows the required access.

---

## 📚 Related Concepts

- [GNU chmod Manual](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html)
- [Linux File Permissions Guide](https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html)

</details>
