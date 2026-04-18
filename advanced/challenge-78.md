# Challenge 78: Debug Application Crashes in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Application crashes are among the most disruptive events in a Linux environment — they can bring down services, corrupt data, and leave no obvious trace of what went wrong. Effective crash investigation requires knowing **where to look**: the systemd journal, the kernel ring buffer, application logs, core dumps, and signal handling all tell different parts of the story.

The combination of `journalctl -xe` and `dmesg | tail` covers the two most important sources:
- **`journalctl -xe`** — the systemd journal, which captures application-level crash messages, service failures, and OOM kills
- **`dmesg`** — the kernel ring buffer, which captures hardware errors, kernel panics, and OOM killer events

---

## 🧩 Task

Investigate why an application crashed — finding error messages, signal information, and kernel-level events.

---



<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# View recent journal entries with explanations
journalctl -xe

# View recent kernel ring buffer messages
dmesg | tail
```

### How it works

| Command | Description |
|---------|-------------|
| `journalctl` | Query the systemd journal |
| `-x` | Add **explanatory text** (catalog messages) for well-known errors |
| `-e` | Jump to the **end** of the journal (most recent entries) |
| `dmesg` | Print the **kernel ring buffer** messages |
| `\| tail` | Show only the last 10 lines |

---

## 🔢 Understanding the Output

### `journalctl -xe` Output

```bash
Mar 21 10:35:01 hostname systemd[1]: myapp.service: Main process exited, code=killed, status=11/SEGV
Mar 21 10:35:01 hostname systemd[1]: myapp.service: Failed with result 'signal'.
Mar 21 10:35:01 hostname systemd[1]: Failed to start My Application Service.
-- Subject: Unit myapp.service has failed
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- Unit myapp.service has entered the 'failed' state with result 'signal'.
```

### `dmesg | tail` Output

```bash
[12345.678901] Out of memory: Kill process 4821 (myapp) score 890 or sacrifice child
[12345.678910] Killed process 4821 (myapp) total-vm:2097152kB, anon-rss:1048576kB, file-rss:0kB
[12346.012345] myapp[4821]: segfault at 0x00007f8b2c001234 ip 0x0000000000401234 sp 0x00007ffd12345678 error 6
```

---

## 🔢 Crash Signal Reference

| Signal | Number | Meaning | Common Cause |
|--------|--------|---------|-------------|
| `SIGSEGV` | 11 | Segmentation fault — illegal memory access | Bug, null pointer, buffer overflow |
| `SIGABRT` | 6 | Abort — process called `abort()` | Failed assertion, heap corruption |
| `SIGKILL` | 9 | Killed — sent by OOM killer or admin | Out of memory, `kill -9` |
| `SIGTERM` | 15 | Terminated — graceful stop requested | `systemctl stop`, process manager |
| `SIGBUS` | 7 | Bus error — misaligned memory access | Memory mapping issues |
| `SIGFPE` | 8 | Floating point exception | Division by zero, arithmetic error |
| `SIGILL` | 4 | Illegal instruction | Corrupt binary, wrong CPU arch |
| `SIGPIPE` | 13 | Broken pipe — writing to closed socket | Remote end closed connection |

---

## 📖 Extended Examples

### Example 1 — Basic Crash Investigation

```bash
# Step 1: Check journal for recent failures
journalctl -xe

# Step 2: Check kernel messages
dmesg | tail -20
```

---

### Example 2 — Filter Journal to a Specific Service

```bash
# View all logs for a crashed service
journalctl -u myapp.service

# View with explanatory notes
journalctl -u myapp.service -xe

# View only errors
journalctl -u myapp.service -p err

# View from current boot only
journalctl -u myapp.service -b
```

---

### Example 3 — Check systemctl Status for Crash Details

```bash
systemctl status myapp.service
```

```bash
● myapp.service - My Application Service
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled)
     Active: failed (Result: signal) since Thu 2025-03-21 10:35:01; 2min ago
    Process: 4821 ExecStart=/usr/bin/myapp (code=killed, signal=SEGV)
   Main PID: 4821 (code=killed, signal=SEGV)

