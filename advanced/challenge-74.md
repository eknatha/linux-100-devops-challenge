# Challenge 74: Disk Latency Analysis in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**Disk I/O performance** directly impacts database throughput, application response times, and overall system stability. Slow disks cause processes to spend time in **uninterruptible sleep** (`D` state), which drives up load averages even when CPU utilization appears normal.

**`iostat`** (part of the `sysstat` package) is the primary tool for measuring disk performance — tracking throughput, IOPS, utilization, and most importantly **latency** — the time each I/O request waits to be serviced.

---

## 🧩 Task

Analyze disk I/O performance to identify latency, throughput, and utilization bottlenecks.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
iostat -x 1
```

### How it works

| Part | Description |
|------|-------------|
| `iostat` | I/O Statistics — reports CPU and device I/O metrics |
| `-x` | **Extended** statistics — shows detailed per-device metrics including latency |
| `1` | **Interval** — refresh every **1 second** (continuous monitoring) |

---

## 🔢 Understanding `iostat -x` Output

```bash
iostat -x 1
```

```
Device  r/s    w/s   rkB/s   wkB/s  rrqm/s  wrqm/s  %rrqm  %wrqm  r_await  w_await  aqu-sz  rareq-sz  wareq-sz  svctm  %util
sda     45.0  120.0  3600.0  9600.0    0.5     2.0    1.1    1.6     2.5     18.4     2.1     80.0      80.0      4.8    79.0
sdb      0.2    1.0    12.8    64.0    0.0     0.0    0.0    0.0     0.8      1.2     0.0     64.0      64.0      1.1     0.1
```

### Key Columns Explained

| Column | Description | Warning | Critical |
|--------|-------------|---------|----------|
| `r/s` | Read operations per second (IOPS read) | — | — |
| `w/s` | Write operations per second (IOPS write) | — | — |
| `rkB/s` | Kilobytes read per second (read throughput) | — | — |
| `wkB/s` | Kilobytes written per second (write throughput) | — | — |
| `r_await` | **Average wait time for read requests (ms)** | > 20ms | > 100ms |
| `w_await` | **Average wait time for write requests (ms)** | > 20ms | > 100ms |
| `aqu-sz` | Average I/O queue length (requests waiting) | > 1 | > 4 |
| `%util` | **Percentage of time device was busy** | > 60% | > 90% |
| `svctm` | Average service time per request (ms) — deprecated | — | — |
| `rrqm/s` | Read requests merged per second | — | — |
| `wrqm/s` | Write requests merged per second | — | — |

> 💡 **The three most important columns for diagnosing performance problems:**
> - `r_await` / `w_await` — how long requests wait (latency)
> - `aqu-sz` — how many requests are queued (congestion)
> - `%util` — how busy the device is (saturation)

---

## 📖 Extended Examples

### Example 1 — Basic Extended Stats (1-second refresh)

```bash
iostat -x 1
```

> Continuously prints a new report every second. Press `Ctrl+C` to stop.

---

### Example 2 — Human-Readable Output

```bash
# MB/s instead of kB/s
iostat -x -m 1
```

```
Device  r/s    w/s   rMB/s  wMB/s  r_await  w_await  aqu-sz  %util
sda     45.0  120.0   3.5    9.4     2.5     18.4     2.1    79.0
```

---

### Example 3 — Monitor a Specific Device Only

```bash
# Monitor only /dev/sda
iostat -x 1 -d sda

# Monitor multiple devices
iostat -x 1 -d sda sdb nvme0n1
```

---

### Example 4 — Run for a Fixed Number of Samples

```bash
# 5 samples, 2 seconds apart
iostat -x 2 5
```

```bash
# Skip the first sample (it shows stats since boot, not since monitoring started)
iostat -x 2 5 | tail -n +7
```

> 💡 The **first report** from `iostat` always shows averages since system boot — not the current real-time rate. Skip it or use `tail` to get only the live samples.

---

### Example 5 — Show Only Device Statistics (No CPU)

```bash
iostat -d -x 1
```

> `-d` suppresses the CPU utilization section and shows only device metrics — cleaner output when you only care about disks.

---

### Example 6 — Include Timestamps in Output

```bash
iostat -x -t 1
```

```bash
03/21/2025 10:35:22
Device  r/s   w/s  r_await  w_await  %util
sda     45.0  120.0   2.5    18.4    79.0

