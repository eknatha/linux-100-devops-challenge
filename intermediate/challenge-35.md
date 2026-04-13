# Challenge 35: Manage System Services in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**System services** (also called daemons) are background processes that run continuously — web servers, database engines, SSH daemons, and more. On modern Linux systems, services are managed by **systemd** through the `systemctl` command.

Understanding how to start, stop, enable, disable, and check services is an essential sysadmin skill. The key distinction:
- **Starting/stopping** — controls whether the service is running **right now**
- **Enabling/disabling** — controls whether it starts **automatically at boot**

---

## 🧩 Task

Start a service and enable it to start automatically at boot.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Start a service now
sudo systemctl start nginx

# Enable it to start automatically at boot
sudo systemctl enable nginx

# Start AND enable in one command
sudo systemctl enable --now nginx
```

### How it works

| Command | Description |
|---------|-------------|
| `systemctl start service` | **Start** the service immediately |
| `systemctl stop service` | **Stop** the service immediately |
| `systemctl enable service` | **Enable** — start automatically at boot |
| `systemctl disable service` | **Disable** — do not start at boot |
| `systemctl enable --now service` | Enable **and** start in one command |

---

## 🔢 Start vs Enable — The Key Difference

```
systemctl start nginx
├── Service starts NOW ✅
└── On next reboot — service does NOT start ❌ (if not enabled)

systemctl enable nginx
├── Service does NOT start now ❌
└── On next reboot — service WILL start ✅

systemctl enable --now nginx
├── Service starts NOW ✅
└── On next reboot — service WILL start ✅  ← Best option
```

---

## 📖 Extended Examples

### Example 1 — Start a Service

```bash
# Start nginx
sudo systemctl start nginx

# Verify it is running
systemctl status nginx
```

```bash
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
     Active: active (running) since Thu 2025-03-21 10:00:01; 5s ago
   Main PID: 5201 (nginx)
```

---

### Example 2 — Enable a Service at Boot

```bash
# Enable nginx to start at boot
sudo systemctl enable nginx
```

```bash
# Output:
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service
  → /lib/systemd/system/nginx.service
```

> 💡 Enabling creates a **symlink** in the systemd wants directory — that is all it does. The service does not start until you also run `start`.

---

### Example 3 — Enable and Start in One Command

```bash
# The most convenient option — enable AND start
sudo systemctl enable --now nginx
```

---

### Example 4 — Stop a Service

```bash
sudo systemctl stop nginx

# Verify it stopped
systemctl status nginx
```

```bash
● nginx.service - A high performance web server
     Active: inactive (dead) since Thu 2025-03-21 10:05:01; 3s ago
```

---

### Example 5 — Restart and Reload a Service

```bash
# Restart — fully stop then start (brief downtime)
sudo systemctl restart nginx

# Reload — reload config without stopping (no downtime if supported)
sudo systemctl reload nginx

# Reload if supported, restart if not
sudo systemctl reload-or-restart nginx
```

| Command | Effect | Downtime |
|---------|--------|---------|
| `restart` | Stop + start | Brief |
| `reload` | Reload config only | None |
| `reload-or-restart` | Try reload, fall back to restart | Minimal |

---

### Example 6 — Check Service Status

```bash
sudo systemctl status nginx
```

```bash
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-03-21 10:00:01 UTC; 35min ago
    Process: 5200 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
   Main PID: 5201 (nginx)
      Tasks: 3 (limit: 4915)
     Memory: 8.2M
      CGroup: /system.slice/nginx.service
              ├─5201 nginx: master process /usr/sbin/nginx
              ├─5202 nginx: worker process
              └─5203 nginx: worker process

Mar 21 10:00:01 hostname nginx[5201]: nginx: the configuration file ...is OK
Mar 21 10:00:01 hostname systemd[1]: Started A high performance web server.
```

---

### Example 7 — Disable a Service

```bash
# Disable nginx from starting at boot
sudo systemctl disable nginx

# Disable and stop in one command
sudo systemctl disable --now nginx
```

---

### Example 8 — Check if a Service is Running or Enabled

```bash
# Check if service is currently active (running)
systemctl is-active nginx
# active / inactive / failed

# Check if service is enabled at boot
systemctl is-enabled nginx
# enabled / disabled / static

# Check if service has failed
systemctl is-failed nginx
# active / failed
```

```bash
# Use in scripts
if systemctl is-active nginx &>/dev/null; then
  echo "nginx is running"
else
  echo "nginx is NOT running"
fi
```

---

### Example 9 — List All Services

```bash
# List all loaded services and their status
systemctl list-units --type=service

# List only running services
systemctl list-units --type=service --state=running

# List only failed services
systemctl list-units --type=service --state=failed

