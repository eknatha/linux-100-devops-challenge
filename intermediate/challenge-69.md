# Challenge 69: Identify Zombie Processes in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
**eknatha**

## 📌 Overview

A **zombie process** (also called a defunct process) is a process that has **finished executing** but still has an entry in the process table because its parent has not yet read its exit status. The process is dead — it consumes no CPU or memory — but its PID remains occupied until the parent cleans it up via the `wait()` system call.

A few zombies are normal. A **large accumulation** of zombie processes signals a bug in the parent process and can eventually **exhaust the PID table**, preventing new processes from starting.

---

## 🧩 Task

Find all zombie processes running on the system.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
ps aux | grep Z
```

### How it works

| Part | Description |
|------|-------------|
| `ps aux` | List **all** processes with detailed info (`a`=all users, `u`=user format, `x`=include processes without a terminal) |
| `\|` | Pipe output into the next command |
| `grep Z` | Filter lines where the **STAT** column contains `Z` (zombie state) |

### `ps aux` Output Columns

```
USER       PID  %CPU  %MEM    VSZ   RSS  TTY   STAT  START   TIME  COMMAND
root         1   0.0   0.1  16952  5612  ?     Ss    Mar20   0:05  /sbin/init
devuser   4821   0.0   0.0      0     0  ?     Z     10:05   0:00  [defunct]
```

| Column | Description |
|--------|-------------|
| `USER` | Owner of the process |
| `PID` | Process ID |
| `%CPU` | CPU usage |
| `%MEM` | Memory usage |
| `STAT` | Process state — `Z` = zombie |
| `COMMAND` | Command name — zombies show `[defunct]` |

---

## 🔄 Process State Codes

| State Code | Meaning |
|------------|---------|
| `R` | Running or runnable |
| `S` | Sleeping (interruptible) |
| `D` | Uninterruptible sleep (I/O wait) |
| `T` | Stopped (by signal or debugger) |
| `Z` | **Zombie — defunct, waiting for parent** |
| `I` | Idle kernel thread |
| `<` | High priority |
| `N` | Low priority (nice) |
| `s` | Session leader |
| `+` | Foreground process group |

---

## 🧠 Zombie Process Lifecycle

```
  Parent Process
       │
       ├── fork() ──► Child Process starts
       │                    │
       │                    │  (executes and finishes)
       │                    │
       │               Child exits ──► becomes ZOMBIE
       │                    │          (PID held in process table)
       │                    │
       └── wait() ─────────►│  Parent reads exit status
                             │
                        Zombie cleaned up ──► PID released
```

> If the **parent never calls `wait()`**, the zombie persists until:
> - The parent process itself terminates (zombie is then adopted by `init`/PID 1, which cleans it up)
> - The system is rebooted

---

## 📖 Extended Examples

### Example 1 — Basic Zombie Search

```bash
ps aux | grep Z
```

```bash
devuser  4821  0.0  0.0   0   0  ?  Z  10:05  0:00  [defunct]
devuser  4822  0.0  0.0   0   0  ?  Z  10:05  0:00  [defunct]
root     4901  0.0  0.0   0   0  ?  Z  10:06  0:00  [defunct]
```

> ⚠️ Note: `grep Z` may also match other lines containing the letter `Z` (e.g., the `grep` process itself or usernames). Use the more precise methods below for accurate results.

---

### Example 2 — Precise Zombie Detection (Recommended)

```bash
# Match only lines where the STAT column is exactly Z
ps aux | awk '$8 == "Z"'
```

```bash
devuser  4821  0.0  0.0   0   0  ?  Z  10:05  0:00  [defunct]
devuser  4822  0.0  0.0   0   0  ?  Z  10:05  0:00  [defunct]
```

---

### Example 3 — Count Total Zombie Processes

```bash
ps aux | awk '$8 == "Z"' | wc -l
```

```bash
3
```

```bash
# One-liner with label
echo "Zombie count: $(ps aux | awk '$8 == "Z"' | wc -l)"
```

```bash
Zombie count: 3
```

---

### Example 4 — List Zombies with PID and Parent PID

```bash
ps -eo pid,ppid,stat,comm | awk '$3 == "Z"'
```

```bash
  PID  PPID STAT COMMAND
 4821  4800    Z [defunct]
 4822  4800    Z [defunct]
 4901  4890    Z [defunct]
```

> 💡 The **PPID** (Parent Process ID) is the key — it tells you which parent is responsible for the zombie children.

---

### Example 5 — Identify the Parent Process Responsible

```bash
# Get PPID of zombie processes
ps -eo pid,ppid,stat,comm | awk '$3 == "Z" {print $2}' | sort -u
```

```bash
4800
4890
```

```bash
# Look up what the parent process is
ps -p 4800 -o pid,ppid,stat,comm,cmd
```

```bash
  PID  PPID STAT COMMAND  CMD
 4800  1234    S  myapp   /usr/bin/python3 /opt/myapp/server.py
```

> The parent `myapp` (PID 4800) is not calling `wait()` — it is the **source of the zombie accumulation**.

---

### Example 6 — Send SIGCHLD to the Parent (Soft Fix)

```bash
# Signal the parent to check and reap its children
kill -SIGCHLD 4800
```

> `SIGCHLD` tells the parent process that one of its children has changed state. A well-written parent will respond by calling `wait()` and cleaning up the zombie. This is the **least disruptive fix**.

---

### Example 7 — Kill the Parent Process to Force Cleanup

If `SIGCHLD` does not work, terminating the parent forces all zombie children to be **reparented to PID 1** (`init`/`systemd`), which immediately reaps them.

```bash
# Graceful termination
kill -SIGTERM 4800

