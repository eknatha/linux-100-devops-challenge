# Challenge 77: Analyze System Calls with strace

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`strace`** is a Linux diagnostic tool that intercepts and records **system calls** (syscalls) made by a process and the signals it receives. Every time a process interacts with the kernel — opening a file, reading data, creating a socket, allocating memory — it makes a system call. `strace` makes these invisible kernel interactions visible.

It is invaluable for:
- Diagnosing why an application fails silently
- Understanding which files and network connections a process uses
- Debugging permission errors with no obvious log output
- Reverse-engineering what a binary does without source code
- Finding performance bottlenecks caused by excessive syscalls

---

## 🧩 Task

Trace the system calls made by a running process to understand its behavior and diagnose issues.

---



<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
strace -p <PID>
```

### How it works

| Part | Description |
|------|-------------|
| `strace` | System call tracer — attaches to a process and logs every kernel interaction |
| `-p <PID>` | **Attach** to an already-running process by its Process ID |

> `strace` can also **launch** a program directly: `strace command arg1 arg2`

---

## 🔢 Understanding `strace` Output

```bash
strace -p 12345
```

```
openat(AT_FDCWD, "/etc/config.conf", O_RDONLY) = 3
read(3, "# Configuration file\nport=8080\n", 4096) = 31
close(3)                                = 0
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 4
connect(4, {sa_family=AF_INET, sin_port=htons(5432), sin_addr=inet_addr("127.0.0.1")}, 16) = -1 ECONNREFUSED (Connection refused)
write(2, "Error: cannot connect to database\n", 34) = 34
exit_group(1)                           = ?
```

| Column | Example | Description |
|--------|---------|-------------|
| Syscall name | `openat` | The system call being made |
| Arguments | `"/etc/config.conf", O_RDONLY` | Arguments passed to the syscall |
| Return value | `= 3` | Return value — file descriptor, bytes read, 0 for success |
| Error | `ECONNREFUSED` | Error name when return is `-1` |
| Description | `(Connection refused)` | Human-readable error description |

### Common Return Values

| Return | Meaning |
|--------|---------|
| `= 0` | Success (for void syscalls) |
| `= N` (positive) | Success — file descriptor, bytes count, etc. |
| `= -1 ERRNO` | **Failure** — `ERRNO` tells you exactly why |

---

## 🗂️ Common System Calls Reference

| Syscall | Description |
|---------|-------------|
| `openat` / `open` | Open a file |
| `read` | Read data from a file descriptor |
| `write` | Write data to a file descriptor |
| `close` | Close a file descriptor |
| `stat` / `fstat` | Get file metadata |
| `mmap` | Map memory |
| `brk` | Change heap size |
| `socket` | Create a network socket |
| `connect` | Connect to a remote host |
| `bind` | Bind socket to an address/port |
| `listen` | Mark socket as passive listener |
| `accept` | Accept incoming connection |
| `sendto` / `recvfrom` | Send/receive network data |
| `execve` | Execute a program |
| `fork` / `clone` | Create a new process/thread |
| `wait4` | Wait for child process |
| `exit_group` | Exit the process |
| `getpid` | Get process ID |
| `ioctl` | Device I/O control |
| `futex` | Fast userspace mutex |

---

## 📖 Extended Examples

### Example 1 — Attach to a Running Process

```bash
sudo strace -p 12345
```

```bash
strace: Process 12345 attached
read(5, 0x7f8b2c001234, 4096)           = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(4, [], 1, 5000)              = 0
read(5, 0x7f8b2c001234, 4096)           = -1 EAGAIN (Resource temporarily unavailable)
```

> Press `Ctrl+C` to detach — the process continues running unaffected.

---

### Example 2 — Launch and Trace a New Command

```bash
# Trace a command from the start
strace ls /tmp

# Trace with full output (no truncation)
strace -s 9999 ls /tmp
```

```bash
execve("/usr/bin/ls", ["ls", "/tmp"], 0x7ffe... /* env */) = 0
openat(AT_FDCWD, "/tmp", O_RDONLY|O_DIRECTORY|O_CLOEXEC) = 3
getdents64(3, /* 12 entries */, 32768)  = 312
write(1, "file1.txt  file2.log  mydir\n", 28) = 28
close(3)                                = 0
exit_group(0)                           = ?
```

---

### Example 3 — Show Timestamps

```bash
# Absolute timestamp for each syscall
strace -t -p 12345

# Relative time since last syscall (useful for finding delays)
strace -r -p 12345