# List all services (including inactive)
systemctl list-units --type=service --all
```

```bash
# Output:
UNIT                   LOAD   ACTIVE SUB     DESCRIPTION
cron.service           loaded active running Regular background program processing daemon
nginx.service          loaded active running A high performance web server
ssh.service            loaded active running OpenBSD Secure Shell server
mysql.service          loaded active running MySQL Community Server
```

---

### Example 10 — Mask and Unmask a Service

**Masking** completely prevents a service from being started — even manually:

```bash
# Mask a service (prevent any start)
sudo systemctl mask nginx

# Try to start — it will fail
sudo systemctl start nginx
# Failed to start nginx.service: Unit nginx.service is masked.

# Unmask to allow again
sudo systemctl unmask nginx
```

> 💡 Use `mask` for services you never want running — more restrictive than `disable`.

---

### Example 11 — Check Service Logs

```bash
# View logs for a service
journalctl -u nginx

# Last 50 lines
journalctl -u nginx -n 50

# Follow live
journalctl -u nginx -f

# Errors only
journalctl -u nginx -p err
```

---

### Example 12 — Reload systemd After Creating a New Service

```bash
# After adding or modifying a unit file — reload systemd
sudo systemctl daemon-reload

# Then start and enable the new service
sudo systemctl enable --now myapp.service
```

---

## 🗂️ `systemctl` Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `systemctl start service` | Start service now |
| `systemctl stop service` | Stop service now |
| `systemctl restart service` | Stop + start service |
| `systemctl reload service` | Reload config (no downtime) |
| `systemctl reload-or-restart service` | Reload if supported, else restart |
| `systemctl enable service` | Enable at boot |
| `systemctl disable service` | Disable at boot |
| `systemctl enable --now service` | Enable + start now |
| `systemctl disable --now service` | Disable + stop now |
| `systemctl status service` | Show status and recent logs |
| `systemctl is-active service` | Check if running |
| `systemctl is-enabled service` | Check if enabled at boot |
| `systemctl is-failed service` | Check if in failed state |
| `systemctl mask service` | Prevent any start |
| `systemctl unmask service` | Remove mask |
| `systemctl daemon-reload` | Reload after unit file changes |
| `systemctl list-units --type=service` | List all services |
| `systemctl --failed` | Show all failed services |

---

## 🔄 Service State — What Each Means

| State | Description |
|-------|-------------|
| `active (running)` | Service is running ✅ |
| `active (exited)` | One-shot service completed successfully |
| `inactive (dead)` | Service is not running |
| `failed` | Service exited with an error |
| `enabled` | Will start at boot |
| `disabled` | Will NOT start at boot |
| `masked` | Cannot be started at all |
| `static` | No [Install] section — cannot be enabled |

---

## 🚀 Full Workflow — Deploy and Verify a Service

```bash
# Step 1: Install the service (e.g., nginx)
sudo apt install nginx    # Debian/Ubuntu
sudo dnf install nginx    # RHEL

# Step 2: Start it and enable at boot
sudo systemctl enable --now nginx

# Step 3: Verify it is running
systemctl status nginx

# Step 4: Check it is enabled
systemctl is-enabled nginx

# Step 5: Check the firewall allows traffic
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

# Step 6: Test the service
curl http://localhost

# Step 7: Check logs if something is wrong
journalctl -u nginx -n 50

# Step 8: If you made config changes, reload
sudo nginx -t              # Validate config
sudo systemctl reload nginx  # Apply without downtime
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Service starts but stops after reboot | Use `enable` — `start` alone does not survive reboots |
| `enable` but service not running now | Use `enable --now` or run `start` separately |
| Config changed but service using old config | Run `systemctl reload service` or `restart` |
| Edited unit file but changes not picked up | Run `systemctl daemon-reload` first |
| Service keeps restarting in a loop | Check `systemctl status` and `journalctl -u service -n 50` for errors |
| `is-active` returns `failed` | Run `systemctl reset-failed service` then fix the root cause |

---

## 🔑 Key Takeaways

- `systemctl enable --now service` is the one command to **both enable at boot AND start now** — use it as the default.
- `start`/`stop` controls **current state only** — does not affect boot behavior.
- `enable`/`disable` controls **boot behavior only** — does not affect current state.
- `systemctl status service` is the first diagnostic command — it shows state, PID, and recent log lines.
- Always run `systemctl daemon-reload` after **editing unit files**.
- Use `systemctl reload` instead of `restart` when possible — it avoids brief service downtime.
- `systemctl mask service` completely prevents a service from starting — more restrictive than `disable`.

---

## 📚 Related Concepts

- [systemd Service Manual](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
- [systemctl Manual](https://www.freedesktop.org/software/systemd/man/systemctl.html)

</details>
