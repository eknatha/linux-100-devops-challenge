# Challenge 70: Debug Failed Services in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
**eknatha**

## 📌 Overview

On modern Linux systems, **systemd** manages all system services. When a service fails to start or crashes unexpectedly, it transitions to a **failed state**. Quickly identifying and diagnosing failed services is a core sysadmin skill — the tools `systemctl` and `journalctl` provide everything needed to find, inspect, and fix broken services.

---

## 🧩 Task

List all failed services and investigate a specific service's failure logs.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# List all failed services
systemctl --failed

# View detailed logs for a specific service
journalctl -u service_name
```

### How it works

| Command | Description |
|---------|-------------|
| `systemctl --failed` | Lists all units currently in a **failed** state |
| `journalctl -u service_name` | Shows all journal logs for the **specified service unit** |
| `-u` | Filter journalctl by **unit** name |

---

## 🔄 systemd Service States

```
  ┌──────────────┐
  │   inactive   │ ◄── Service has never started or was stopped cleanly
  └──────┬───────┘
         │ start
         ▼
  ┌──────────────┐
  │  activating  │ ◄── Service is in the process of starting
  └──────┬───────┘
         │
    ┌────┴─────┐
    ▼          ▼
┌────────┐  ┌────────┐
│ active │  │ failed │ ◄── Start failed or service crashed
└────────┘  └────────┘
    │
    │ stop / crash
    ▼
┌─────────────┐
│ deactivating│ ◄── Service is shutting down
└─────────────┘
```

---

## 📖 Extended Examples

### Example 1 — List All Failed Services

```bash
systemctl --failed
```

```bash
  UNIT              LOAD   ACTIVE SUB    DESCRIPTION
● nginx.service     loaded failed failed A high performance web server
● myapp.service     loaded failed failed My Application Service
● postgresql.service loaded failed failed PostgreSQL Database

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state.
SUB    = The low-level unit activation state.

3 loaded units listed.
```

---

### Example 2 — View Logs for a Specific Failed Service

```bash
journalctl -u nginx.service
```

```bash
Mar 21 10:00:01 hostname systemd[1]: Starting A high performance web server...
Mar 21 10:00:01 hostname nginx[4821]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Mar 21 10:00:01 hostname nginx[4821]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 21 10:00:01 hostname systemd[1]: nginx.service: Control process exited with error code.
Mar 21 10:00:01 hostname systemd[1]: Failed to start A high performance web server.
```

---

### Example 3 — View Only Recent Logs (Last 50 Lines)

```bash
journalctl -u nginx.service -n 50
```

---

### Example 4 — Follow Logs in Real Time While Restarting

```bash
# In one terminal — follow logs live
journalctl -u nginx.service -f

# In another terminal — attempt to start the service
sudo systemctl start nginx.service
```

> 💡 This is the **most effective debugging technique** — watch logs stream in real time as the service attempts to start, so you see the exact error the moment it occurs.

---

### Example 5 — View Logs from the Current Boot Only

```bash
journalctl -u nginx.service -b
```

> `-b` limits output to the current boot session — avoids confusion with logs from previous failures on earlier boots.

---

### Example 6 — Check Service Status in Detail

```bash
systemctl status nginx.service
```

```bash
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2025-03-21 10:00:01 UTC; 5min ago
    Process: 4821 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
    Process: 4822 ExecStart=/usr/sbin/nginx (code=exited, status=1/FAILURE)
   Main PID: 4822 (code=exited, status=1/FAILURE)

Mar 21 10:00:01 hostname nginx[4821]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Mar 21 10:00:01 hostname systemd[1]: Failed to start A high performance web server.
```

| Field | Description |
|-------|-------------|
| `Loaded` | Whether the unit file was found and loaded |
| `Active` | Current state and how long it has been in that state |
| `Process` | Commands run and their exit codes |
| `Main PID` | PID of the main service process |
| Last log lines | Most recent journal entries for quick context |

---

### Example 7 — Check and Fix Port Conflicts

A common cause of service failure — another process already using the required port:

```bash
# Find what is using port 80
sudo ss -tlnp | grep ':80'
sudo lsof -i :80

