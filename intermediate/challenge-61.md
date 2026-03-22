# Challenge 61: Recursive Permissions in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

## 📌 Overview

**File permissions** in Linux control who can read, write, and execute files and directories. The `chmod` (**ch**ange **mod**e) command modifies these permissions. The `-R` flag applies changes **recursively** — to a directory and everything inside it, including all subdirectories and files.

Understanding permissions is fundamental to Linux security, preventing unauthorized access and ensuring applications run with the minimum privileges they need.

---

## 🧩 Task

Recursively set permissions on a directory and all its contents.

---


<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
chmod -R 755 mydir/
```

### How it works

| Part | Description |
|------|-------------|
| `chmod` | **Ch**ange file **mod**e (permissions) |
| `-R` | **Recursive** — apply to directory and all contents |
| `755` | Octal permission value — `rwxr-xr-x` |
| `mydir/` | Target directory |

### Decoding `755`

```
  7       5       5
 rwx     r-x     r-x
Owner   Group   Others
```

| Digit | Binary | Permissions | Meaning |
|-------|--------|-------------|---------|
| `7` | `111` | `rwx` | Owner can read, write, execute |
| `5` | `101` | `r-x` | Group can read and execute |
| `5` | `101` | `r-x` | Others can read and execute |

---

## 📖 Permission Bits Explained

Each permission is represented by a **bit** with a numeric value:

| Permission | Symbol | Octal Value |
|------------|--------|-------------|
| Read | `r` | `4` |
| Write | `w` | `2` |
| Execute | `x` | `1` |
| No permission | `-` | `0` |

Add the values together for each category (owner, group, others):

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

---

## 📖 Extended Examples

### Example 1 — Recursive Permission `755` (Standard Directories)

```bash
chmod -R 755 mydir/
```

```bash
# Verify
ls -l mydir/
```

```bash
drwxr-xr-x 3 user group 4096 Mar 21 10:00 mydir/
-rwxr-xr-x 1 user group  512 Mar 21 10:00 mydir/script.sh
-rwxr-xr-x 1 user group 1024 Mar 21 10:00 mydir/subdir/
```

> Best for web directories and shared project folders where everyone should be able to read and execute.

---

### Example 2 — Recursive Permission `644` (Standard Files)

```bash
chmod -R 644 mydir/
```

```bash
-rw-r--r-- 1 user group  512 Mar 21 10:00 mydir/config.txt
-rw-r--r-- 1 user group 1024 Mar 21 10:00 mydir/notes.md
```

> Owner can read and write; group and others can only read. Ideal for configuration and document files.

---

### Example 3 — Recursive Permission `700` (Private Directory)

```bash
chmod -R 700 mydir/
```

```bash
drwx------ 3 user group 4096 Mar 21 10:00 mydir/
-rwx------ 1 user group  512 Mar 21 10:00 mydir/private.sh
```

> Only the owner has any access. Perfect for private scripts, SSH keys, and sensitive data directories.

---

### Example 4 — Recursive Permission `777` (Full Access — Use With Caution)

```bash
chmod -R 777 mydir/
```

```bash
drwxrwxrwx 3 user group 4096 Mar 21 10:00 mydir/
-rwxrwxrwx 1 user group  512 Mar 21 10:00 mydir/script.sh
```

> Everyone can read, write, and execute. **Avoid in production** — it is a major security risk.

---

### Example 5 — Symbolic Mode (Add or Remove Permissions)

Instead of octal numbers, use symbolic notation to **add**, **remove**, or **set** specific permissions:

```bash
# Add execute permission for everyone
chmod -R +x mydir/

# Remove write permission for group and others
chmod -R go-w mydir/

# Set exact permissions using symbolic mode
chmod -R u=rwx,g=rx,o=rx mydir/
```

| Symbol | Meaning |
|--------|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u+g+o) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

---

### Example 6 — Set Different Permissions for Files and Directories

A common real-world need: directories need execute (`x`) to be entered, but regular files usually don't. Use `find` to apply different permissions selectively:

```bash
# Set directories to 755
find mydir/ -type d -exec chmod 755 {} \;

# Set files to 644
find mydir/ -type f -exec chmod 644 {} \;
```

> 💡 This is **safer than `chmod -R 755`** which would make all files executable — usually undesirable for text and config files.

---

### Example 7 — Set Permissions on Files with a Specific Extension

```bash
# Make all shell scripts executable
find mydir/ -name "*.sh" -exec chmod +x {} \;

# Restrict all config files to owner-read-only
find mydir/ -name "*.conf" -exec chmod 600 {} \;
```

---

### Example 8 — Change Ownership Recursively with `chown`

Permissions and ownership go hand in hand. Use `chown` with `-R` to recursively change the owner and/or group:

```bash
# Change owner
sudo chown -R devuser mydir/

# Change owner and group
sudo chown -R devuser:devgroup mydir/

