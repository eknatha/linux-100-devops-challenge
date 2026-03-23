# Challenge 79: Failed Service Investigation in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
**eknatha**

## 📌 Overview

When a Linux service fails, **two commands give you the complete picture** — `systemctl status` provides a concise snapshot including the exit code, signal, and last log lines, while `journalctl -u` delivers the full historical log output for deep investigation.

Mastering this workflow is one of the highest-leverage skills in Linux administration. Nearly every service failure — web servers, databases, custom applications — can be diagnosed in minutes once you know where to look and what the output means.

---

## 🧩 Task

Determine why a systemd service has failed and gather enough information to fix it.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Step 1: Check service status and recent logs
systemctl status service_name

# Step 2: View full service logs
journalctl -u service_name
```

### How it works

| Command | Description |
|---------|-------------|
| `systemctl status service_name` | Shows service state, exit code, last log lines, and PID info |
| `journalctl -u service_name` | Shows the **complete journal history** for the service unit |
| `-u` | Filter journalctl output by **unit** name |

---

## 🔄 systemd Service Lifecycle

```
  ┌────────────┐      start        ┌─────────────┐
  │  inactive  │ ───────────────► │  activating  │
  └────────────┘                   └──────┬───────┘
        ▲                                 │
        │ stop                      ┌─────┴──────┐
        │                           │            │
  ┌─────┴──────┐              ┌──────┴──┐  ┌─────┴──┐
  │deactivating│ ◄────────── │  active  │  │ failed │
  └────────────┘    crash/    └─────────┘  └────────┘
                   exit-code               ▲
                                           │
                              bad config / crash / OOM kill
```

---

## 🔢 Key `systemctl status` Fields

```bash
systemctl status nginx.service
```

```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2025-03-21 10:35:01 UTC; 3min ago
    Process: 4821 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
    Process: 4822 ExecStart=/usr/sbin/nginx (code=exited, status=1/FAILURE)
   Main PID: 4822 (code=exited, status=1/FAILURE)

Mar 21 10:35:01 hostname nginx[4821]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Mar 21 10:35:01 hostname systemd[1]: nginx.service: Control process exited with error code.
Mar 21 10:35:01 hostname systemd[1]: Failed to start A high performance web server.
```

| Field | Description |
|-------|-------------|
| `Loaded` | Whether the unit file was found and is valid |
| `Active: failed` | Service is in the **failed** state |
| `Result: exit-code` | How it failed — `exit-code`, `signal`, `timeout`, `oom-kill` |
| `Process` | Commands run and their exit codes |
| `status=1/FAILURE` | Non-zero exit code — application-level error |
| `status=11/SEGV` | Killed by signal 11 (SIGSEGV — segmentation fault) |
| Last log lines | The most recent journal entries for context |

### `Result` Values and What They Mean

| Result | Meaning | Common Cause |
|--------|---------|-------------|
| `exit-code` | Process exited with non-zero status | Config error, missing dependency, startup script error |
| `signal` | Process killed by a signal | SEGV (crash), KILL (OOM or admin), ABRT (assertion) |
| `timeout` | Process took too long to start | Slow startup, blocked on network/resource |
| `oom-kill` | Killed by the Out Of Memory killer | Process used too much RAM |
| `start-limit-hit` | Restarted too many times in short period | Rapid crash loop |
| `resources` | Resource limit exceeded | File descriptor limit, cgroup limit |
| `core-dump` | Process crashed and produced a core dump | Segfault, heap corruption |

---

## 📖 Extended Examples

### Example 1 — Basic Status Check

```bash
systemctl status nginx.service
```

```bash
# Service is running:
● nginx.service - A high performance web server
     Active: active (running) since Thu 2025-03-21 09:00:01; 1h 35min ago
   Main PID: 1234 (nginx)

# Service has failed:
● nginx.service - A high performance web server
     Active: failed (Result: exit-code) since Thu 2025-03-21 10:35:01; 3min ago
```

---

### Example 2 — View Full Service Logs

```bash
# Full log history for the service
journalctl -u nginx.service