# Force kill if SIGTERM is ignored
kill -SIGKILL 4800
```

> ⚠️ Only kill the parent if it is safe to do so — this will also terminate all living child processes.

---

### Example 8 — Check for Zombies Using `top`

```bash
top
```

```bash
Tasks: 210 total,   1 running, 206 sleeping,   0 stopped,   3 zombie
```

> The `zombie` count is shown directly in the `Tasks` line — the fastest visual check. A non-zero value warrants investigation.

---

### Example 9 — Use `htop` to Visually Find Zombies

```bash
htop
```

Then filter:
- Press `F4` (Filter) and type `Z`
- Or press `F5` (Tree) to see the parent-child relationship

> `htop` color-codes zombie processes and lets you navigate the process tree to find the parent visually.

---

### Example 10 — Check System-Wide with `/proc`

```bash
# Count zombies via the /proc filesystem
cat /proc/[0-9]*/status 2>/dev/null | grep -c "^State:.*Z"
```

```bash
3
```

```bash
# View zombie process details via /proc
for pid in $(ps -eo pid,stat | awk '$2 == "Z" {print $1}'); do
  echo "=== Zombie PID: $pid ==="
  cat /proc/$pid/status 2>/dev/null | grep -E "^(Name|State|PPid)"
done
```

```bash
=== Zombie PID: 4821 ===
Name:   myworker
State:  Z (zombie)
PPid:   4800
```

---

### Example 11 — Zombie Monitoring Script

```bash
#!/bin/bash

THRESHOLD=5          # Alert if zombies exceed this count
LOGFILE="/var/log/zombie_monitor.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

ZOMBIES=$(ps -eo pid,ppid,stat,comm | awk '$3 == "Z"')
COUNT=$(echo "$ZOMBIES" | grep -c "Z" 2>/dev/null || echo 0)

if [ "$COUNT" -gt "$THRESHOLD" ]; then
  echo "[$DATE] ⚠️  WARNING: $COUNT zombie processes detected!" | tee -a "$LOGFILE"
  echo "$ZOMBIES" | tee -a "$LOGFILE"

  # Identify parent processes
  echo -e "\n[Parent Processes Responsible]"
  ps -eo pid,ppid,stat,comm | awk '$3 == "Z" {print $2}' | sort -u | while read ppid; do
    echo "  Parent PID $ppid: $(ps -p $ppid -o cmd= 2>/dev/null || echo 'already gone')"
  done | tee -a "$LOGFILE"
else
  echo "[$DATE] ✅ OK: $COUNT zombie processes (threshold: $THRESHOLD)" >> "$LOGFILE"
fi
```

---

## 🗂️ Zombie vs Other Problem Processes

| Process Type | State | CPU | Memory | Risk | Fix |
|-------------|-------|-----|--------|------|-----|
| **Zombie** | `Z` | 0% | 0 | PID exhaustion | Kill parent or send SIGCHLD |
| **Orphan** | `S`/`R` | Varies | Yes | Running unchecked | Kill directly |
| **Defunct** | `Z` | 0% | 0 | Same as zombie | Same as zombie |
| **Uninterruptible** | `D` | Varies | Yes | I/O hang | Fix I/O issue or reboot |
| **Stopped** | `T` | 0% | Yes | Resource lock | Send SIGCONT or kill |

---

## 🚀 Full Workflow — Find and Resolve Zombie Processes

```bash
# Step 1: Check if any zombies exist
ps aux | awk '$8 == "Z"'

# Step 2: Count them
echo "Zombies: $(ps aux | awk '$8 == "Z"' | wc -l)"

# Step 3: Find the parent responsible
ps -eo pid,ppid,stat,comm | awk '$3 == "Z" {print "Zombie:", $1, "Parent:", $2}'

# Step 4: Identify the parent application
ps -p <PPID> -o pid,ppid,cmd

# Step 5: Try a soft fix first — signal the parent
kill -SIGCHLD <PPID>

# Step 6: Check if zombies are gone
ps aux | awk '$8 == "Z"' | wc -l

# Step 7: If still present, terminate the parent (if safe)
kill -SIGTERM <PPID>

# Step 8: Force kill if SIGTERM is ignored
kill -SIGKILL <PPID>

# Step 9: Verify cleanup
ps aux | awk '$8 == "Z"'
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `grep Z` also matches the grep process itself | Use `awk '$8 == "Z"'` for precise state matching |
| Trying to kill a zombie directly | You **cannot** kill a zombie — it is already dead. Kill the **parent** instead |
| Assuming zombies use resources | Zombies consume no CPU or memory — only a PID slot |
| Single zombie = immediate crisis | A few zombies are normal; only a growing count is a problem |
| Killing parent without checking its children | All living children will also be affected — verify first |
| Zombies return after clearing | The parent has a bug — it needs to be fixed at the code level |

---

## 🔑 Key Takeaways

- A zombie process has **exited but not been reaped** by its parent — it shows `Z` in the STAT column and `[defunct]` as the command.
- Zombies consume **no CPU or memory** — only a PID slot.
- You **cannot kill a zombie directly** — send `SIGCHLD` to the parent or kill the parent process.
- Use `ps -eo pid,ppid,stat,comm | awk '$3 == "Z"'` for accurate zombie detection without false positives.
- The **PPID** column tells you which process is responsible for creating the zombies.
- A growing zombie count indicates a **programming bug** in the parent — the real fix is at the application level.
- `top` shows zombie count instantly in the Tasks summary line.

---

## 📚 Related Concepts

- [Linux Process States](https://man7.org/linux/man-pages/man1/ps.1.html)
- [The `wait()` System Call](https://man7.org/linux/man-pages/man2/wait.2.html) — how parents reap children
- [proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html) — `/proc/[PID]/status` for raw process details

</details>