# Change group only
sudo chown -R :devgroup mydir/
```

```bash
# Verify
ls -l mydir/
```

```bash
drwxr-xr-x 3 devuser devgroup 4096 Mar 21 10:00 mydir/
```

---

### Example 9 — Set SUID, SGID, and Sticky Bit

Beyond standard permissions, Linux supports three special bits:

```bash
# SUID — execute as file owner (common on /usr/bin/passwd)
chmod u+s script.sh
# or
chmod 4755 script.sh

# SGID — new files in directory inherit the group
chmod g+s mydir/
# or
chmod 2755 mydir/

# Sticky Bit — only file owner can delete files (used on /tmp)
chmod +t mydir/
# or
chmod 1755 mydir/
```

| Special Bit | Octal Prefix | Effect |
|-------------|-------------|--------|
| SUID | `4xxx` | File runs as owner's user |
| SGID | `2xxx` | Files inherit directory's group |
| Sticky | `1xxx` | Only owner can delete their files |

```bash
# View special bits
ls -l mydir/
# drwxr-sr-x → SGID set (s in group execute position)
# drwxrwxrwt → Sticky bit set (t in others execute position)
```

---

### Example 10 — View and Verify Permissions

```bash
# Long listing with permissions
ls -l mydir/

# Recursive listing
ls -lR mydir/

# Numeric permissions (using stat)
stat -c "%a %n" mydir/*
```

```bash
# ls -l output explained:
drwxr-xr-x  2  devuser  devgroup  4096  Mar 21  mydir/
│││││││││││  │     │        │
│││││││││││  │     │        └── Group name
│││││││││││  │     └─────────── Owner name
│││││││││││  └───────────────── Number of hard links
│└┤└┤└┤└┤└┤
│ │  │  │  └── Others: r-x
│ │  │  └───── Group:  r-x
│ │  └──────── Owner:  rwx
│ └─────────── File type: d=directory, -=file, l=symlink
└───────────── File type indicator
```

---

## 🗂️ Common Permission Combinations

| Octal | Symbolic | Use Case |
|-------|----------|----------|
| `700` | `rwx------` | Private scripts, SSH keys |
| `755` | `rwxr-xr-x` | Web directories, shared project folders |
| `644` | `rw-r--r--` | Config files, documents, web assets |
| `600` | `rw-------` | Private files, `.env`, credential files |
| `777` | `rwxrwxrwx` | ⚠️ Avoid — full access for everyone |
| `400` | `r--------` | Read-only files (backups, certificates) |
| `664` | `rw-rw-r--` | Shared files within a group |
| `1777` | `rwxrwxrwt` | Shared temp directories (like `/tmp`) |
| `2755` | `rwxr-sr-x` | Shared directories with group inheritance |

---

## 🚀 Full Workflow — Set Up a Secure Web Directory

```bash
# Step 1: Create the directory structure
mkdir -p /var/www/myapp/{public,config,logs}

# Step 2: Set directory permissions
find /var/www/myapp/ -type d -exec chmod 755 {} \;

# Step 3: Set file permissions
find /var/www/myapp/ -type f -exec chmod 644 {} \;

# Step 4: Secure config and log directories
chmod 700 /var/www/myapp/config/
chmod 700 /var/www/myapp/logs/

# Step 5: Make scripts executable
find /var/www/myapp/ -name "*.sh" -exec chmod +x {} \;

# Step 6: Set ownership to web server user
sudo chown -R www-data:www-data /var/www/myapp/

# Step 7: Verify
ls -lR /var/www/myapp/
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `chmod -R 755` makes all files executable | Use `find -type f` and `find -type d` to set files and directories separately |
| Setting `777` for convenience | Use the least permissive setting that still works — start with `755`/`644` |
| Forgetting execute bit on directories | Directories need `x` to be entered — `644` on a directory blocks `cd` |
| Changing permissions on system files | Never run `chmod -R` on `/`, `/etc/`, or `/usr/` — it can break the OS |
| Permissions correct but access denied | Check **ownership** with `ls -l` — `chown` may also be needed |
| SSH key not working after chmod | SSH private keys must be `600` — any broader permission is rejected |

---

## 🔑 Key Takeaways

- `-R` makes `chmod` apply **recursively** to all files and subdirectories.
- Permissions have three categories: **owner (u)**, **group (g)**, **others (o)**.
- Each category has three bits: **read (4)**, **write (2)**, **execute (1)** — add them for the octal value.
- `755` is the standard for directories; `644` is the standard for files.
- Use **`find -type d` and `find -type f`** to set directories and files to different permissions in one sweep.
- Use **symbolic mode** (`+x`, `go-w`) for relative changes without overwriting all bits.
- Permissions and **ownership (`chown`)** work together — always check both when troubleshooting access issues.

---

## 📚 Related Concepts

- [Linux File Permissions Guide](https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html)
- [Access Control Lists (ACLs)](https://linux.die.net/man/5/acl) — for fine-grained per-user permissions beyond owner/group/others
- [umask](https://linux.die.net/man/2/umask) — sets default permissions for newly created files and directories

</details>
