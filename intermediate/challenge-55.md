# Challenge 55: Manage Users and Groups in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

User and group management is a core part of **Linux system administration**. Linux uses a permission model where every file and process is owned by a **user** and belongs to a **group**. Managing users and groups lets you control **who can access what** on the system — a fundamental principle of security and multi-user environments.

---

## 🧩 Task

Create a new group and a new user, then assign the user to that group.

---


<details>
<summary>💡 Click to view solution</summary>
  
## ✅ Solution

```bash
# Create a new group
sudo groupadd devgroup

# Create a new user and assign to the group
sudo useradd -G devgroup devuser
```

### How it works

| Command | Description |
|---------|-------------|
| `sudo` | Run the command with superuser (root) privileges |
| `groupadd devgroup` | Creates a new group named `devgroup` |
| `useradd -G devgroup devuser` | Creates a new user `devuser` and adds them to `devgroup` |
| `-G` | Specifies supplementary (secondary) group(s) for the user |

---

## 📖 Extended Examples

### Example 1 — Create a Group

```bash
sudo groupadd devgroup
```

Verify it was created:

```bash
cat /etc/group | grep devgroup
```

```bash
devgroup:x:1002:
```

---

### Example 2 — Create a User and Assign to a Group

```bash
sudo useradd -G devgroup devuser
```

Verify the user and group assignment:

```bash
id devuser
```

```bash
uid=1001(devuser) gid=1001(devuser) groups=1001(devuser),1002(devgroup)
```

---

### Example 3 — Create a User with Full Options

```bash
sudo useradd \
  -m \               # Create home directory
  -s /bin/bash \     # Set default shell
  -G devgroup \      # Add to supplementary group
  -c "Dev User" \    # Add a comment / full name
  devuser
```

Or on a single line:

```bash
sudo useradd -m -s /bin/bash -G devgroup -c "Dev User" devuser
```

---

### Example 4 — Set a Password for the User

```bash
sudo passwd devuser
```

```bash
New password:
Retype new password:
passwd: password updated successfully
```

> 💡 A user without a password cannot log in interactively. Always set a password after creating a user.

---

### Example 5 — Add an Existing User to a Group

```bash
sudo usermod -aG devgroup existinguser
```

> ⚠️ Always use `-aG` (append + group). Using `-G` alone **overwrites** all existing group memberships.

```bash
# Verify
groups existinguser
```

```bash
existinguser : existinguser devgroup
```

---

### Example 6 — Create Multiple Groups and Assign All to a User

```bash
sudo groupadd developers
sudo groupadd testers
sudo groupadd deployers

sudo useradd -m -s /bin/bash -G developers,testers,deployers ciuser
```

```bash
# Verify
id ciuser
```

```bash
uid=1003(ciuser) gid=1003(ciuser) groups=1003(ciuser),1004(developers),1005(testers),1006(deployers)
```

---

### Example 7 — Modify a User's Primary Group

```bash
sudo usermod -g devgroup devuser
```

> 💡 `-g` (lowercase) changes the **primary group**. `-G` (uppercase) manages **supplementary groups**.

---

### Example 8 — Delete a User

```bash
# Delete user only
sudo userdel devuser

# Delete user AND their home directory
sudo userdel -r devuser
```

> ⚠️ `-r` permanently removes the user's home directory and mail spool. Use with caution.

---

### Example 9 — Delete a Group

```bash
sudo groupdel devgroup
```

> 💡 You cannot delete a group that is the **primary group** of any existing user. Remove or reassign those users first.

---

### Example 10 — Lock and Unlock a User Account

```bash
# Lock account (disables login)
sudo usermod -L devuser

# Unlock account
sudo usermod -U devuser

# Verify — a locked account shows '!' in /etc/shadow
sudo grep devuser /etc/shadow
```

```bash
# Locked:
devuser:!$6$hash...:19000:0:99999:7:::

# Unlocked:
devuser:$6$hash...:19000:0:99999:7:::
```

---

## 🗂️ Key System Files