# Last 50 lines
journalctl -u nginx.service -n 50

# From current boot only
journalctl -u nginx.service -b

# Follow in real time
journalctl -u nginx.service -f

# Only errors and above
journalctl -u nginx.service -p err
```

---

### Example 3 — Identify the Exact Failure Reason

The `Result` field in `systemctl status` tells you **why** the service failed:

```bash
# Exit code failure — look for application error messages
systemctl status myapp.service
# Active: failed (Result: exit-code)
journalctl -u myapp.service -n 50 | grep -i "error\|fail\|fatal"

# Signal failure — check for crash/OOM
systemctl status myapp.service
# Active: failed (Result: signal)
# Process: ... (code=killed, signal=SEGV)
dmesg | grep -i "segfault\|oom"

# Timeout — service too slow to start
systemctl status myapp.service
# Active: failed (Result: timeout)
# → Increase TimeoutStartSec in unit file
```

---

### Example 4 — Investigate a Start Limit Hit

```bash
systemctl status myapp.service
```

```bash
Active: failed (Result: start-limit-hit)
```

```bash
# See how fast it was crashing
journalctl -u myapp.service --since "10 min ago" | grep "Started\|Failed"
```

```bash
Mar 21 10:30:01 hostname systemd[1]: Started My App.
Mar 21 10:30:02 hostname systemd[1]: myapp.service: Failed with result 'exit-code'.
Mar 21 10:30:07 hostname systemd[1]: Started My App.
Mar 21 10:30:08 hostname systemd[1]: myapp.service: Failed with result 'exit-code'.
# → Crashing within 1 second of starting = crash loop
```

```bash
# Reset the failed state to allow a fresh restart
sudo systemctl reset-failed myapp.service

# Then restart after fixing the issue
sudo systemctl start myapp.service
```

---

### Example 5 — Check Service Configuration for Errors

```bash
# View the unit file
systemctl cat nginx.service

# Validate unit file syntax
systemd-analyze verify /etc/systemd/system/myapp.service

# Check if the binary exists
systemctl show nginx.service -p ExecStart

# Validate nginx config
sudo nginx -t

# Validate Apache config
sudo apachectl configtest

# Validate MySQL config
sudo mysqld --validate-config
```

```bash
# systemd-analyze verify output:
/etc/systemd/system/myapp.service:8: Unknown key name 'ExecStar' in section 'Service'
#                                                  ^^^^^^^^^^ typo found!
```

---

### Example 6 — Find Port Conflicts

One of the most common service failure causes:

```bash
# Find what is using the port nginx needs (80)
sudo ss -tlnp | grep ':80'
sudo lsof -i :80

# Find what is using port 3306 (MySQL)
sudo ss -tlnp | grep ':3306'
```

```bash
# Output showing Apache is already on port 80:
tcp  LISTEN 0 128  0.0.0.0:80  0.0.0.0:*  users:(("apache2",pid=1234,fd=4))
```

```bash
# Fix: stop the conflicting service
sudo systemctl stop apache2
sudo systemctl start nginx
```

---

### Example 7 — Check File and Directory Permissions

```bash
# Check the service is running as the right user
systemctl show nginx.service -p User
systemctl show nginx.service -p WorkingDirectory

# Check permissions on key files
ls -la /etc/nginx/nginx.conf
ls -la /var/log/nginx/

# Check if the binary is executable
ls -la /usr/sbin/nginx
```

```bash
# Common permission problem:
ls -la /var/run/myapp/
# drwxr-x--- 2 root root 40 Mar 21 10:00 /var/run/myapp/
#                                         ← myapp user can't write here!

# Fix:
sudo chown myapp:myapp /var/run/myapp/
sudo chmod 755 /var/run/myapp/
```

---

### Example 8 — Check Environment Variables and Dependencies

```bash
# View environment variables the service uses
systemctl show myapp.service -p Environment

# Check if a required service is running
systemctl is-active postgresql.service
systemctl is-active redis.service