# Time spent IN each syscall (latency measurement)
strace -T -p 12345
```

```bash
# -T output — time in syscall shown on right:
read(5, "", 4096)   = 0 <0.000012>
epoll_wait(4, ...) = 1 <2.004321>    ← 2 seconds waiting here
write(1, "data\n", 5) = 5 <0.000008>
```

> 💡 `-T` is the most useful flag for **performance investigation** — it shows exactly how long the kernel spent in each syscall.

---

### Example 4 — Count Syscall Frequency (Summary Mode)

```bash
# Show a summary of syscall counts and time spent
strace -c ls /tmp
```

```bash
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- --------
 62.45    0.000421          42        10           mmap
 18.32    0.000124          12        10           openat
  8.91    0.000060           6        10           close
  5.44    0.000037           4         9           read
  2.14    0.000014           7         2           write
  ...
------ ----------- ----------- --------- --------- --------
100.00    0.000675                    68         2 total
```

> 💡 `-c` is the fastest way to **identify which syscall is consuming the most time** — the `% time` column immediately shows the bottleneck.

---

### Example 5 — Filter to Specific Syscalls

```bash
# Trace only file-related syscalls
strace -e trace=openat,read,write,close -p 12345

# Trace only network-related syscalls
strace -e trace=network -p 12345

# Trace only process-related syscalls
strace -e trace=process -p 12345

# Trace only memory-related syscalls
strace -e trace=memory -p 12345

# Exclude a specific syscall (e.g., exclude noisy futex calls)
strace -e trace='!futex' -p 12345
```

| `-e trace` Category | Syscalls Included |
|--------------------|------------------|
| `file` | All file I/O: `open`, `read`, `write`, `stat`, etc. |
| `network` | Socket operations: `socket`, `connect`, `bind`, `recv`, etc. |
| `process` | Process management: `fork`, `exec`, `wait`, `exit` |
| `memory` | Memory: `mmap`, `mprotect`, `brk`, `munmap` |
| `signal` | Signal handling: `sigaction`, `kill`, `sigprocmask` |
| `ipc` | Inter-process communication |
| `%file` | Alias for `file` category |

---

### Example 6 — Follow Child Processes

```bash
# Follow all child processes spawned by the traced process
strace -f -p 12345

# Follow children + show PIDs
strace -f -e trace=openat command
```

```bash
[pid 12345] openat(AT_FDCWD, "/etc/hosts", O_RDONLY) = 3
[pid 12346] clone(child_stack=NULL, flags=CLONE_CHILD...) = 12347
[pid 12347] execve("/usr/bin/worker", ...) = 0
```

> 💡 `-f` is essential for tracing **multi-process or multi-threaded applications** — without it, you only see the parent process.

---

### Example 7 — Save strace Output to a File

```bash
# Write output to a file
strace -o /tmp/strace_output.txt -p 12345

# Write with timestamps and follow children
strace -f -T -o /tmp/strace_full.txt -p 12345

# Append to existing file
strace -o /tmp/strace_output.txt -O 12345
```

> 💡 Always save to a file for complex sessions — `strace` output can scroll too fast to read in a terminal.

---

### Example 8 — Diagnose a Permission Denied Error

```bash
# Trace only syscalls returning EACCES or EPERM (permission errors)
strace -e trace=file command 2>&1 | grep -E "EACCES|EPERM"
```

```bash
openat(AT_FDCWD, "/etc/shadow", O_RDONLY) = -1 EACCES (Permission denied)
openat(AT_FDCWD, "/var/log/secure", O_RDONLY) = -1 EACCES (Permission denied)
```

> Now you know **exactly which files** are causing the permission error — no guesswork.

---

### Example 9 — Find Which Files a Process Opens

```bash
# Trace only file open syscalls for a running process
strace -e trace=openat -p 12345 2>&1 | grep -v ENOENT

# Or trace a command and extract all opened files
strace -e trace=openat ls /tmp 2>&1 | awk -F'"' '/openat/{print $2}'
```

```bash
/etc/ld.so.cache
/lib/x86_64-linux-gnu/libselinux.so.1
/lib/x86_64-linux-gnu/libc.so.6
/tmp
```

> This technique reveals **exactly which configuration files, libraries, and data files** a process accesses — useful for security auditing and container image optimization.

---

### Example 10 — Diagnose a Network Connection Failure

```bash
strace -e trace=network -p 12345
```

```bash
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 4
connect(4, {sa_family=AF_INET, sin_port=htons(5432), sin_addr=inet_addr("127.0.0.1")}, 16) = -1 ECONNREFUSED (Connection refused)
```

> Instantly confirms: the process is trying to connect to **PostgreSQL on port 5432 at 127.0.0.1** and getting `ECONNREFUSED` — the database is not running or is on a different host.

---

### Example 11 — Find Slow Syscalls (Performance Profiling)

```bash
# Trace with timing and sort by slowest syscall
strace -c -p 12345

