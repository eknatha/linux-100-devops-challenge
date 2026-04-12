# Challenge 64: Configure SSH Key Authentication

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**SSH key authentication** replaces password-based login with a cryptographic key pair — a **private key** (kept secret on your local machine) and a **public key** (placed on the remote server). When you connect, the server verifies your identity using the key pair without ever transmitting a password over the network.

Key benefits over password authentication:
- **More secure** — no password to brute-force or intercept
- **Faster** — no typing required once set up
- **Automatable** — enables passwordless scripts, CI/CD pipelines, and cron jobs
- **Auditable** — each key can be named and revoked independently

---

## 🧩 Task

Generate an SSH key pair and configure passwordless login to a remote host.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Step 1: Generate the key pair
ssh-keygen

# Step 2: Copy the public key to the remote server
ssh-copy-id user@host
```

### How it works

| Command | Description |
|---------|-------------|
| `ssh-keygen` | Generates a new SSH key pair (private + public key) |
| `ssh-copy-id user@host` | Appends your public key to `~/.ssh/authorized_keys` on the remote host |

---

## 🔑 How SSH Key Authentication Works

```
  Local Machine                        Remote Server
  ─────────────                        ─────────────
  ~/.ssh/id_ed25519      (private)
  ~/.ssh/id_ed25519.pub  (public)  →   ~/.ssh/authorized_keys

  1. Client sends connection request
  2. Server sends a random challenge
  3. Client signs the challenge with the private key
  4. Server verifies the signature using the public key
  5. ✅ Access granted — no password needed
```

---

## 📖 Extended Examples

### Example 1 — Generate a Default SSH Key Pair (RSA)

```bash
ssh-keygen
```

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):   ← press Enter for default
Enter passphrase (empty for no passphrase):                      ← optional but recommended
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_rsa
Your public key has been saved in /home/user/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:abc123xyz... user@localhost
```

---

### Example 2 — Generate a More Secure Ed25519 Key (Recommended)

```bash
ssh-keygen -t ed25519 -C "devuser@myserver"
```

| Flag | Description |
|------|-------------|
| `-t ed25519` | Key type — Ed25519 is modern, fast, and more secure than RSA |
| `-C "comment"` | Comment label to identify the key (usually email or hostname) |

```bash
# Files created:
~/.ssh/id_ed25519       ← private key (keep secret, never share)
~/.ssh/id_ed25519.pub   ← public key  (safe to share)
```

---

### Example 3 — Generate a Named Key for a Specific Server

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github -C "github-key"
```

```bash
# Files created:
~/.ssh/id_ed25519_github
~/.ssh/id_ed25519_github.pub
```

> 💡 Using named keys lets you maintain **separate key pairs** for different services — GitHub, work servers, personal VPS — and revoke them independently.

---

### Example 4 — Copy Public Key to Remote Server

```bash
ssh-copy-id user@remote-server
```

```bash
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed.
user@remote-server's password:    ← enter password one last time
Number of key(s) added: 1

Now try logging into the machine with: "ssh 'user@remote-server'"
```

---

### Example 5 — Copy a Specific Key to a Remote Server

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_github.pub user@remote-server
```

> `-i` specifies which public key file to copy when you have multiple keys.

---

### Example 6 — Copy Public Key Manually (When `ssh-copy-id` Is Unavailable)

```bash
# Method 1: pipe via ssh
cat ~/.ssh/id_ed25519.pub | ssh user@remote-server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Method 2: using ssh command substitution
ssh user@remote-server "echo '$(cat ~/.ssh/id_ed25519.pub)' >> ~/.ssh/authorized_keys"
```

---

### Example 7 — Connect Using SSH Key

```bash
# Default key (auto-detected)
ssh user@remote-server

# Specify a key explicitly
ssh -i ~/.ssh/id_ed25519_github user@remote-server

# Custom port
ssh -i ~/.ssh/id_ed25519 -p 2222 user@remote-server
```

---

### Example 8 — Use SSH Config File for Easy Connections

Instead of typing long `ssh` commands, define shortcuts in `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

```bash
# ~/.ssh/config

Host myserver
    HostName 192.168.1.100
    User devuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github

Host work
    HostName work.example.com
    User john
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_work
```

```bash
# Set correct permissions on config file
chmod 600 ~/.ssh/config

# Now connect with a simple alias
ssh myserver
ssh github
ssh work
```

---

### Example 9 — Set Correct File Permissions (Critical)

SSH is strict about file permissions and **will refuse to work** if they are too permissive:

```bash
# Local machine permissions
chmod 700 ~/.ssh                        # Directory
chmod 600 ~/.ssh/id_ed25519             # Private key (owner read/write only)
chmod 644 ~/.ssh/id_ed25519.pub         # Public key
chmod 600 ~/.ssh/config                 # SSH config file

# Remote server permissions
chmod 700 ~/.ssh                        # SSH directory
chmod 600 ~/.ssh/authorized_keys        # Authorized keys file
```

> ⚠️ If permissions are wrong, SSH will print `WARNING: UNPROTECTED PRIVATE KEY FILE!` and refuse to use the key.

---

### Example 10 — Use SSH Agent to Avoid Repeated Passphrase Entry

If you set a passphrase on your key, use `ssh-agent` to cache it for the session:

```bash
# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key to the agent
ssh-add ~/.ssh/id_ed25519