Mar 21 10:35:00 hostname myapp[4821]: Starting application...
Mar 21 10:35:00 hostname myapp[4821]: Loading config from /etc/myapp/config.yaml
Mar 21 10:35:01 hostname myapp[4821]: Segmentation fault (core dumped)
```

| Field | Description |
|-------|-------------|
| `Result: signal` | Process was killed by a signal |
| `signal=SEGV` | The signal was SIGSEGV (segmentation fault) |
| `code=killed` | The process was terminated by a signal |
| `core dumped` | A core dump file was created for debugging |

---

### Example 4 — Check `dmesg` for OOM Kills and Segfaults

```bash
# Recent kernel messages
dmesg | tail -30

# Filter for OOM events
dmesg | grep -i "oom\|out of memory\|killed process"

# Filter for segfaults
dmesg | grep -i "segfault"

# Filter for general errors
dmesg | grep -i "error\|fail\|warn"

# Show with timestamps
dmesg -T | tail -20
```

```bash
# OOM kill example:
[Mar21 10:35:00] Out of memory: Kill process 4821 (myapp) score 890 or sacrifice child
[  +0.000012] Killed process 4821 (myapp) total-vm:2097152kB, anon-rss:1048576kB

# Segfault example:
[Mar21 10:35:01] myapp[4821]: segfault at 7f8b2c00 ip 00401234 sp 7ffd1234 error 6 in myapp[400000+12000]
```

#### Decoding a Segfault Message

```
myapp[4821]: segfault at 7f8b2c00 ip 00401234 sp 7ffd1234 error 6
              ^^^^^^^^           ^^^^^^^^         ^^^^^^^^^^^^^^^^
              Address            Instruction      Error code:
              being accessed     Pointer           4 = user space read
              (NULL = 0x0        (where crash       6 = user space write
              means null ptr     happened)
              deref)
```

---

### Example 5 — Check for OOM Killer Activity

```bash
# Check systemd journal for OOM kills
journalctl -k | grep -i "oom\|killed process\|out of memory"

# Check dmesg for OOM (with timestamps)
dmesg -T | grep -i "oom\|killed"

# Check if memory was the cause
free -h
```

```bash
[Thu Mar 21 10:35:00 2025] Out of memory: Kill process 4821 (myapp) score 890 or sacrifice child
[Thu Mar 21 10:35:00 2025] Killed process 4821 (myapp) total-vm:2097152kB, anon-rss:1048576kB, file-rss:0kB
```

> 💡 The **OOM score** (0–1000) determines which process gets killed first — a score of 890 means `myapp` was using a very large proportion of available memory.

---

### Example 6 — Enable and Find Core Dumps

A **core dump** is a snapshot of the process's memory at the time of crash — it is the most powerful tool for post-mortem debugging.

```bash
# Check if core dumps are enabled
ulimit -c

# Enable core dumps for the current session
ulimit -c unlimited

# Check where core dumps are saved (systemd systems)
cat /proc/sys/kernel/core_pattern
```

```bash
# Typical core_pattern values:
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h
# OR
/var/lib/systemd/coredump/core.%e.%u.%g.%s.%t.%c
```

```bash
# List core dumps managed by systemd
coredumpctl list

# Show details of the most recent crash
coredumpctl info

# Open the most recent core dump in gdb
coredumpctl gdb myapp
```

```bash
# coredumpctl list output:
TIME                            PID  UID  GID SIG COREFILE EXE
Thu 2025-03-21 10:35:01 UTC   4821 1001 1001  11 present  /usr/bin/myapp
```

---

### Example 7 — Analyze a Core Dump with `gdb`

```bash
# Load the executable and core dump in gdb
gdb /usr/bin/myapp /var/lib/systemd/coredump/core.myapp.1001.1001.11.1711017301.0

# Inside gdb:
(gdb) bt          # Backtrace — show function call stack at crash
(gdb) info regs   # Show CPU registers
(gdb) list        # Show source code at crash location (if debug symbols available)
(gdb) quit        # Exit gdb
```

```bash
# gdb backtrace output:
#0  0x00007f8b2c001234 in process_request (req=0x0) at server.c:142
#1  0x0000000000401234 in handle_connection (conn=0x7f8b00001234) at server.c:89
#2  0x0000000000400abc in main () at server.c:42
```

> The backtrace immediately tells you the exact **function**, **file**, and **line number** where the crash occurred — with debug symbols compiled in.

---

### Example 8 — Check Application-Specific Logs

```bash
# Common application log locations
tail -50 /var/log/myapp/error.log
tail -50 /var/log/nginx/error.log
tail -50 /var/log/mysql/error.log
tail -50 /var/log/postgresql/postgresql-*.log
tail -50 /var/log/apache2/error.log

