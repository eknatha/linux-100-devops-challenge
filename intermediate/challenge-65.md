# Challenge 65: Restrict SSH Access in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**SSH (Secure Shell)** is the primary remote access protocol on Linux servers. While powerful, an improperly configured SSH service is one of the most common attack vectors — automated bots constantly scan the internet probing for weak SSH configurations.

Hardening SSH access — starting with **disabling root login** — is a foundational step in securing any Linux server. The main configuration file is `/etc/ssh/sshd_config`, which controls the behavior of the SSH daemon (`sshd`).

---

## 🧩 Task

Disable root login via SSH to prevent direct root access to the server.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Open the SSH daemon config file
sudo vi /etc/ssh/sshd_config

# Find and set this line
PermitRootLogin no
```

Then restart SSH to apply:

```bash
sudo systemctl restart sshd
```

### How it works

| Part | Description |
|------|-------------|
| `/etc/ssh/sshd_config` | The SSH **daemon** configuration file (server-side settings) |
| `PermitRootLogin no` | Prevents the `root` user from logging in via SSH entirely |
| `systemctl restart sshd` | Applies the new configuration by restarting the SSH service |

> ⚠️ Always ensure **another sudo-capable user exists** before disabling root login — otherwise you may lock yourself out.

---

## 📖 Extended Examples

### Example 1 — Disable Root SSH Login

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
# Find this line and change it:
PermitRootLogin no
```

```bash
# Apply the change
sudo systemctl restart sshd
```

Verify it is active:

```bash
sudo sshd -T | grep permitrootlogin
```

```bash
permitrootlogin no
```

---

### Example 2 — `PermitRootLogin` Options Explained

```bash
# Option 1: Block root login entirely (recommended)
PermitRootLogin no

# Option 2: Allow root login with key only (no password)
PermitRootLogin prohibit-password

# Option 3: Allow root login only if a 'forced command' is set in authorized_keys
PermitRootLogin forced-commands-only

# Option 4: Allow root login with any method (default — insecure)
PermitRootLogin yes
```

| Value | Meaning | Recommended |
|-------|---------|-------------|
| `no` | Root login completely blocked | ✅ Best |
| `prohibit-password` | Root can use key-based auth only | ✅ Good |
| `forced-commands-only` | Root only for specific commands | ⚠️ Situational |
| `yes` | Root login allowed with any method | ❌ Avoid |

---

### Example 3 — Disable Password Authentication (Keys Only)

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
PasswordAuthentication no
PubkeyAuthentication yes
```

```bash
sudo systemctl restart sshd
```

> 💡 Disabling password auth forces all users to use SSH keys — eliminates brute-force password attacks entirely.

---

### Example 4 — Allow Only Specific Users

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
# Only these users can log in via SSH
AllowUsers devuser adminuser john

# Or allow an entire group
AllowGroups sshusers developers
```

```bash
sudo systemctl restart sshd
```

> 💡 `AllowUsers` and `AllowGroups` create a **whitelist** — any user not listed is denied SSH access even if they have a valid account and key.

---

### Example 5 — Deny Specific Users or Groups

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
# Block specific users from SSH login
DenyUsers baduser tempuser

# Block an entire group from SSH login
DenyGroups noremote contractors
```

> `DenyUsers` and `DenyGroups` create a **blacklist** — all other users are allowed unless restricted elsewhere.

---

### Example 6 — Change the Default SSH Port

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
Port 2222
```

```bash
sudo systemctl restart sshd
```

```bash
# Connect using the custom port
ssh -p 2222 user@remote-server
```

> 💡 Changing the default port from `22` significantly reduces automated bot scan noise — not a security measure by itself, but reduces log pollution.

> ⚠️ Remember to update firewall rules to allow the new port before restarting SSH.

---

### Example 7 — Limit Authentication Attempts

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
MaxAuthTries 3          # Max failed attempts before disconnecting
LoginGraceTime 30       # Seconds to complete login before timeout
MaxSessions 5           # Max concurrent sessions per connection
MaxStartups 10:30:60    # Throttle unauthenticated connections
```

| Setting | Description |
|---------|-------------|
| `MaxAuthTries 3` | Disconnect after 3 failed auth attempts |
| `LoginGraceTime 30` | Timeout unauthenticated connections after 30s |
| `MaxSessions 5` | Max multiplexed sessions per connection |
| `MaxStartups 10:30:60` | Start refusing at 10 pending, drop 30%, hard stop at 60 |

---

### Example 8 — Restrict SSH to Specific IP Addresses (Using `sshd_config`)

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
# Allow SSH only from a specific subnet
ListenAddress 192.168.1.0
```

Or use `Match Address` blocks for fine-grained control:

```bash
# Global defaults
PasswordAuthentication no

# Allow password auth from office IP only
Match Address 192.168.1.50
    PasswordAuthentication yes
```

---

### Example 9 — Restrict SSH via TCP Wrappers (`/etc/hosts.allow` and `/etc/hosts.deny`)

```bash
# Allow SSH only from a specific IP or subnet
echo "sshd: 192.168.1.0/24" | sudo tee -a /etc/hosts.allow

# Deny SSH from all other hosts
echo "sshd: ALL" | sudo tee -a /etc/hosts.deny
```

```bash
# /etc/hosts.allow — takes priority over hosts.deny
sshd: 192.168.1.0/24
sshd: 10.0.0.5

# /etc/hosts.deny — deny everyone else
sshd: ALL
```

---