# Verify keys loaded
ssh-add -l
```

```bash
Identity added: /home/user/.ssh/id_ed25519 (devuser@myserver)
256 SHA256:abc123... devuser@myserver (ED25519)
```

> Now SSH uses the agent to authenticate — you only type the passphrase **once per session**.

---

### Example 11 — Disable Password Authentication on the Server (Harden SSH)

Once key authentication is working, disable password login on the server to prevent brute-force attacks:

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
# /etc/ssh/sshd_config — security hardening

PasswordAuthentication no        # Disable password login
PubkeyAuthentication yes         # Ensure key auth is enabled
PermitRootLogin no               # Disable direct root login
AuthorizedKeysFile .ssh/authorized_keys
```

```bash
# Restart SSH to apply changes
sudo systemctl restart sshd

# ⚠️ Test in a NEW terminal before closing your current session
# to avoid locking yourself out
```

---

### Example 12 — Revoke / Remove a Key

```bash
# On the remote server, edit authorized_keys
nano ~/.ssh/authorized_keys

# Delete the line containing the key to revoke
# Each line is one public key — simply remove the unwanted one

# Or use grep to remove a specific key by comment
grep -v "old-key-comment" ~/.ssh/authorized_keys > /tmp/ak_tmp && mv /tmp/ak_tmp ~/.ssh/authorized_keys
```

---

## 🗂️ SSH Key Types — Comparison

| Type | Flag | Key Size | Security | Speed | Recommendation |
|------|------|----------|----------|-------|----------------|
| Ed25519 | `-t ed25519` | 256-bit | ⭐⭐⭐⭐⭐ | Fastest | ✅ **Recommended** |
| ECDSA | `-t ecdsa` | 256–521-bit | ⭐⭐⭐⭐ | Fast | Good alternative |
| RSA | `-t rsa` | 2048–4096-bit | ⭐⭐⭐ | Slower | Legacy compatibility |
| DSA | `-t dsa` | 1024-bit | ⭐ | — | ❌ Deprecated |

```bash
# Generate each type
ssh-keygen -t ed25519           # Recommended
ssh-keygen -t ecdsa -b 521      # Alternative
ssh-keygen -t rsa -b 4096       # For legacy systems
```

---

## 📁 SSH File Reference

| File | Location | Purpose |
|------|----------|---------|
| Private key | `~/.ssh/id_ed25519` | Your secret identity — never share |
| Public key | `~/.ssh/id_ed25519.pub` | Placed on remote servers |
| Authorized keys | `~/.ssh/authorized_keys` | Remote server's list of trusted public keys |
| Config file | `~/.ssh/config` | SSH client shortcuts and per-host settings |
| Known hosts | `~/.ssh/known_hosts` | Fingerprints of previously connected servers |

---

## 🚀 Full Workflow — Set Up Passwordless SSH from Scratch

```bash
# ── On your LOCAL machine ────────────────────────────────

# Step 1: Generate key pair
ssh-keygen -t ed25519 -C "devuser@myserver"

# Step 2: Copy public key to remote server
ssh-copy-id devuser@192.168.1.100

# Step 3: Add SSH config shortcut
echo -e "Host myserver\n  HostName 192.168.1.100\n  User devuser\n  IdentityFile ~/.ssh/id_ed25519" >> ~/.ssh/config
chmod 600 ~/.ssh/config

# Step 4: Test the connection
ssh myserver

# ── On the REMOTE server ─────────────────────────────────

# Step 5: Verify the key was added
cat ~/.ssh/authorized_keys

# Step 6: Harden SSH (disable password auth)
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd

# Step 7: Test again in a new terminal to confirm access
ssh myserver
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Permission denied (publickey) | Check permissions: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| WARNING: UNPROTECTED PRIVATE KEY FILE | Fix private key permissions: `chmod 600 ~/.ssh/id_ed25519` |
| Locked out after disabling passwords | Always test in a new terminal before closing the current SSH session |
| Wrong key used | Use `-i ~/.ssh/specific_key` or configure `IdentityFile` in `~/.ssh/config` |
| Known hosts mismatch warning | Server fingerprint changed — run `ssh-keygen -R hostname` to clear it |
| `ssh-copy-id` not available | Use the manual `cat ~/.ssh/id.pub | ssh user@host "cat >> ~/.ssh/authorized_keys"` method |

---

## 🔑 Key Takeaways

- `ssh-keygen -t ed25519` is the modern, recommended way to generate SSH keys.
- `ssh-copy-id user@host` copies your public key to the remote server in one command.
- **Never share your private key** — only the `.pub` file goes on remote servers.
- SSH is strict about permissions — `chmod 600` on private keys and `authorized_keys` is mandatory.
- Use `~/.ssh/config` to define aliases — connect with `ssh myserver` instead of long commands.
- Disable `PasswordAuthentication` on servers once key auth is confirmed working.
- Use `ssh-agent` to cache your passphrase for the session — security without inconvenience.

---

## 📚 Related Concepts

- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [ssh_config Reference](https://linux.die.net/man/5/ssh_config)
- [sshd_config Reference](https://linux.die.net/man/5/sshd_config)

</details>