# Or with netstat
sudo netstat -tlnp | grep ':80'
```

```bash
tcp  0  0 0.0.0.0:80  0.0.0.0:*  LISTEN  4500/apache2
```

```bash
# Stop the conflicting service
sudo systemctl stop apache2

# Now retry nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

---

### Example 8 — Restart, Reload, and Reset a Failed Service

```bash
# Restart the service (stop + start)
sudo systemctl restart nginx.service

# Reload config without full restart (if supported)
sudo systemctl reload nginx.service

# Reset a failed state so systemctl stop showing it as failed
sudo systemctl reset-failed nginx.service

# Enable service to start automatically at boot
sudo systemctl enable nginx.service

# Disable autostart
sudo systemctl disable nginx.service
```

---

### Example 9 — Validate a Service Configuration File

```bash
# Check nginx config for syntax errors
sudo nginx -t

# Check Apache config
sudo apachectl configtest

# Validate a systemd unit file
systemd-analyze verify /etc/systemd/system/myapp.service
```

```bash
# nginx -t output when OK:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# nginx -t output with error:
nginx: [emerg] unknown directive "servre_name" in /etc/nginx/sites-enabled/mysite:3
nginx: configuration file /etc/nginx/nginx.conf test failed
```

---

### Example 10 — Inspect the Systemd Unit File

```bash
# View the unit file
systemctl cat nginx.service

# Edit the unit file (opens in default editor)
sudo systemctl edit --full nginx.service

# View unit file path and overrides
systemctl show nginx.service -p FragmentPath
```

```bash
# /lib/systemd/system/nginx.service
[Unit]
Description=A high performance web server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
```

---

### Example 11 — Check Systemd Dependencies

```bash
# What does nginx depend on?
systemctl list-dependencies nginx.service

# What services depend on nginx?
systemctl list-dependencies --reverse nginx.service

# Check if all dependencies are satisfied
systemd-analyze critical-chain nginx.service
```

```bash
# critical-chain output:
nginx.service +2ms
└─network.target +1ms
  └─network-pre.target
    └─firewalld.service +800ms
```

> 💡 `critical-chain` shows what delayed the service start and helps identify slow dependencies.

---

### Example 12 — Debug a Custom Service Unit File

A misconfigured unit file is a common cause of failures. Here is a correct template:

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application Service
After=network.target
Wants=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

```bash
# After editing, reload systemd to pick up the changes
sudo systemctl daemon-reload

# Enable and start the service
sudo systemctl enable --now myapp.service

# Check status
systemctl status myapp.service

# Watch logs
journalctl -u myapp.service -f
```

---

### Example 13 — Full Debug Script for Failed Services

```bash
#!/bin/bash

echo "============================================"
echo " Failed Service Report — $(date)"
echo "============================================"

# List all failed units
FAILED=$(systemctl --failed --no-legend | awk '{print $1}')

if [ -z "$FAILED" ]; then
  echo "✅ No failed services."
  exit 0
fi

echo -e "\n[Failed Services Found]"
systemctl --failed

# For each failed service, dump recent logs
for SERVICE in $FAILED; do
  echo ""
  echo "──────────────────────────────────────"
  echo " Logs for: $SERVICE"
  echo "──────────────────────────────────────"
  journalctl -u "$SERVICE" -n 30 --no-pager
done
```

---

## 🗂️ `systemctl` Service Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `systemctl --failed` | List all failed units |
| `systemctl status unit` | Show status and recent logs |
| `systemctl start unit` | Start a service |
| `systemctl stop unit` | Stop a service |
| `systemctl restart unit` | Stop and start a service |
| `systemctl reload unit` | Reload config without full restart |
| `systemctl enable unit` | Enable autostart at boot |
| `systemctl disable unit` | Disable autostart |
| `systemctl reset-failed unit` | Clear the failed state |
| `systemctl cat unit` | View the unit file |
| `systemctl edit unit` | Edit unit file (creates override) |
| `systemctl daemon-reload` | Reload systemd after unit file changes |
| `systemd-analyze verify unit` | Validate unit file syntax |
| `systemd-analyze critical-chain unit` | Show startup dependency chain |

