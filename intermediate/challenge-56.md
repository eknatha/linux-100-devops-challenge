# Challenge 56: Set Password Policy in Linux 

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

## 📌 Overview 

**Password policy** in Linux controls how passwords age, expire, and are enforced across user accounts. The primary tool for managing password aging is `chage` (**ch**ange **age**), which works by modifying the `/etc/shadow` file. A strong password policy is a critical pillar of Linux system security.

---

## 🧩 Task

Set a password expiry of 90 days for a user.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
sudo chage -M 90 user
```

### How it works

| Part | Description |
|------|-------------|
| `sudo` | Run with superuser privileges |
| `chage` | Change user password expiry information |
| `-M 90` | Set the **maximum** number of days a password remains valid |
| `user` | The target username to apply the policy to |

---

## 📖 Extended Examples

### Example 1 — Set Maximum Password Age (Expiry)

```bash
sudo chage -M 90 devuser
```

> Password must be changed every **90 days**. After 90 days, the user is forced to set a new password on next login.

---

### Example 2 — Set Minimum Password Age

```bash
sudo chage -m 7 devuser
```

> The user must wait at least **7 days** before changing their password again. Prevents users from cycling back to old passwords immediately.

---

### Example 3 — Set Password Expiry Warning Period

```bash
sudo chage -W 14 devuser
```

> The user receives a **warning 14 days before** their password expires at login.

---

### Example 4 — Set Account Inactivity Lock

```bash
sudo chage -I 30 devuser
```

> If the user does not log in for **30 days after password expiry**, the account is automatically **locked**.

---

### Example 5 — Set Account Expiration Date

```bash
sudo chage -E 2025-12-31 devuser
```

> The account will be **completely disabled** on December 31, 2025, regardless of password status.

```bash
# To remove the expiration date (never expires)
sudo chage -E -1 devuser
```

---

### Example 6 — Set a Full Password Policy in One Command

```bash
sudo chage -m 7 -M 90 -W 14 -I 30 devuser
```

| Flag | Value | Meaning |
|------|-------|---------|
| `-m` | `7` | Min 7 days before password can be changed |
| `-M` | `90` | Max 90 days before password must be changed |
| `-W` | `14` | Warn user 14 days before expiry |
| `-I` | `30` | Lock account 30 days after expiry if not changed |

---

### Example 7 — View Current Password Policy for a User

```bash
sudo chage -l devuser
```

```bash
Last password change                    : Mar 01, 2025
Password expires                        : May 30, 2025
Password inactive                       : Jun 29, 2025
Account expires                         : never
Minimum number of days between changes  : 7
Maximum number of days between changes  : 90
Number of days of warning before expiry : 14
```

---

### Example 8 — Force User to Change Password on Next Login

```bash
sudo chage -d 0 devuser
```

> Setting the last password change date to `0` (epoch) **forces an immediate password reset** on next login.

```bash
# Alternative using passwd
sudo passwd --expire devuser
```

---

### Example 9 — Set Password Policy Interactively

```bash
sudo chage devuser
```

```bash
Changing the aging information for devuser
Enter the new value, or press ENTER for the default

	Minimum Password Age [7]: 7
	Maximum Password Age [90]: 90
	Last Password Change (YYYY-MM-DD) [2025-03-01]:
	Password Expiration Warning [14]: 14
	Password Inactive [-1]: 30
	Account Expiration Date (YYYY-MM-DD) [-1]:
```

> Running `chage` without flags launches an **interactive prompt** to configure all fields step by step.

---

### Example 10 — Apply System-Wide Default Policy (for New Users)

Edit `/etc/login.defs` to set defaults for all newly created users:

```bash
sudo nano /etc/login.defs
```

```bash
# /etc/login.defs — System-wide password aging defaults

PASS_MAX_DAYS   90     # Maximum password age
PASS_MIN_DAYS   7      # Minimum password age
PASS_WARN_AGE   14     # Warning days before expiry
```

> 💡 These defaults only apply to **new users** created after editing the file. Use `chage` to update existing users.

---

### Example 11 — Enforce Password Complexity with PAM

For stricter policies (length, character types), configure PAM:

```bash
sudo apt install libpam-pwquality   # Debian/Ubuntu
sudo nano /etc/security/pwquality.conf
```

```bash
# /etc/security/pwquality.conf

