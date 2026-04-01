# Challenge 83: Debug SSH Login Issues in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**SSH login failures** are one of the most common and frustrating issues in Linux administration. They can stem from many different causes — misconfigured `sshd_config`, wrong file permissions, expired keys, firewall rules, or DNS resolution problems. Knowing where to look and what the log messages mean is the fastest path to resolution.

The two primary sources for SSH debugging are:
- **`journalctl -u sshd`** — the systemd journal for the SSH daemon (modern, structured, filterable)
- **`/var/log/secure`** (RHEL) / **`/var/log/auth.log`** (Debian) — the traditional authentication log file

---

## 🧩 Task

Diagnose and resolve SSH login failures by reading and interpreting authentication logs.

---


<details>
<summary>💡 Click to view solution</summary>



## ✅ Solution

```bash
# View SSH daemon logs via systemd journal
journalctl -u sshd

# View authentication log (RHEL/CentOS/Fedora)
cat /var/log/secure

# View authentication log (Debian/Ubuntu)
cat /var/log/auth.log
```

### How it works

| Command | Description |
|---------|-------------|
| `journalctl -u sshd` | Filter journal to the **sshd** service unit only |
| `-u sshd` | Show logs from the SSH daemon (`sshd.service`) |
| `/var/log/secure` | Auth log on **RHEL/CentOS** — all auth events including SSH |
| `/var/log/auth.log` | Auth log on **Debian/Ubuntu** — equivalent to `/var/log/secure` |

---

## 🗂️ SSH Log Locations by Distribution

| Distribution | Journal Command | Log File |
|-------------|----------------|----------|
| RHEL / CentOS / Fedora | `journalctl -u sshd` | `/var/log/secure` |
| Debian / Ubuntu | `journalctl -u ssh` | `/var/log/auth.log` |
| Arch Linux | `journalctl -u sshd` | Journal only |
| Any systemd distro | `journalctl -u sshd` | Both journal and flat file |

> 💡 Note: on Debian/Ubuntu the service is `ssh` (not `sshd`):
> ```bash
> journalctl -u ssh
> ```

---

## 📖 Extended Examples

### Example 1 — View SSH Daemon Logs

```bash
# All sshd logs
journalctl -u sshd

# Last 50 lines
journalctl -u sshd -n 50

# Follow in real time while testing a login
journalctl -u sshd -f

# Current boot only
journalctl -u sshd -b

# Only errors
journalctl -u sshd -p err
```

---

### Example 2 — Common SSH Error Messages and What They Mean

```bash
# Error 1: Permission denied (publickey)
Mar 21 10:00 hostname sshd[1024]: Connection closed by authenticating user devuser 192.168.1.5 port 54312 [preauth]

# Error 2: Authentication refused — bad key file permissions
Mar 21 10:01 hostname sshd[1025]: Authentication refused: bad ownership or modes for directory /home/devuser/.ssh

# Error 3: No supported authentication methods
Mar 21 10:02 hostname sshd[1026]: userauth_pubkey: key type ssh-dss not in PubkeyAcceptedAlgorithms

# Error 4: Too many authentication failures
Mar 21 10:03 hostname sshd[1027]: Disconnecting authenticating user devuser: Too many authentication failures [preauth]

# Error 5: User not allowed
Mar 21 10:04 hostname sshd[1028]: User devuser from 192.168.1.5 not allowed because not listed in AllowUsers
```

---

### Example 3 — Enable Verbose SSH Client Logging

The fastest way to pinpoint a client-side problem is to run SSH with verbose output:

```bash
# Single verbose (-v)
ssh -v user@hostname

# Double verbose (-vv)
ssh -vv user@hostname

# Maximum verbosity (-vvv)
ssh -vvv user@hostname
```

```bash
# -vvv output example (permission denied):
debug1: Offering public key: /home/user/.ssh/id_ed25519
debug1: Server accepts key: /home/user/.ssh/id_ed25519 ED25519 SHA256:abc...
debug1: Authentication that can continue: publickey
debug1: Trying private key: /home/user/.ssh/id_ed25519
debug3: sign_and_send_pubkey: using private key "/home/user/.ssh/id_ed25519"
debug3: send packet: type 50
debug1: Authentications that can continue: publickey
debug2: we did not send a packet, disable method
debug1: No more authentication methods to try.
user@hostname: Permission denied (publickey).
```