# Real-time view — find the single slowest call
strace -T -p 12345 2>&1 | awk -F'<' '{if(NF>1 && $2+0 > 0.01) print $0}' | head -20
```

```bash
# Output showing slow calls (> 10ms):
epoll_wait(4, [{...}], 1024, 5000) = 1 <2.004231>    ← waiting 2s for I/O
read(7, "SELECT * FROM users\n", 4096) = 20 <0.000015>
write(7, "ERROR: timeout\n", 15) = 15 <0.000009>
```


---

## 🛠️ strace vs Related Tools

| Tool | What It Traces | Best For |
|------|---------------|----------|
| `strace` | System calls (user → kernel) | File access, network, process behavior |
| `ltrace` | Library calls (user → shared libs) | libc function calls, higher-level behavior |
| `perf` | CPU events, kernel functions | Performance profiling, hot paths |
| `bpftrace` | Kernel + user probes via eBPF | Advanced tracing without process slowdown |
| `ftrace` | Kernel function tracing | Deep kernel investigation |
| `gdb` | Full debugger | Source-level debugging with breakpoints |

---

## 🔢 Common Error Codes Reference

| errno | Name | Meaning |
|-------|------|---------|
| 1 | `EPERM` | Operation not permitted |
| 2 | `ENOENT` | No such file or directory |
| 13 | `EACCES` | Permission denied |
| 17 | `EEXIST` | File already exists |
| 22 | `EINVAL` | Invalid argument |
| 28 | `ENOSPC` | No space left on device |
| 32 | `EPIPE` | Broken pipe |
| 98 | `EADDRINUSE` | Address already in use |
| 99 | `EADDRNOTAVAIL` | Address not available |
| 110 | `ETIMEDOUT` | Connection timed out |
| 111 | `ECONNREFUSED` | Connection refused |

---

## 🚀 Full Workflow — Debug a Failing Application with strace

```bash
# Step 1: Find the PID of the failing process
pgrep -a myapp
# OR
ps aux | grep myapp

# Step 2: Quick summary to understand what it's doing
sudo strace -c -p <PID>

# Step 3: Attach and watch in real time
sudo strace -f -T -p <PID>

# Step 4: Filter to only the relevant syscall category
sudo strace -e trace=file -p <PID>        # File issues
sudo strace -e trace=network -p <PID>     # Network issues

# Step 5: Look for errors specifically
sudo strace -p <PID> 2>&1 | grep "= -1"

# Step 6: Find permission problems
sudo strace -e trace=openat -p <PID> 2>&1 | grep -E "EACCES|EPERM"

# Step 7: Save full trace for later analysis
sudo strace -f -T -s 9999 -o /tmp/full_trace.txt -p <PID>

# Step 8: Generate a syscall summary from the saved trace
grep "= -1" /tmp/full_trace.txt | awk '{print $NF}' | sort | uniq -c | sort -rn
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| strace slows down the traced process significantly | Use `-c` for summary only; use `-e trace=` to narrow scope; use `bpftrace` for production |
| Output scrolls too fast to read | Always use `-o file.txt` to save to a file |
| Multithreaded app output is mixed up | Add `-f` to label each line with `[pid XXXX]` |
| Missing child process behavior | Add `-f` to follow forks and clones |
| String arguments are truncated | Use `-s 9999` to increase string length limit |
| `strace` not installed | Install: `sudo apt install strace` or `sudo dnf install strace` |
| Attaching to a process fails | Requires root or `CAP_PTRACE` capability |

---

## 🔑 Key Takeaways

- `strace -p PID` **attaches to a running process** and logs every system call it makes.
- `strace command` **launches and traces** a new command from the beginning.
- **`-c`** gives a summary of syscall counts and time — the fastest way to find the bottleneck.
- **`-T`** shows time spent in each syscall — use it to find slow kernel operations.
- **`-e trace=file|network|process`** narrows output to only the relevant category — essential for readability.
- **`-f`** follows child processes and threads — required for multi-process applications.
- Filter for `= -1` lines to instantly see all **failing syscalls and their error codes**.
- Be aware that `strace` **adds significant overhead** — use it with caution in production, or prefer `bpftrace` for low-overhead tracing.

---

## 📚 Related Concepts

- [ltrace](https://linux.die.net/man/1/ltrace) — library call tracer (user-space equivalent of strace)
- [bpftrace](https://bpftrace.org/) — modern low-overhead tracing using eBPF
- [perf](https://perf.wiki.kernel.org/) — Linux performance profiling framework

</details>