# See service dependencies
systemctl list-dependencies myapp.service

# Check if a required port is open
ss -tlnp | grep ':5432'   # PostgreSQL
ss -tlnp | grep ':6379'   # Redis
```

---

### Example 9 — Compare Logs Across Multiple Boots

```bash
# List all boots
journalctl --list-boots

# Did it fail in the previous boot too?
journalctl -u myapp.service -b -1 | grep "Failed\|Error"

# Compare current vs previous boot failure messages
echo "=== Previous Boot ===" && journalctl -u myapp.service -b -1 | grep "Failed" | tail -5
echo "=== Current Boot ===" && journalctl -u myapp.service -b 0 | grep "Failed" | tail -5
```

---

### Example 10 — Investigate a Timeout Failure

```bash
systemctl status myapp.service
# Active: failed (Result: timeout)
```

```bash
# Check the current timeout settings
systemctl show myapp.service -p TimeoutStartUSec
systemctl show myapp.service -p TimeoutStopUSec
```

```bash
# Increase timeout in the unit file
sudo systemctl edit myapp.service
```

```ini
# Override file: /etc/systemd/system/myapp.service.d/override.conf
[Service]
TimeoutStartSec=120    # Give it 120 seconds to start
TimeoutStopSec=30      # Give it 30 seconds to stop
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
```

---

### Example 11 — Export Service Logs for Sharing or Archiving

```bash
# Export to a text file
journalctl -u myapp.service --no-pager > /tmp/myapp_logs_$(date +%Y%m%d).txt

# Export as JSON
journalctl -u myapp.service -o json --no-pager > /tmp/myapp_logs.json

# Export only the failures
journalctl -u myapp.service -p err --no-pager > /tmp/myapp_errors.txt

# Export last boot logs only
journalctl -u myapp.service -b --no-pager > /tmp/myapp_boot_logs.txt
```

---

### Example 12 — Automated Service Health Check Script

```bash
#!/bin/bash

SERVICES=("nginx" "mysql" "myapp")
REPORT="/tmp/service_health_$(date +%Y%m%d_%H%M%S).txt"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "========================================" | tee "$REPORT"
echo " Service Health Report — $DATE"           | tee -a "$REPORT"
echo "========================================" | tee -a "$REPORT"

ALL_OK=true

for SVC in "${SERVICES[@]}"; do
  STATUS=$(systemctl is-active "$SVC.service" 2>/dev/null)
  RESULT=$(systemctl show "$SVC.service" -p Result --value 2>/dev/null)

  if [ "$STATUS" == "active" ]; then
    echo "✅ $SVC: running"                     | tee -a "$REPORT"
  else
    ALL_OK=false
    echo "❌ $SVC: $STATUS (Result: $RESULT)"  | tee -a "$REPORT"
    echo "   Last 10 log lines:"               | tee -a "$REPORT"
    journalctl -u "$SVC.service" -n 10 --no-pager | sed 's/^/   /' | tee -a "$REPORT"
    echo ""                                     | tee -a "$REPORT"
  fi
done

if $ALL_OK; then
  echo -e "\n✅ All services healthy."          | tee -a "$REPORT"
else
  echo -e "\n⚠️  Some services need attention." | tee -a "$REPORT"
fi

