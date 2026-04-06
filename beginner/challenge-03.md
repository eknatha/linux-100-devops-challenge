# Challenge 03: Create a New User in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**User management** is a core Linux administration skill. Every person or service that interacts with a Linux system does so through a **user account**. Creating users correctly — with the right home directory, shell, group membership, and password — ensures security, proper access control, and system organization.

Linux stores user information in two key files:
- `/etc/passwd` — usernames, UIDs, home directories, and default shells
- `/etc/shadow` — encrypted passwords and password aging policy

---

## 🧩 Task

Create a new user account in the Linux system.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Create a new user
sudo useradd username

# Create a user with a home directory and bash shell (recommended)
sudo useradd -m -s /bin/bash username

# Set a password for the new user
sudo passwd username
```

### How it works

| Part | Description |
|------|-------------|
| `sudo` | Run with superuser privileges — required for user management |
| `useradd` | Add a new user account to the system |
| `-m` | Create the user's **home directory** (`/home/username`) |
| `-s /bin/bash` | Set the default **login shell** to bash |
| `username` | The name of the new user account |
| `passwd username` | Set or change the user's **password** |

---

## 🔢 What Happens When You Create a User

```
sudo useradd -m -s /bin/bash devuser
        │
        ├── Adds entry to /etc/passwd
        │   devuser:x:1001:1001::/home/devuser:/bin/bash
        │
        ├── Adds entry to /etc/shadow
        │   devuser:!:19000:0:99999:7:::
        │   (! = password not yet set)
        │
        ├── Creates primary group in /etc/group
        │   devuser:x:1001:
        │
        └── Creates home directory
            /home/devuser/   (copied from /etc/skel/)
```

---

## 📖 Extended Examples

### Example 1 — Basic User Creation

```bash
sudo useradd username
```

> ⚠️ Without `-m`, no home directory is created. Always add `-m` for interactive users.

---

### Example 2 — Create a User with Home Directory and Shell (Recommended)

```bash
sudo useradd -m -s /bin/bash devuser

# Verify the user was created
id devuser
```

```bash
uid=1001(devuser) gid=1001(devuser) groups=1001(devuser)
```

---

### Example 3 — Set a Password

```bash
sudo passwd devuser
```

```bash
New password:
Retype new password:
passwd: password updated successfully
```

> 💡 A user without a password **cannot log in interactively**. Always set a password after creating an interactive user.

---

### Example 4 — Create a User with Full Options

```bash
sudo useradd \
  -m \                    # Create home directory
  -s /bin/bash \          # Set default shell
  -c "Dev User" \         # Full name / comment
  -G sudo,developers \    # Add to supplementary groups
  devuser
```

Or on a single line:

```bash
sudo useradd -m -s /bin/bash -c "Dev User" -G sudo,developers devuser
```

| Flag | Description |
|------|-------------|
| `-m` | Create home directory at `/home/username` |
| `-s SHELL` | Set the login shell |
| `-c COMMENT` | Full name or description (shown in `finger`/`getent`) |
| `-G GROUP,...` | Add to supplementary groups (comma-separated) |
| `-d DIR` | Set a custom home directory path |
| `-u UID` | Set a specific User ID |
| `-g GID` | Set the primary group |
| `-e DATE` | Set account expiration date (YYYY-MM-DD) |

---

### Example 5 — Create a User with a Custom Home Directory

```bash
# Create user with home in a non-standard location
sudo useradd -m -d /opt/appuser -s /bin/bash appuser

# Verify
ls -la /opt/appuser
```

---

### Example 6 — Create a System User (No Login)

Service accounts for applications like nginx, mysql, or custom daemons should be **system users** — they have no home directory and cannot log in interactively:

```bash
# Create a system user (no home, no login shell)
sudo useradd --system --no-create-home --shell /usr/sbin/nologin serviceaccount

# Verify — system users have UID < 1000
id serviceaccount
```

```bash
uid=998(serviceaccount) gid=998(serviceaccount) groups=998(serviceaccount)
```

---

### Example 7 — Verify the User Was Created

```bash
# Check the user entry in /etc/passwd
grep devuser /etc/passwd
```

```bash
devuser:x:1001:1001:Dev User:/home/devuser:/bin/bash
#  │    │  │    │   │         │              └── shell
#  │    │  │    │   │         └── home directory
#  │    │  │    │   └── comment / full name
#  │    │  │    └── primary GID
#  │    │  └── UID
#  │    └── password (x = stored in /etc/shadow)
#  └── username
```

```bash
# Check the home directory was created
ls -la /home/devuser/