minlen = 12          # Minimum password length
dcredit = -1         # At least 1 digit
ucredit = -1         # At least 1 uppercase letter
lcredit = -1         # At least 1 lowercase letter
ocredit = -1         # At least 1 special character
retry = 3            # Allow 3 attempts before error
```

---

## 🗂️ `chage` Flags — Quick Reference

| Flag | Long Option | Description |
|------|-------------|-------------|
| `-m days` | `--mindays` | Minimum days between password changes |
| `-M days` | `--maxdays` | Maximum days before password must be changed |
| `-W days` | `--warndays` | Days of warning before password expires |
| `-I days` | `--inactive` | Days after expiry before account is locked |
| `-E date` | `--expiredate` | Date when the account expires (YYYY-MM-DD) |
| `-d date` | `--lastday` | Set last password change date (use `0` to force reset) |
| `-l` | `--list` | Display current password aging information |

---

## 🔐 Password Aging Lifecycle

```
 Password Set
      │
      ▼
 [Minimum Age: -m]
 User cannot change password yet
      │
      ▼
 [Free Change Window]
 User can change password anytime
      │
      ▼
 [Warning Period: -W]
 User is warned at login
      │
      ▼
 [Maximum Age: -M] ◄── Password EXPIRES here
      │
      ▼
 [Inactive Period: -I]
 Account still accessible, but locked soon
      │
      ▼
 Account LOCKED — must contact admin
```

---

## 🗝️ Related Files

| File | Purpose |
|------|---------|
| `/etc/shadow` | Stores encrypted passwords and aging data per user |
| `/etc/login.defs` | System-wide default password policy for new users |
| `/etc/pam.d/common-password` | PAM configuration for password complexity rules |
| `/etc/security/pwquality.conf` | Password quality/complexity settings (via PAM) |

```bash
# View raw aging data in /etc/shadow
# Format: username:password:lastchange:min:max:warn:inactive:expire
sudo grep devuser /etc/shadow
```

```bash
devuser:$6$hash...:19783:7:90:14:30::
#                  ^^^^^ ^^ ^^ ^^ ^^
#                  last  mn mx wn in
#                  chng  dy dy dy active
```

---

## 🚀 Full Workflow — Apply a Secure Password Policy

```bash
# Step 1: Create the user
sudo useradd -m -s /bin/bash devuser

# Step 2: Set initial password
sudo passwd devuser

# Step 3: Apply password aging policy
sudo chage -m 7 -M 90 -W 14 -I 30 devuser

# Step 4: Force password reset on first login
sudo chage -d 0 devuser

# Step 5: Verify the policy
sudo chage -l devuser
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Policy not applying to existing users | `chage` applies per-user; update each user or use a script |
| `/etc/login.defs` changes don't affect existing users | Use `chage` to update existing accounts individually |
| Setting `-M 0` | Means password expires immediately — avoid unless intentional |
| Locking yourself out | Always test policy on a non-admin user first |
| Account expiry vs password expiry | `-E` disables the account completely; `-M` only expires the password |

---

## 🔑 Key Takeaways

- `chage -M` sets the **maximum password age** — the most commonly enforced policy.
- `chage -m` sets a **minimum age** to prevent immediate password recycling.
- `chage -W` adds a **warning period** so users aren't surprised by sudden lockouts.
- `chage -d 0` **forces a password reset** on the very next login.
- `/etc/login.defs` sets **system-wide defaults** for new users only.
- For **password complexity** (length, special characters), use PAM with `pwquality`.
- Always verify applied policies with `sudo chage -l username`.

---

## 📚 Related Concepts

- [Linux PAM Documentation](https://www.linux-pam.org/Linux-PAM-html/)
- [passwd command](https://linux.die.net/man/1/passwd)
- [login.defs manual](https://man7.org/linux/man-pages/man5/login.defs.5.html)
- [chage manual](https://man7.org/linux/man-pages/man1/chage.1.html)

</details>