---

## 🔍 Common Failure Causes and Fixes

| Error in Logs | Likely Cause | Fix |
|--------------|-------------|-----|
| `Address already in use` | Port conflict with another service | Stop conflicting service; check with `ss -tlnp` |
| `Permission denied` | Wrong file/directory permissions | Fix with `chmod`/`chown` |
| `No such file or directory` | Missing binary, config, or working dir | Verify paths in unit file with `systemctl cat` |
| `Failed to connect to bus` | D-Bus / systemd not running | Restart systemd or check container environment |
| `exit-code` or `status=1` | Application-level error | Check app logs with `journalctl -u service -n 100` |
| `start-limit-hit` | Service restarted too many times | Run `systemctl reset-failed` then restart |
| `Timeout` | Service took too long to start | Increase `TimeoutStartSec` in unit file |
| `Configuration file test failed` | Config syntax error | Run service's own config test (e.g., `nginx -t`) |

---

## 🚀 Full Workflow — Debug a Failed Service

```bash
# Step 1: Identify all failed services
systemctl --failed

# Step 2: Check detailed status of the failing service
systemctl status nginx.service

# Step 3: View full logs for context
journalctl -u nginx.service -n 100 --no-pager

# Step 4: Follow logs while attempting restart
journalctl -u nginx.service -f &
sudo systemctl start nginx.service

# Step 5: Check for port conflicts or permission issues
sudo ss -tlnp | grep ':80'
ls -la /var/log/nginx/ /etc/nginx/

# Step 6: Validate service configuration
sudo nginx -t

# Step 7: Fix the issue (edit config, free port, fix permissions, etc.)
sudo nano /etc/nginx/nginx.conf

# Step 8: Reset failed state and restart
sudo systemctl reset-failed nginx.service
sudo systemctl start nginx.service

# Step 9: Verify it is running
systemctl status nginx.service

# Step 10: Ensure it starts on boot
sudo systemctl enable nginx.service
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Service keeps failing after restart | Check for `start-limit-hit` — run `systemctl reset-failed` before retrying |
| Edited unit file but changes not applied | Always run `sudo systemctl daemon-reload` after editing unit files |
| `journalctl -u` shows no output | Make sure the service name matches exactly — use `systemctl list-units` to find exact names |
| Service starts manually but fails at boot | Check `After=` and `Wants=` dependencies in the unit file |
| `systemctl status` shows old errors | The status shows recent logs only — use `journalctl` for full history |
| Config file edited but service not reloaded | Run `systemctl reload service` (or `restart` if reload is not supported) |

---

## 🔑 Key Takeaways

- `systemctl --failed` is the **fastest way** to get a list of all broken services in one view.
- `journalctl -u service_name` shows the **full log history** for that service — always the next step after finding a failure.
- Combine `journalctl -u service -f` with `systemctl start service` in split terminals for **real-time debugging**.
- `systemctl status` is great for a **quick snapshot** but `journalctl` gives the complete picture.
- Always run `sudo systemctl daemon-reload` after editing any unit file.
- Use `systemctl reset-failed` to clear the failed state before retrying after fixing an issue.
- Most failures fall into a few categories: **port conflict**, **permission error**, **missing file**, or **config syntax error** — check those first.

---

## 📚 Related Concepts

- [Challenge 62 — Analyze Boot Logs]
- [Challenge 63 — Use journalctl Effectively]
- [Challenge 66 — Configure Firewall]
- [systemd Service Manual](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
- [systemd Unit File Reference](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
- [systemd-analyze](https://www.freedesktop.org/software/systemd/man/systemd-analyze.html) — boot performance analysis tool

</details>