# Check group membership
groups devuser
id devuser
```

---

### Example 8 — Use `adduser` (Interactive — Debian/Ubuntu)

On Debian/Ubuntu, `adduser` is a friendlier wrapper around `useradd`:

```bash
sudo adduser devuser
```

```bash
Adding user 'devuser' ...
Adding new group 'devuser' (1001) ...
Adding new user 'devuser' (1001) with group 'devuser' ...
Creating home directory '/home/devuser' ...
Copying files from '/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for devuser
Enter the new value, or press ENTER for the default
  Full Name []: Dev User
  Room Number []:
  Work Phone []:
  Home Phone []:
  Other []:
Is the information correct? [Y/n] Y
```

> 💡 `adduser` is **interactive** — it asks for password and user info during creation. It is Debian/Ubuntu specific. On RHEL/CentOS use `useradd`.

---

### Example 9 — Create Multiple Users from a Script

```bash
#!/bin/bash

USERS=("alice" "bob" "charlie")
PASSWORD="ChangeMe123!"

for USER in "${USERS[@]}"; do
  if id "$USER" &>/dev/null; then
    echo "⚠️  User $USER already exists — skipping"
  else
    sudo useradd -m -s /bin/bash "$USER"
    echo "$USER:$PASSWORD" | sudo chpasswd
    echo "✅ Created user: $USER"
  fi
done
```

```bash
✅ Created user: alice
✅ Created user: bob
✅ Created user: charlie
```

---

### Example 10 — Lock and Unlock a New User Account

```bash
# Create user but lock the account (cannot log in yet)
sudo useradd -m -s /bin/bash devuser
sudo usermod -L devuser        # Lock

# Unlock when ready
sudo usermod -U devuser        # Unlock

# Check lock status (! before hash in shadow = locked)
sudo grep devuser /etc/shadow | cut -d: -f2 | cut -c1
# Output: ! = locked, $ = active
```

---

## 🗂️ `useradd` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-m` | Create home directory |
| `-M` | Do NOT create home directory |
| `-d DIR` | Custom home directory path |
| `-s SHELL` | Login shell (e.g., `/bin/bash`, `/bin/zsh`) |
| `-c COMMENT` | Full name or description |
| `-G GROUP,...` | Supplementary group(s) |
| `-g GROUP` | Primary group |
| `-u UID` | Specify custom UID |
| `-e DATE` | Account expiration (YYYY-MM-DD) |
| `-f DAYS` | Days after password expires before account disabled |
| `--system` | Create a system account (UID < 1000) |
| `--no-create-home` | Do not create home directory |
| `--shell /sbin/nologin` | Prevent interactive login |

---

## 🔄 `useradd` vs `adduser`

| Feature | `useradd` | `adduser` |
|---------|-----------|-----------|
| Type | Low-level binary | High-level script |
| Availability | All distros | Debian/Ubuntu only |
| Home directory | Manual (`-m`) | Automatic |
| Password prompt | Separate `passwd` | Built-in |
| Interactivity | Non-interactive | Interactive |
| Best for | Scripts, RHEL | Desktop/Ubuntu |

---

## 🚀 Full Workflow — Create a Fully Configured User

```bash
# Step 1: Create the user with home directory and shell
sudo useradd -m -s /bin/bash -c "Dev User" devuser

# Step 2: Set a strong password
sudo passwd devuser

# Step 3: Add to necessary groups (e.g., sudo for admin access)
sudo usermod -aG sudo devuser      # Debian/Ubuntu
sudo usermod -aG wheel devuser     # RHEL/CentOS

# Step 4: Verify user and group membership
id devuser
groups devuser

# Step 5: Verify home directory
ls -la /home/devuser/

# Step 6: Check /etc/passwd entry
grep devuser /etc/passwd

# Step 7: Test the account (switch to it)
su - devuser
# Or
ssh devuser@localhost
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Creating user without `-m` | User has no home directory — add `-m` or create manually |
| Forgetting to set a password | Run `sudo passwd username` immediately after `useradd` |
| Not specifying a shell | Default shell may be `/sh` or `/nologin` — always use `-s /bin/bash` |
| Using `useradd` without `sudo` | Requires root — always prefix with `sudo` |
| User already exists | Check with `id username` before creating |
| Wrong group name in `-G` | Group must exist before assigning — create with `groupadd` first |

---

## 🔑 Key Takeaways

- `useradd -m -s /bin/bash username` is the standard command — always include `-m` (home directory) and `-s` (shell) for interactive users.
- Always run `sudo passwd username` immediately after creation — a user without a password cannot log in.
- Use `-G` to add the user to supplementary groups at creation time, or `usermod -aG` later.
- **System accounts** (for services) use `--system --no-create-home --shell /sbin/nologin` — they should never be able to log in interactively.
- `adduser` on Debian/Ubuntu is more beginner-friendly and interactive; `useradd` is universal and better for scripting.
- Verify every new user with `id username` and `grep username /etc/passwd`.

---

## 📚 Related Concepts

- [GNU useradd Manual](https://linux.die.net/man/8/useradd)
- [passwd Manual](https://linux.die.net/man/1/passwd)

</details>
