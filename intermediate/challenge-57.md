# Challenge 57: Configure Sudo Access in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
## 📌 Overview

**`sudo`** (Super User Do) allows permitted users to run commands as the root user or another user, as defined by the security policy. Instead of logging in as root — which is a security risk — `sudo` grants **controlled, audited, temporary elevated privileges**.

Access is managed either by adding users to the `wheel` (RHEL/CentOS/Fedora) or `sudo` (Debian/Ubuntu) group, or by editing the **`/etc/sudoers`** file directly using `visudo`.

---

## 🧩 Task

Grant a user full sudo access.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
sudo usermod -aG wheel user
```

### How it works

| Part | Description |
|------|-------------|
| `sudo` | Execute the command with superuser privileges |
| `usermod` | Modify an existing user account |
| `-aG` | **Append** the user to a supplementary group (`-a` = append, `-G` = group) |
| `wheel` | The privileged group that grants sudo access (RHEL/CentOS/Fedora) |
| `user` | The target username to receive sudo access |

> 💡 On **Debian/Ubuntu**, the privileged group is `sudo` instead of `wheel`:
> ```bash
> sudo usermod -aG sudo user
> ```

---

## 📖 Extended Examples

### Example 1 — Grant Sudo Access (RHEL / CentOS / Fedora)

```bash
sudo usermod -aG wheel devuser
```

Verify:

```bash
groups devuser
```

```bash
devuser : devuser wheel
```

---

### Example 2 — Grant Sudo Access (Debian / Ubuntu)

```bash
sudo usermod -aG sudo devuser
```

Verify:

```bash
groups devuser
```

```bash
devuser : devuser sudo
```

---

### Example 3 — Verify Sudo Access Works

```bash
# Switch to the user
su - devuser

# Test sudo
sudo whoami
```

```bash
[sudo] password for devuser:
root
```

> ✅ If it prints `root`, sudo access is working correctly.

---

### Example 4 — Grant Sudo Access via `/etc/sudoers`

Use `visudo` — never edit `/etc/sudoers` directly, as syntax errors can lock you out.

```bash
sudo visudo
```

Add this line at the end of the file:

```bash
devuser ALL=(ALL) ALL
```

#### Sudoers Rule Syntax

```
devuser   ALL=(ALL)   ALL
│         │    │       │
│         │    │       └── Commands allowed (ALL = everything)
│         │    └────────── Users/groups to run as (ALL = any user)
│         └─────────────── Hosts this applies to (ALL = any host)
└───────────────────────── Username or %groupname
```

---

### Example 5 — Grant Sudo Access Without Password Prompt

```bash
sudo visudo
```

```bash
devuser ALL=(ALL) NOPASSWD: ALL
```

> ⚠️ Use `NOPASSWD` only for service accounts or automation scripts — not for interactive admin users, as it removes an important security checkpoint.

---

### Example 6 — Grant Sudo Access to a Group

```bash
sudo visudo
```

```bash
%devgroup ALL=(ALL) ALL
```

> 💡 Prefix with `%` to target a **group** instead of an individual user. All members of `devgroup` will have sudo access.

---

### Example 7 — Restrict Sudo to Specific Commands Only

```bash
sudo visudo
```

```bash
devuser ALL=(ALL) /usr/bin/systemctl, /usr/bin/apt
```

Now `devuser` can only run `systemctl` and `apt` with sudo — nothing else.

```bash
# Allowed
sudo systemctl restart nginx

# Denied
sudo rm -rf /etc
```

---

### Example 8 — Grant Sudo for Specific Commands Without Password

```bash
sudo visudo
```

```bash
devuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl status, /usr/bin/journalctl
```

> Useful for monitoring scripts that need to check service status without interactive password prompts.

---

### Example 9 — Check Who Has Sudo Privileges

```bash
# List members of the sudo/wheel group
grep -Po '^sudo.+:\K.*$' /etc/group        # Debian/Ubuntu
grep -Po '^wheel.+:\K.*$' /etc/group       # RHEL/CentOS/Fedora
```

```bash
# Or list all sudoers rules
sudo cat /etc/sudoers
sudo ls /etc/sudoers.d/
```

---

### Example 10 — Add a Sudoers Drop-in File (Best Practice)

Instead of editing `/etc/sudoers` directly, add a file to `/etc/sudoers.d/`:

```bash
sudo visudo -f /etc/sudoers.d/devuser
```

```bash
devuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl
```

```bash
# Set correct permissions
sudo chmod 0440 /etc/sudoers.d/devuser
```

> 💡 Drop-in files keep the main `sudoers` file clean and make it easy to manage per-user or per-team policies.

---

### Example 11 — Revoke Sudo Access

```bash
# Method 1: Remove from the sudo/wheel group
sudo gpasswd -d devuser wheel      # RHEL/CentOS
sudo gpasswd -d devuser sudo       # Debian/Ubuntu