03/21/2025 10:35:23
Device  r/s   w/s  r_await  w_await  %util
sda     52.0  135.0   3.1    22.1    85.0
```

> Timestamps are essential when logging `iostat` output to a file for later analysis.

---

### Example 7 — Diagnose High `%util` — Saturated Disk

```bash
iostat -x 1
```

```
Device  r/s    w/s   r_await  w_await  aqu-sz  %util
sda     210.0  480.0   48.5   182.4    12.8    99.8
```

**Diagnosis:**
- `%util` at **99.8%** — disk is fully saturated
- `r_await` at **48.5ms** and `w_await` at **182.4ms** — severe latency (healthy SSDs: < 1ms; HDDs: < 10ms)
- `aqu-sz` at **12.8** — large backlog of pending I/O requests

**Actions to investigate:**
```bash
# Find which processes are driving the I/O
sudo iotop -o

# Find which files are being written to most
sudo lsof | grep -E "REG.*sda"

# Check if a specific service is causing it
sudo systemctl status mysql
journalctl -u mysql -n 50
```

---

### Example 8 — Use `iotop` to Find the I/O Offender

```bash
# Install iotop
sudo apt install iotop    # Debian/Ubuntu
sudo dnf install iotop    # RHEL

# Show only processes actively doing I/O
sudo iotop -o

# Non-interactive batch output
sudo iotop -b -n 5 -o
```

```bash
Total DISK READ:       45.2 MB/s | Total DISK WRITE:      180.8 MB/s
  TID  PRIO  USER    DISK READ  DISK WRITE  SWAPIN  IO>  COMMAND
12345  be/4  mysql   12.5 MB/s  165.4 MB/s   0.0% 82.5%  mysqld
 8901  be/4  appuser  31.2 MB/s   12.3 MB/s   0.0% 12.1%  python3
```

> 💡 `iotop -o` shows only processes **currently doing I/O** — the fastest way to identify which application is thrashing the disk.

---

### Example 9 — Use `ioping` for Latency Testing

```bash
sudo apt install ioping    # Debian/Ubuntu
sudo dnf install ioping    # RHEL

# Measure latency to a specific disk/directory
ioping /var/lib/mysql/

# Test with larger I/O blocks
ioping -s 4k /dev/sda
```

```bash
4 KiB <<< /var/lib/mysql/ (ext4 /dev/sda1 50 GiB):
request=1 time=0.8 ms
request=2 time=1.1 ms
request=3 time=0.9 ms

--- /var/lib/mysql/ (ext4 /dev/sda1 50 GiB) ioping statistics ---
3 requests completed in 2.8 ms, 12 KiB read, 1.07 k iops, 4.29 MiB/s
min/avg/max/mdev = 0.8 ms / 0.9 ms / 1.1 ms / 0.1 ms
```

---

### Example 10 — Benchmark Disk Throughput with `dd`

```bash
# Write speed test (sequential writes)
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=dsync

# Read speed test (sequential reads)
dd if=/tmp/testfile of=/dev/null bs=1G count=1

# Clean up
rm /tmp/testfile
```

```bash
# Write test output:
1073741824 bytes (1.1 GB) copied, 2.53 s, 424 MB/s

# Read test output:
1073741824 bytes (1.1 GB) copied, 0.91 s, 1.18 GB/s
```

> ⚠️ `dd` is a rough benchmark — use `fio` for production-quality I/O testing.

---

### Example 11 — Historical Disk Stats with `sar`

```bash
# Disk I/O stats sampled every 2 seconds, 5 times
sar -d 2 5

# Historical disk stats for today
sar -d

# Historical stats from a specific date
sar -d -f /var/log/sa/sa21
```

```bash
10:35:22    DEV    tps   rkB/s   wkB/s   areq-sz  aqu-sz  await  svctm  %util
10:35:24    sda    165.0  3600.0  9600.0   80.0     2.1    18.4    4.8   79.0
```

> 💡 `sar -d` provides **historical disk performance data** — invaluable when investigating a performance incident that happened before you were alerted.

---

### Example 12 — Disk Latency Monitoring Script

```bash
#!/bin/bash

DEVICE="sda"
INTERVAL=5
AWAIT_THRESHOLD=20          # ms — alert if read or write latency exceeds this
UTIL_THRESHOLD=80           # % — alert if device utilization exceeds this
LOGFILE="/var/log/disk_latency.log"

echo "Monitoring /dev/$DEVICE every ${INTERVAL}s (latency threshold: ${AWAIT_THRESHOLD}ms, util threshold: ${UTIL_THRESHOLD}%)"
echo "Press Ctrl+C to stop."