| File | Purpose |
|------|---------|
| `/etc/passwd` | Stores user account information (username, UID, home dir, shell) |
| `/etc/shadow` | Stores encrypted passwords and password policy |
| `/etc/group` | Stores group names, GIDs, and group members |
| `/etc/gshadow` | Stores secure group account information |

```bash
# View user info
cat /etc/passwd | grep devuser

# View group info
cat /etc/group | grep devgroup
```

```bash
# /etc/passwd format:
# username:password:UID:GID:comment:home_dir:shell
devuser:x:1001:1001:Dev User:/home/devuser:/bin/bash

# /etc/group format:
# groupname:password:GID:members
devgroup:x:1002:devuser
```

---

## 🛠️ Useful Commands — Quick Reference

### User Management

| Command | Description |
|---------|-------------|
| `sudo useradd username` | Create a new user |
| `sudo useradd -m -s /bin/bash username` | Create user with home dir and bash shell |
| `sudo passwd username` | Set or change a user's password |
| `sudo usermod -aG groupname username` | Add user to a group (append) |
| `sudo usermod -s /bin/zsh username` | Change user's default shell |
| `sudo usermod -l newname oldname` | Rename a user |
| `sudo userdel username` | Delete a user |
| `sudo userdel -r username` | Delete user and home directory |
| `sudo usermod -L username` | Lock a user account |
| `sudo usermod -U username` | Unlock a user account |
| `id username` | Show UID, GID, and group memberships |
| `whoami` | Show current logged-in user |
| `groups username` | List all groups a user belongs to |

### Group Management

| Command | Description |
|---------|-------------|
| `sudo groupadd groupname` | Create a new group |
| `sudo groupdel groupname` | Delete a group |
| `sudo groupmod -n newname oldname` | Rename a group |
| `cat /etc/group` | List all groups on the system |
| `getent group groupname` | Show group details and members |

---

## 👤 Primary Group vs Supplementary Groups

| Type | Description |
|------|-------------|
| **Primary Group** | The default group assigned to files created by the user. Set with `-g`. Each user has exactly one primary group. |
| **Supplementary Groups** | Additional groups the user belongs to, granting extra permissions. Set with `-G` or appended with `-aG`. A user can belong to many. |

```bash
# Check both
id devuser
# uid=1001(devuser) gid=1001(devuser) groups=1001(devuser),1002(devgroup)
#                   ^^^^ primary              ^^^^ supplementary
```

---

## 🚀 Full Workflow — Create and Configure a User

```bash
# Step 1: Create the group
sudo groupadd devgroup

# Step 2: Create the user with home directory and bash shell
sudo useradd -m -s /bin/bash -G devgroup -c "Dev User" devuser

# Step 3: Set a password
sudo passwd devuser

# Step 4: Verify
id devuser
groups devuser
ls /home/devuser
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Using `-G` instead of `-aG` with `usermod` | Always use `usermod -aG` to avoid overwriting existing groups |
| Forgetting to set a password | Run `sudo passwd username` after `useradd` |
| Deleting a group that is a user's primary group | Reassign the user's primary group first with `usermod -g` |
| Changes not taking effect in current session | Log out and back in, or run `newgrp groupname` |
| Home directory not created | Use `-m` flag with `useradd`: `sudo useradd -m username` |

---

## 🔑 Key Takeaways

- `groupadd` creates groups; `useradd` creates users.
- Use `-G` with `useradd` to assign supplementary groups **at creation time**.
- Use `usermod -aG` to add an existing user to a group — the `-a` flag **appends** instead of replacing.
- Every user has one **primary group** (`-g`) and can belong to many **supplementary groups** (`-G`).
- User and group data is stored in `/etc/passwd`, `/etc/shadow`, and `/etc/group`.
- Always run `id username` or `groups username` to verify memberships after making changes.

---

## 📚 Related Concepts

- [Linux File Permissions](https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html)
- [sudo and sudoers](https://www.sudo.ws/docs/man/sudoers.man/)
- [PAM — Pluggable Authentication Modules](https://www.linux-pam.org/)
- [ACLs — Access Control Lists](https://linux.die.net/man/5/acl) for fine-grained permission control

</details>