### Example 10 — Restrict SSH via UFW Firewall (Recommended)

```bash
# Allow SSH from a specific IP only
sudo ufw allow from 192.168.1.50 to any port 22

# Allow SSH on a custom port
sudo ufw allow 2222/tcp

# Deny SSH from all other sources
sudo ufw deny 22

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

---

### Example 11 — Disable X11 Forwarding and Other Unused Features

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
X11Forwarding no              # Disable graphical forwarding
AllowAgentForwarding no       # Disable agent forwarding
AllowTcpForwarding no         # Disable TCP tunneling
PermitTunnel no               # Disable VPN-like tunneling
PrintMotd no                  # Suppress message of the day
Banner none                   # No pre-login banner (hides server info)
```

> 💡 Disable features you don't use — every enabled feature is a potential attack surface.

---

### Example 12 — Full Hardened `sshd_config` Template

```bash
sudo vi /etc/ssh/sshd_config
```

```bash
# ── Protocol and Port ────────────────────────────────────
Port 2222
Protocol 2
AddressFamily inet

# ── Authentication ───────────────────────────────────────
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# ── Access Control ───────────────────────────────────────
AllowUsers devuser adminuser
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 5
MaxStartups 10:30:60

# ── Feature Restrictions ─────────────────────────────────
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
PrintMotd no
Banner none

# ── Keepalive ────────────────────────────────────────────
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
# Validate config before restarting (prevents lockout from typos)
sudo sshd -t && echo "✅ Config is valid"

# Apply changes
sudo systemctl restart sshd
```

---

## 🔄 `sshd_config` Key Directives — Quick Reference

| Directive | Recommended Value | Description |
|-----------|------------------|-------------|
| `Port` | `2222` (custom) | Change from default 22 |
| `PermitRootLogin` | `no` | Disable root SSH login |
| `PasswordAuthentication` | `no` | Keys only |
| `PubkeyAuthentication` | `yes` | Enable key-based auth |
| `PermitEmptyPasswords` | `no` | Block empty passwords |
| `AllowUsers` | specific users | Whitelist SSH users |
| `MaxAuthTries` | `3` | Limit failed attempts |
| `LoginGraceTime` | `30` | Timeout unauthenticated sessions |
| `X11Forwarding` | `no` | Disable GUI forwarding |
| `AllowTcpForwarding` | `no` | Disable tunneling |
| `ClientAliveInterval` | `300` | Idle session keepalive |
| `ClientAliveCountMax` | `2` | Max missed keepalives before disconnect |

---

## 🛡️ SSH Hardening Priority — Where to Start

```
 Priority   Action
 ────────   ──────────────────────────────────────────────
 🔴 High    PermitRootLogin no
 🔴 High    PasswordAuthentication no   (use keys only)
 🔴 High    AllowUsers <specific users>
 🟠 Medium  Change Port from 22 to custom
 🟠 Medium  MaxAuthTries 3
 🟠 Medium  LoginGraceTime 30
 🟡 Low     X11Forwarding no
 🟡 Low     AllowTcpForwarding no
 🟡 Low     ClientAliveInterval + CountMax
```

---

## 🚀 Full Workflow — Harden SSH Access on a New Server

```bash
# Step 1: Create a sudo user before disabling root
sudo useradd -m -s /bin/bash devuser
sudo usermod -aG wheel devuser       # RHEL/CentOS
sudo passwd devuser

# Step 2: Set up SSH key for the new user (from local machine)
ssh-copy-id devuser@server-ip

# Step 3: Test key login in a NEW terminal before any changes
ssh devuser@server-ip

# Step 4: Edit sshd_config
sudo vi /etc/ssh/sshd_config
# Set: PermitRootLogin no
# Set: PasswordAuthentication no
# Set: AllowUsers devuser

# Step 5: Validate the config
sudo sshd -t && echo "Config OK"

# Step 6: Restart SSH
sudo systemctl restart sshd

# Step 7: Test login again in a new terminal
ssh devuser@server-ip

# Step 8: (Optional) Restrict via firewall
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw enable
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Locking yourself out | Always test in a **new terminal** before closing your current SSH session |
| Typo in `sshd_config` breaks SSH | Always run `sudo sshd -t` to validate before restarting |
| Disabling root before creating another user | Create and test a sudo user **first** |
| Forgetting to restart `sshd` after changes | Run `sudo systemctl restart sshd` — changes are not live until restarted |
| Port change blocked by firewall | Open the new port in UFW/firewalld before restarting SSH |
| `AllowUsers` blocks all users | Double-check the username spelling — one typo locks everyone out |

---

## 🔑 Key Takeaways

- `PermitRootLogin no` is the **first and most important** SSH hardening step.
- Always validate config with `sudo sshd -t` before restarting to avoid lockouts.
- **Test in a new terminal** after every change — never close a working session until the new config is confirmed.
- `PasswordAuthentication no` + `AllowUsers` together form the most effective access control.
- `AllowUsers` is a whitelist — only listed users can SSH in, regardless of other settings.
- Defense in depth: combine `sshd_config` restrictions with **firewall rules** for layered security.

---

## 📚 Related Concepts

- [sshd_config Manual](https://linux.die.net/man/5/sshd_config)
- [fail2ban](https://www.fail2ban.org/) — automatically bans IPs with repeated failed SSH attempts
- [UFW Documentation](https://help.ubuntu.com/community/UFW) — Uncomplicated Firewall for IP-level SSH restrictions
</details>