iostat -x -d $INTERVAL -y $DEVICE 2>/dev/null | awk -v dev="$DEVICE" -v awt="$AWAIT_THRESHOLD" -v ut="$UTIL_THRESHOLD" -v log="$LOGFILE" '
/^Device/ { next }
$1 == dev {
  r_await = $9
  w_await = $10
  util    = $NF
  ts = strftime("%Y-%m-%d %H:%M:%S", systime())

  alert = ""
  if (r_await + 0 > awt) alert = alert " HIGH_READ_LATENCY=" r_await "ms"
  if (w_await + 0 > awt) alert = alert " HIGH_WRITE_LATENCY=" w_await "ms"
  if (util + 0 > ut)     alert = alert " HIGH_UTIL=" util "%"

  if (alert != "")
    msg = "[" ts "] ⚠️  ALERT:" alert
  else
    msg = "[" ts "] ✅ OK r_await=" r_await "ms w_await=" w_await "ms util=" util "%"

  print msg | "tee -a " log
}'
```

---

## 🛠️ Disk I/O Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `iostat -x` | Sampled | Per-device latency, IOPS, throughput, utilization |
| `iotop` | Interactive | Real-time per-process I/O identification |
| `ioping` | Latency test | Measure raw disk latency with micro benchmarks |
| `dd` | Benchmark | Quick sequential read/write throughput test |
| `fio` | Benchmark | Production-quality I/O testing (random, mixed workloads) |
| `sar -d` | Historical | Historical disk I/O investigation |
| `blktrace` | Tracing | Block-level I/O tracing for deep analysis |
| `lsof` | File-level | See which files/processes are doing I/O |

---

## 🔢 Disk Latency Reference

| Storage Type | Expected `r_await` / `w_await` | Warning | Critical |
|-------------|-------------------------------|---------|----------|
| NVMe SSD | < 0.1 ms | > 1 ms | > 5 ms |
| SATA SSD | < 1 ms | > 5 ms | > 20 ms |
| HDD (7200 RPM) | < 10 ms | > 20 ms | > 50 ms |
| Network storage (NFS/iSCSI) | < 5 ms | > 20 ms | > 100 ms |

---

## 🚀 Full Workflow — Diagnose a Disk Latency Problem

```bash
# Step 1: Check overall disk utilization and latency
iostat -x 1

# Step 2: Look for high %util, r_await, w_await, or aqu-sz
# Focus on: %util > 80%, r/w_await > 20ms, aqu-sz > 2

# Step 3: Find which process is driving the I/O
sudo iotop -o

# Step 4: Identify the specific files being accessed
sudo lsof -p <PID> | grep REG

# Step 5: Check if it is read-heavy or write-heavy
iostat -x -m 1
# High rkB/s → read-heavy (check for missing indexes in DB, sequential scans)
# High wkB/s → write-heavy (check for excessive logging, fsync calls)

# Step 6: Test raw disk latency
ioping /path/to/mountpoint

# Step 7: Check for disk errors
sudo dmesg | grep -i "error\|I/O error\|ata\|reset"
sudo smartctl -a /dev/sda   # S.M.A.R.T. health data

# Step 8: Historical context
sar -d

# Step 9: Take action
# Reduce I/O load: tune application, add caching, optimize queries
# Replace hardware: if %util is consistently > 90%, disk is undersized
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| First `iostat` report shows since-boot averages | Skip first report or use `-y` flag to suppress it |
| High `%util` but low `r_await`/`w_await` | Normal for SSDs — NVMe can be 100% utilized with sub-ms latency |
| High load average but low `%util` | Processes may be in `D` state waiting for network storage |
| `iostat` not available | Install sysstat: `sudo apt install sysstat` or `sudo dnf install sysstat` |
| `aqu-sz` is 0 but latency is high | Single-threaded I/O — queue never builds but each request is slow |
| Disk looks healthy but app is slow | Check network storage latency or filesystem-level issues (`df -i` for inodes) |

---

## 🔑 Key Takeaways

- `iostat -x 1` gives a continuous stream of extended disk metrics — **`r_await`, `w_await`, and `%util`** are the key indicators.
- **`r_await`/`w_await`** measure latency — the time each request spends waiting to be served. High latency is the primary cause of slow applications.
- **`%util`** measures saturation — a device consistently at 90%+ is a bottleneck that needs replacement or load redistribution.
- **`aqu-sz` > 1** means requests are queuing — the disk cannot keep up with demand.
- The **first `iostat` sample** always shows averages since boot — ignore it or use `-y` to suppress it.
- Combine `iostat` with **`iotop -o`** to identify which process is causing the I/O load.
- `sar -d` enables **historical investigation** of past disk performance events.

---

## 📚 Related Concepts

- [fio](https://fio.readthedocs.io/) — flexible I/O tester for production benchmarking
- [blktrace](https://linux.die.net/man/8/blktrace) — block layer I/O tracing
- [S.M.A.R.T. / smartctl](https://linux.die.net/man/8/smartctl) — disk health monitoring


</details>