> 💡 `-vvv` shows exactly which key is being offered, whether the server accepts it, and at which stage authentication fails.

---

### Example 4 — Fix Permission Denied (publickey)

This is the most common SSH issue. Check in this order:

```bash
# On the REMOTE server:

# Step 1: Check authorized_keys exists and has correct content
cat ~/.ssh/authorized_keys

# Step 2: Check file and directory permissions (critical — sshd is strict)
ls -la ~/.ssh/
ls -la ~/.ssh/authorized_keys

# Correct permissions must be:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Step 3: Check home directory permissions
ls -la /home/
# Home directory must NOT be world-writable:
chmod 755 /home/devuser
```

```bash
# Required permissions:
~/.ssh/              → 700  (drwx------)
~/.ssh/authorized_keys → 600  (-rw-------)
~/                   → 755  (drwxr-xr-x) or more restrictive
```

> ⚠️ SSH will silently **refuse to use** `authorized_keys` if any of these permissions are too permissive — even `chmod 644` on `authorized_keys` causes a failure.

---

### Example 5 — Check the sshd Configuration

```bash
# Check the sshd config for misconfigurations
sudo cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"

# Validate sshd_config syntax before applying
sudo sshd -t

# Check if PasswordAuthentication is enabled
grep "PasswordAuthentication" /etc/ssh/sshd_config

# Check if PubkeyAuthentication is enabled
grep "PubkeyAuthentication" /etc/ssh/sshd_config

# Check if a user is in AllowUsers or blocked by DenyUsers
grep -E "AllowUsers|DenyUsers|AllowGroups|DenyGroups" /etc/ssh/sshd_config
```

```bash
# Common misconfigurations that cause login failures:
PasswordAuthentication no    ← User tries password but it's disabled
PubkeyAuthentication no      ← User tries key but keys are disabled
AllowUsers john mary         ← devuser not listed → access denied
PermitRootLogin no           ← Root login attempt fails
```

---

### Example 6 — Check SSH Host Key Issues

When connecting to a server for the first time or after a server rebuild, SSH may refuse connection due to a known hosts mismatch:

```bash
# Client-side error:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

```bash
# Fix: remove the old host key entry
ssh-keygen -R hostname
ssh-keygen -R 192.168.1.100

# Or manually edit known_hosts
nano ~/.ssh/known_hosts
# Delete the line for the affected host

# Then reconnect (you will be asked to confirm the new key)
ssh user@hostname
```

> ⚠️ Only remove the known_hosts entry if you are **certain** the server was legitimately rebuilt or its key changed — the warning can indicate a man-in-the-middle attack.

---

### Example 7 — Debug SELinux / AppArmor Blocking SSH

On RHEL/CentOS with SELinux or Ubuntu with AppArmor, security policies can block SSH even when permissions look correct:

```bash
# Check for SELinux denials related to SSH
sudo ausearch -m AVC -c sshd 2>/dev/null | tail -20

# Check SELinux context of authorized_keys
ls -Z ~/.ssh/authorized_keys

# Fix wrong SELinux context on .ssh directory
restorecon -Rv ~/.ssh/

# Check if SELinux is the cause
sudo setenforce 0        # Temporarily disable (test only — not for production)
ssh user@hostname        # If this now works, SELinux was blocking it
sudo setenforce 1        # Re-enable SELinux
```

```bash
# Example SELinux denial:
type=AVC msg=audit(1711017301): avc: denied { read } for
  pid=4821 comm="sshd" name="authorized_keys" scontext=system_u:system_r:sshd_t
  tcontext=unconfined_u:object_r:user_home_t:s0
```

---

### Example 8 — Check Firewall and Network Connectivity

```bash
# Test if SSH port is reachable from client
nc -zv hostname 22
telnet hostname 22

# Check the firewall on the server
sudo firewall-cmd --list-all | grep ssh       # firewalld (RHEL)
sudo ufw status | grep 22                      # ufw (Ubuntu)
sudo iptables -L -n | grep 22                  # iptables

# Check if sshd is listening
sudo ss -tlnp | grep ':22'
sudo ss -tlnp | grep ':2222'   # If custom port
```

```bash
# If nc/telnet fails:
nc: connect to hostname port 22 (tcp) failed: Connection refused
# → sshd is not running or firewall is blocking port 22