echo "Report saved: $REPORT"
```

---

## 🗂️ Failure Cause → Investigation Path

| Failure Type | `Result` in Status | Where to Look | Likely Fix |
|-------------|-------------------|--------------|------------|
| Config error | `exit-code` | `journalctl -u svc -n 50` + `service -t` | Fix config file syntax |
| Port conflict | `exit-code` | `ss -tlnp | grep :PORT` | Stop conflicting service |
| Permission error | `exit-code` | `strace -e trace=file -p PID` | Fix `chown`/`chmod` |
| Crash / SEGV | `signal` | `coredumpctl info` + `dmesg | grep segfault` | Fix bug or update version |
| OOM kill | `oom-kill` | `dmesg | grep -i oom` + `free -h` | Add memory or tune limits |
| Slow startup | `timeout` | `journalctl -u svc -f` while starting | Increase `TimeoutStartSec` |
| Crash loop | `start-limit-hit` | `journalctl -u svc --since "5 min ago"` | Fix root cause, then `reset-failed` |
| Missing binary | `exit-code` | `systemctl cat svc | grep ExecStart` | Install or fix binary path |

---

## 🛠️ Investigation Commands — Quick Reference

| Command | Purpose |
|---------|---------|
| `systemctl status svc` | First look: state, exit code, last log lines |
| `journalctl -u svc -n 100` | Full logs, last 100 lines |
| `journalctl -u svc -b -p err` | Errors only from current boot |
| `journalctl -u svc -f` | Follow live |
| `systemctl cat svc` | View the unit file |
| `systemctl show svc` | All unit properties |
| `systemd-analyze verify unit` | Validate unit file syntax |
| `systemctl reset-failed svc` | Clear failed state before retrying |
| `systemctl daemon-reload` | Reload after editing unit files |
| `systemctl edit svc` | Add override configuration |
| `ss -tlnp | grep :PORT` | Find port conflicts |
| `coredumpctl list` | List core dumps |

---

## 🚀 Full Workflow — Investigate and Fix a Failed Service

```bash
# Step 1: See all failed services at once
systemctl --failed

# Step 2: Get the status snapshot
systemctl status myapp.service

# Step 3: Read the full logs
journalctl -u myapp.service -n 100 --no-pager

# Step 4: Watch live as you attempt to start it
journalctl -u myapp.service -f &
sudo systemctl start myapp.service

# Step 5: Identify the root cause from the output
# Port conflict? → ss -tlnp
# Permission? → ls -la on the relevant paths
# Config syntax? → validate with the service's own check tool
# Missing file? → trace with strace

# Step 6: Fix the issue

# Step 7: If in start-limit-hit state, reset it
sudo systemctl reset-failed myapp.service

# Step 8: Reload daemon if unit file was edited
sudo systemctl daemon-reload

# Step 9: Start and verify
sudo systemctl start myapp.service
systemctl status myapp.service

# Step 10: Ensure it starts at boot
sudo systemctl enable myapp.service
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `systemctl start` fails with no clear message | Use `journalctl -u svc -n 50` for the full error |
| Service keeps failing after fix | Check for `start-limit-hit` — run `systemctl reset-failed` |
| Edited unit file but changes not applied | Always run `systemctl daemon-reload` after editing |
| `journalctl -u` shows nothing | Confirm the exact service name with `systemctl list-units --all | grep myapp` |
| Service starts manually but not at boot | Check `After=` dependencies — required services may not be ready yet |
| `Result: exit-code` with no useful log | Add `StandardOutput=journal` and `StandardError=journal` to unit file |

---

## 🔑 Key Takeaways

- **`systemctl status`** is always step 1 — it shows the exit code, signal, and last few log lines at a glance.
- **`journalctl -u service`** is step 2 — it gives the complete story with every log line.
- The **`Result` field** in `systemctl status` is the most important diagnostic signal — `exit-code`, `signal`, `timeout`, and `oom-kill` each point to a different investigation path.
- **`start-limit-hit`** means the service crashed too many times too fast — fix the root cause first, then `systemctl reset-failed` before retrying.
- Always run **`systemctl daemon-reload`** after editing any unit file.
- **`systemd-analyze verify`** catches unit file syntax errors before they cause failures.
- Combine `journalctl -u svc -f` with `systemctl start svc` in separate terminals to see the exact error in real time.

---

## 📚 Related Concepts

- [Challenge 70 — Debug Failed Services]
- [Challenge 78 — Debug Application Crash]
- [Challenge 63 — Use journalctl Effectively]
- [systemd.service Manual](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
- [systemd-analyze](https://www.freedesktop.org/software/systemd/man/systemd-analyze.html)
- [journalctl Manual](https://www.freedesktop.org/software/systemd/man/journalctl.html)


</details>