# Or follow live
tail -f /var/log/myapp/error.log
```

---

### Example 9 — Check for Crash Patterns Over Time

```bash
# Count how many times a service has crashed today
journalctl -u myapp.service --since today | grep -c "Failed\|crashed\|SEGV\|killed"

# See crash timestamps
journalctl -u myapp.service | grep "code=killed\|Failed with result"

# Compare crash frequency across boots
journalctl --list-boots | head -10
journalctl -u myapp.service -b -1 | grep "Failed"   # Previous boot
journalctl -u myapp.service -b 0  | grep "Failed"   # Current boot
```

---

### Example 10 — Inspect Resource Limits (Common Crash Cause)

```bash
# Check limits for a running process
cat /proc/<PID>/limits

# Check system-wide limits
ulimit -a

# Check open file descriptor limit (common cause of crashes at scale)
cat /proc/<PID>/limits | grep "open files"
lsof -p <PID> | wc -l      # How many FDs the process currently has open
```

```bash
# /proc/PID/limits output:
Limit                     Soft Limit   Hard Limit  Units
Max cpu time              unlimited    unlimited   seconds
Max file size             unlimited    unlimited   bytes
Max data size             unlimited    unlimited   bytes
Max open files            1024         65536       files   ← common culprit
Max stack size            8388608      unlimited   bytes
```

> ⚠️ A process hitting its **open file limit** (`Max open files`) often crashes with cryptic errors — always check this when FD-intensive services crash.

---

### Example 11 — Enable Persistent Core Dumps via systemd-coredump

```bash
# Configure systemd-coredump
sudo nano /etc/systemd/coredump.conf
```

```ini
[Coredump]
Storage=external          # Save to disk (not memory)
Compress=yes              # Compress core dumps
ProcessSizeMax=2G         # Max core dump size
ExternalSizeMax=2G        # Max stored size
KeepFree=1G               # Keep 1GB free on disk
MaxUse=10G                # Max total space for core dumps
```

```bash
# Apply changes
sudo systemctl daemon-reload

# Verify it is working after next crash
coredumpctl list
```

---

### Example 12 — Full Crash Investigation Script

```bash
#!/bin/bash

SERVICE=${1:-"myapp"}
OUTPUT="/tmp/crash_report_$(date +%Y%m%d_%H%M%S).txt"

echo "============================================" | tee "$OUTPUT"
echo " Crash Report: $SERVICE — $(date)"           | tee -a "$OUTPUT"
echo "============================================" | tee -a "$OUTPUT"

echo -e "\n[1] Service Status]"                    | tee -a "$OUTPUT"
systemctl status "$SERVICE.service" --no-pager     | tee -a "$OUTPUT"

echo -e "\n[2] Recent Journal Logs (last 50)]"     | tee -a "$OUTPUT"
journalctl -u "$SERVICE.service" -n 50 --no-pager  | tee -a "$OUTPUT"

echo -e "\n[3] OOM Killer Activity]"               | tee -a "$OUTPUT"
journalctl -k | grep -i "oom\|killed process" | tail -10 | tee -a "$OUTPUT"

echo -e "\n[4] Kernel Segfault Messages]"          | tee -a "$OUTPUT"
dmesg -T | grep -i "segfault\|error" | tail -10    | tee -a "$OUTPUT"

echo -e "\n[5] Recent Core Dumps]"                 | tee -a "$OUTPUT"
coredumpctl list 2>/dev/null | tail -10            | tee -a "$OUTPUT"

echo -e "\n[6] Memory Status]"                     | tee -a "$OUTPUT"
free -h                                            | tee -a "$OUTPUT"

echo -e "\n[7] Disk Space]"                        | tee -a "$OUTPUT"
df -h | awk '$5 > "80%"'                           | tee -a "$OUTPUT"