# If sshd not listening:
sudo systemctl status sshd
sudo systemctl start sshd
```

---

### Example 9 — Check SSH Key Type Compatibility

Modern SSH servers may reject older key types:

```bash
# Check which key algorithms the server accepts
ssh -vvv user@hostname 2>&1 | grep "PubkeyAcceptedAlgorithms\|Server accepts"

# On the server — view accepted algorithms
sudo sshd -T | grep pubkeyacceptedalgorithms

# Generate a modern Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "mykey@host"

# If stuck with an old server that needs RSA
ssh-keygen -t rsa -b 4096
```

```bash
# Common compatibility error:
sshd[4821]: userauth_pubkey: key type ssh-dss not in PubkeyAcceptedAlgorithms
# → DSA keys are disabled on modern servers — generate an Ed25519 or RSA-4096 key
```

---

### Example 10 — Debug sshd in Verbose Mode (Server-Side)

When client `-vvv` isn't enough, run a debug instance of sshd on a different port:

```bash
# Launch sshd in debug mode on port 2222 (no impact on production SSH)
sudo /usr/sbin/sshd -d -p 2222

# Connect from the client using the debug port
ssh -p 2222 user@hostname
```

```bash
# Debug sshd output example:
debug1: userauth_pubkey: test whether pkalg/pkblob are acceptable for RSA SHA256:abc
debug1: temporarily_use_uid: 1001/1001 (e=0/0)
debug1: trying public key file /home/devuser/.ssh/authorized_keys
debug1: Could not open authorized keys '/home/devuser/.ssh/authorized_keys': Permission denied
#                                                                             ^^^ root cause found!
```

> 💡 Running sshd in debug mode shows **exactly what it is checking** — file paths, permission tests, key matching — making it the most precise debugging method.

---

### Example 11 — SSH Config File Troubleshooting

```bash
# Check the client-side SSH config
cat ~/.ssh/config

# Validate the config
ssh -G user@hostname | head -20
```

```bash
# Common client config issues:
Host myserver
    HostName 192.168.1.100
    User devuser
    IdentityFile ~/.ssh/id_ed25519_OLD    ← Key file that no longer exists
    Port 22
```

```bash
# Check if the specified identity file exists
ls -la ~/.ssh/id_ed25519_OLD
# No such file or directory → SSH silently falls back or fails
```

---

### Example 12 — SSH Debugging Checklist Script

```bash
#!/bin/bash

TARGET_HOST="${1:-localhost}"
TARGET_USER="${2:-$USER}"
SSH_PORT="${3:-22}"
KEY_FILE="${4:-$HOME/.ssh/id_ed25519}"

echo "============================================"
echo " SSH Debug Checklist"
echo " Target: $TARGET_USER@$TARGET_HOST:$SSH_PORT"
echo "============================================"

echo -e "\n[1] Client-side key file exists?"
[ -f "$KEY_FILE" ] && echo "✅ $KEY_FILE exists" || echo "❌ Key file not found: $KEY_FILE"

echo -e "\n[2] Client key permissions:"
if [ -f "$KEY_FILE" ]; then
  PERM=$(stat -c "%a" "$KEY_FILE")
  [ "$PERM" = "600" ] && echo "✅ Permissions: $PERM" || echo "⚠️  Permissions: $PERM (should be 600)"
fi

echo -e "\n[3] Can we reach the SSH port?"
nc -zv -w 3 "$TARGET_HOST" "$SSH_PORT" 2>&1 && echo "✅ Port $SSH_PORT reachable" || echo "❌ Port $SSH_PORT unreachable"

echo -e "\n[4] SSH verbose test (first 30 lines):"
ssh -vvv -o ConnectTimeout=5 -o BatchMode=yes \
  -i "$KEY_FILE" -p "$SSH_PORT" "$TARGET_USER@$TARGET_HOST" exit 2>&1 | head -30

echo -e "\n[5] Server SSH service status:"
ssh -p "$SSH_PORT" "$TARGET_USER@$TARGET_HOST" \
  "sudo systemctl is-active sshd && sudo sshd -T | grep -E 'passwordauth|pubkeyauth|permitroot|allowusers'" 2>/dev/null \
  || echo "Could not connect to check server status"