# Method 2: Comment out the rule in sudoers
sudo visudo
# devuser ALL=(ALL) ALL   ← add # to disable

# Method 3: Delete the drop-in file
sudo rm /etc/sudoers.d/devuser
```

---

## 🗂️ `sudo` Group by Distro

| Distribution | Sudo Group | Command |
|-------------|-----------|---------|
| Ubuntu / Debian | `sudo` | `sudo usermod -aG sudo username` |
| RHEL / CentOS / Fedora | `wheel` | `sudo usermod -aG wheel username` |
| Arch Linux | `wheel` | `sudo usermod -aG wheel username` |
| openSUSE | `wheel` | `sudo usermod -aG wheel username` |

---

## 🔐 `/etc/sudoers` Rule Reference

### Full Syntax

```bash
WHO  WHERE=(AS_WHOM)  NOPASSWD:  COMMANDS
```

| Token | Example | Meaning |
|-------|---------|---------|
| `WHO` | `devuser` | A specific user |
| `WHO` | `%devgroup` | All members of a group |
| `WHERE` | `ALL` | Any host |
| `AS_WHOM` | `(ALL)` | Run as any user |
| `AS_WHOM` | `(root)` | Run as root only |
| `COMMANDS` | `ALL` | All commands |
| `COMMANDS` | `/usr/bin/apt` | A specific command |
| `NOPASSWD:` | Optional | Skip password prompt |

### Common Patterns

```bash
# Full access with password
devuser ALL=(ALL) ALL

# Full access without password
devuser ALL=(ALL) NOPASSWD: ALL

# Specific commands only
devuser ALL=(ALL) /usr/bin/systemctl, /usr/bin/journalctl

# Group-based full access
%wheel ALL=(ALL) ALL

# Group-based, passwordless specific command
%devops ALL=(ALL) NOPASSWD: /usr/bin/docker
```

---

## 🚀 Full Workflow — Grant and Verify Sudo Access

```bash
# Step 1: Create the user (if not already existing)
sudo useradd -m -s /bin/bash devuser
sudo passwd devuser

# Step 2: Grant sudo access
sudo usermod -aG wheel devuser        # RHEL/CentOS/Fedora
# OR
sudo usermod -aG sudo devuser         # Debian/Ubuntu

# Step 3: Verify group membership
groups devuser

# Step 4: Test sudo as the user
su - devuser
sudo whoami        # Should print: root

# Step 5 (Optional): Restrict to specific commands
sudo visudo -f /etc/sudoers.d/devuser
# Add: devuser ALL=(ALL) /usr/bin/systemctl
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Editing `/etc/sudoers` directly with `nano`/`vim` | Always use `visudo` — it validates syntax before saving |
| Forgetting `-a` in `usermod -aG` | Without `-a`, all existing groups are **overwritten** |
| Changes not taking effect | User must **log out and back in** for new group membership to apply |
| Using `NOPASSWD: ALL` for admin users | Restrict to specific commands or avoid entirely for security |
| Syntax error in sudoers | Use `visudo` — it catches errors and prevents lockout |
| Wrong group name per distro | Use `wheel` on RHEL/Arch, `sudo` on Debian/Ubuntu |

---

## 🔑 Key Takeaways

- `usermod -aG wheel user` is the fastest way to grant full sudo on RHEL-based systems; use `sudo` group on Debian/Ubuntu.
- Always use **`visudo`** to edit `/etc/sudoers` — never a plain text editor.
- Prefix group names with `%` in sudoers rules: `%wheel ALL=(ALL) ALL`.
- Use **drop-in files** in `/etc/sudoers.d/` for clean, maintainable per-user rules.
- `NOPASSWD` is convenient but reduces security — restrict it to specific commands when possible.
- The user must **re-login** after being added to the sudo/wheel group for the change to take effect.

---

## 📚 Related Concepts

- [sudo manual](https://www.sudo.ws/docs/man/sudo.man/)
- [sudoers manual](https://www.sudo.ws/docs/man/sudoers.man/)
- [Linux PAM](https://www.linux-pam.org/) — authentication stack that sudo hooks into
- [polkit](https://www.freedesktop.org/software/polkit/docs/latest/) — alternative privilege escalation for desktop environments

</details>