echo -e "\nReport saved: $OUTPUT"
```

---

## 🗂️ Crash Investigation — Where to Look First

| Symptom | First Place to Check |
|---------|---------------------|
| Service shows `failed` | `systemctl status service` |
| Crash with no output | `journalctl -xe` → look for signal number |
| OOM suspected | `dmesg -T | grep -i oom` + `free -h` |
| Segmentation fault | `dmesg | grep segfault` + `coredumpctl info` |
| Crash after file operations | `strace -e trace=file -p PID` |
| Crash after many connections | Check `ulimit -n` and open FD count |
| Crash with no core dump | `ulimit -c unlimited` + check `core_pattern` |
| Crash only under load | Profile with `perf` or `valgrind` |

---

## 🛠️ Crash Investigation Tools — Summary

| Tool | What It Reveals |
|------|----------------|
| `journalctl -xe` | Service logs, crash signals, OOM kills, systemd messages |
| `dmesg -T` | Kernel ring buffer: segfaults, hardware errors, OOM events |
| `systemctl status` | Service state, exit code, signal, last log lines |
| `coredumpctl` | List and analyze core dumps |
| `gdb` | Stack backtrace, register state, memory at crash time |
| `strace` | Syscall trace leading up to crash |
| `valgrind` | Memory error detection (buffer overflows, use-after-free) |
| `perf` | CPU profiling and performance event tracing |

---

## 🚀 Full Workflow — Investigate an Application Crash

```bash
# Step 1: Identify the crash
systemctl status myapp.service

# Step 2: Get full logs with explanations
journalctl -u myapp.service -xe

# Step 3: Check for OOM or kernel-level cause
dmesg -T | grep -i "oom\|segfault\|killed" | tail -20

# Step 4: Check if a core dump was saved
coredumpctl list

# Step 5: Examine the core dump
coredumpctl info myapp
coredumpctl gdb myapp    # Interactive gdb session

# Step 6: Get the backtrace inside gdb
# (gdb) bt
# (gdb) info locals
# (gdb) quit

# Step 7: Check resource limits if crash is under load
cat /proc/<PID>/limits
lsof -p <PID> | wc -l

# Step 8: Correlate with application logs
tail -100 /var/log/myapp/error.log

# Step 9: Run with strace to capture syscalls before crash
sudo strace -f -o /tmp/crash_trace.txt myapp

# Step 10: After fixing, monitor for recurrence
journalctl -u myapp.service -f
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| No core dump found | Check `ulimit -c` and `core_pattern`; enable with `ulimit -c unlimited` |
| `coredumpctl list` shows nothing | Ensure `systemd-coredump` is installed and `core_pattern` points to it |
| Crash happens only in production | Use `strace -c` for low-overhead profiling; check resource limits |
| `journalctl -xe` shows no crash info | Service may have crashed before systemd captured it — check `dmesg` |
| OOM kill but memory looks free | Check swap (`free -h`) and memory fragmentation (`/proc/buddyinfo`) |
| Backtrace in `gdb` shows `??` | Binary stripped of debug symbols — recompile with `-g` or install debug packages |
| Crash only under specific conditions | Use `valgrind --tool=memcheck` to detect memory errors in test environment |

---

## 🔑 Key Takeaways

- `journalctl -xe` is the **first stop** — it shows recent logs with explanations and the signal that killed the process.
- `dmesg | tail` reveals **kernel-level events** like OOM kills and segfaults that application logs never capture.
- The **signal number** in the crash message tells you why: `SEGV` = memory bug, `KILL` = OOM or admin, `ABRT` = assertion failure.
- **Core dumps** combined with `gdb` give you the exact **function and line number** where the crash occurred.
- `coredumpctl` on systemd systems manages core dumps automatically — always check it first before searching the filesystem.
- Always check **resource limits** (`ulimit -a`, `/proc/PID/limits`) when crashes occur under load.
- Use `dmesg -T` (with human-readable timestamps) to correlate kernel events with application crash times.

---

## 📚 Related Concepts

- [gdb Manual](https://sourceware.org/gdb/documentation/)
- [valgrind](https://valgrind.org/) — memory error detector for C/C++ applications
- [coredumpctl Manual](https://www.freedesktop.org/software/systemd/man/coredumpctl.html)

</details>