```

---

## 🗂️ SSH Error → Cause → Fix

| Error Message | Cause | Fix |
|--------------|-------|-----|
| `Permission denied (publickey)` | Wrong key, bad permissions, key not in `authorized_keys` | Check permissions, verify key is in `authorized_keys` |
| `Authentication refused: bad ownership or modes` | `~/.ssh` or `authorized_keys` has wrong permissions | `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| `Connection refused` | sshd not running or firewall blocking | `systemctl start sshd`, open port in firewall |
| `Connection timed out` | Firewall dropping packets, wrong IP | Check firewall rules, verify IP/hostname |
| `Host key verification failed` | Server key changed since last connection | `ssh-keygen -R hostname` then reconnect |
| `Too many authentication failures` | SSH agent offering too many keys | `ssh -o IdentitiesOnly=yes -i key user@host` |
| `User not allowed because not listed in AllowUsers` | `AllowUsers` in `sshd_config` excludes the user | Add user to `AllowUsers` or remove the restriction |
| `No route to host` | Network routing issue | Check network connectivity and routing table |
| `key type ssh-dss not in PubkeyAcceptedAlgorithms` | DSA key rejected by modern sshd | Generate Ed25519 or RSA-4096 key |
| `Could not open authorized keys: Permission denied` | SELinux or wrong file ownership | `restorecon -Rv ~/.ssh/` or fix ownership |

---

## 🚀 Full Workflow — Debug an SSH Login Failure

```bash
# Step 1: Try connecting with verbose output
ssh -vvv user@hostname

# Step 2: Check server SSH logs in real time
# (on the server, in a second terminal)
journalctl -u sshd -f

# Step 3: Identify the error message and category
# "Permission denied" → check keys and permissions
# "Connection refused" → check sshd status and firewall
# "Host key changed" → run ssh-keygen -R hostname

# Step 4: Check key permissions (if key-related)
ls -la ~/.ssh/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519

# Step 5: Validate sshd config on the server
sudo sshd -t

# Step 6: Check firewall
sudo ss -tlnp | grep ':22'
sudo firewall-cmd --list-all | grep ssh

# Step 7: Check SELinux (RHEL/CentOS)
restorecon -Rv ~/.ssh/
ausearch -m AVC -c sshd

# Step 8: Test with debug sshd instance if still stuck
sudo /usr/sbin/sshd -d -p 2222
ssh -p 2222 user@hostname

# Step 9: Restart sshd after config changes
sudo systemctl restart sshd

# Step 10: Verify fix
ssh user@hostname
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Restarting sshd locks yourself out | Always test in a **new terminal** before closing your current session |
| Fixing permissions but still failing | Check **SELinux context** with `ls -Z ~/.ssh/` and run `restorecon -Rv ~/.ssh/` |
| Too many keys in ssh-agent causing failures | Use `ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key user@host` |
| `authorized_keys` has correct content but wrong line endings | Use `cat` not a Windows editor — Windows CRLF breaks the file |
| sshd config edited but not reloaded | Run `sudo systemctl reload sshd` (or `restart`) after every config change |
| Debug sshd on port 2222 conflicts | Ensure nothing else is on port 2222 first: `ss -tlnp | grep 2222` |

---

## 🔑 Key Takeaways

- `journalctl -u sshd -f` combined with `ssh -vvv` in another terminal gives a **real-time client+server view** of exactly where the failure occurs.
- **File permissions** are the most common cause of `Permission denied (publickey)` — `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/authorized_keys` resolve the majority of cases.
- `sudo sshd -t` validates the config file syntax — always run this before restarting sshd.
- Running **`sshd -d -p 2222`** is the most powerful server-side debug technique — it logs every decision the daemon makes.
- **SELinux** silently blocks SSH even when permissions look correct — always check `restorecon -Rv ~/.ssh/` on RHEL systems.
- `ssh-keygen -R hostname` removes stale known_hosts entries — use it when the server has been rebuilt.
- Always test changes in a **new terminal** before closing your current working SSH session.

---

## 📚 Related Concepts

- [Challenge 64 — Configure SSH Keys]
- [Challenge 65 — Restrict SSH Access]
- [Challenge 82 — Unauthorized Access Check]
- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [sshd_config Reference](https://linux.die.net/man/5/sshd_config)
- [SSH Troubleshooting Guide](https://wiki.gentoo.org/wiki/SSH#Troubleshooting)


</details>
