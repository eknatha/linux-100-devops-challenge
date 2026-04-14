# Challenge 37: Analyze Disk I/O in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Disk I/O performance** directly impacts databases, web servers, log processing, and any application that reads or writes data. When a system becomes slow and CPU usage appears normal, the bottleneck is often disk I/O — processes blocked waiting for data to be read from or written to disk.

Key metrics to monitor:
- **Throughput** — how much data is being read/written per second (MB/s)
- **IOPS** — how many I/O operations per second
- **Latency** — how long each I/O request takes (milliseconds)
- **Utilization** — what percentage of time the disk is busy

---

## 🧩 Task

Analyze disk I/O performance to identify bottlenecks and measure throughput.

---

## ✅ Solution


<details>
<summary>💡 Click to view solution</summary>

```bash
# Extended disk I/O statistics (1-second intervals)
iostat -x 1

# Interactive per-process I/O monitor
sudo iotop

# Disk I/O summary
vmstat 1
```

### How it works

| Command | Description |
|---------|-------------|
| `iostat -x 1` | Extended per-device I/O stats every 1 second |
| `sudo iotop` | Real-time per-process disk I/O monitor |
| `vmstat 1` | System-wide stats including I/O activity |

---

## 🔢 Understanding `iostat -x` Output

```bash
iostat -x 1
```

```
Device  r/s    w/s   rkB/s   wkB/s  r_await  w_await  aqu-sz  %util
sda     45.0  480.0  3600.0 38400.0    2.5    182.4     8.1    99.8
sdb      0.2    1.0    12.8    64.0    0.8      1.2     0.0     0.1
```

| Column | Description | Warning | Critical |
|--------|-------------|---------|---------|
| `r/s` | Read operations per second | — | — |
| `w/s` | Write operations per second | — | — |
| `rkB/s` | Read throughput (KB/s) | — | — |
| `wkB/s` | Write throughput (KB/s) | — | — |
| `r_await` | Read request latency (ms) | > 20ms | > 100ms |
| `w_await` | Write request latency (ms) | > 20ms | > 100ms |
| `aqu-sz` | Average I/O queue length | > 1 | > 4 |
| `%util` | % of time device was busy | > 60% | > 90% |

> 💡 **The three key indicators of a disk bottleneck:**
> - `%util` approaching **100%** — disk is saturated
> - `r_await`/`w_await` above **20ms** — high latency
> - `aqu-sz` above **1** — requests are queuing up

---

## 📖 Extended Examples

### Example 1 — Basic `iostat`

```bash
# Install sysstat if needed
sudo apt install sysstat    # Debian/Ubuntu
sudo dnf install sysstat    # RHEL

# Basic I/O stats
iostat

# Extended stats every 1 second
iostat -x 1
```

---

### Example 2 — Human-Readable Output in MB/s

```bash
# Show throughput in MB/s instead of KB/s
iostat -x -m 1
```

```bash
Device  r/s   w/s  rMB/s  wMB/s  r_await  w_await  aqu-sz  %util
sda     45.0  480.0   3.5   37.5     2.5    182.4     8.1   99.8
```

---

### Example 3 — Monitor a Specific Device

```bash
# Monitor only /dev/sda
iostat -x 1 -d sda

# Monitor NVMe device
iostat -x 1 -d nvme0n1

# Monitor multiple specific devices
iostat -x 1 -d sda sdb
```

---

### Example 4 — Skip the First Report (Boot Average)

```bash
# The first iostat report shows averages since boot — skip it
iostat -x 2 5 | tail -n +7

# Or use -y to skip the first report
iostat -xy 1
```

> ⚠️ The **first `iostat` report** always shows averages since system boot — not current activity. Skip it to see real-time metrics.

---

### Example 5 — Real-Time Per-Process I/O with `iotop`

```bash
# Show all processes with I/O activity
sudo iotop

# Show only processes actively doing I/O
sudo iotop -o

# Non-interactive batch output (5 samples)
sudo iotop -b -n 5 -o
```

```bash
# iotop -o output:
Total DISK READ:  3.5 MB/s  | Total DISK WRITE: 37.5 MB/s
  TID USER     DISK READ  DISK WRITE  SWAPIN    IO>  COMMAND
 2341 mysql    0.3 MB/s   36.8 MB/s    0.0%  92.1%  mysqld
12345 appuser  3.2 MB/s    0.7 MB/s    0.0%   7.9%  python3
```

> 💡 `iotop -o` is the **fastest way to find which process is thrashing the disk** — it shows only active I/O consumers.

---

### Example 6 — I/O Wait in `vmstat`

```bash
vmstat 1 5
```

```bash
procs ---memory-- ---swap-- --io-- -system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 3  2      0 124000 65536 643000    0    0 12288 45056  850 1450 78 10  6  6  0
```

| Column | Description |
|--------|-------------|
| `b` | Processes **blocked** waiting for I/O |
| `bi` | **Blocks in** — data read from disk (blocks/s) |
| `bo` | **Blocks out** — data written to disk (blocks/s) |
| `wa` | **CPU I/O wait %** — CPU idle waiting for disk |

> High `wa` (I/O wait %) in `vmstat` means the CPU is idle but waiting for disk — the bottleneck is disk, not CPU.

---

### Example 7 — Check Disk I/O with `dstat`

```bash
# Install dstat
sudo apt install dstat

# Disk I/O stats every 1 second
dstat -d 1

# Combined view: CPU, disk, network, memory
dstat -cdnm 1
```

```bash
# dstat -d output:
-dsk/total-
 read  writ
  45k  480k
 3.5M 37.5M    ← High write activity
```

---

### Example 8 — Benchmark Sequential Read/Write with `dd`

```bash
# Write speed test (sequential writes)
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=dsync
```

```bash
# Output:
1073741824 bytes (1.1 GB) copied, 2.53 s, 424 MB/s
```

```bash
# Read speed test (sequential reads)
dd if=/tmp/testfile of=/dev/null bs=1G count=1

# Clean up
rm /tmp/testfile
```

> ⚠️ `dd` measures **sequential** I/O only. For realistic random I/O benchmarks (databases, etc.), use `fio`.

---

### Example 9 — Advanced Benchmarking with `fio`

```bash
# Install fio
sudo apt install fio    # Debian/Ubuntu
sudo dnf install fio    # RHEL

# Test random read performance (simulates database workload)
fio --name=randread \
    --ioengine=libaio \
    --rw=randread \
    --bs=4k \
    --direct=1 \
    --size=1G \
    --numjobs=4 \
    --runtime=60 \
    --output-format=json \
    --filename=/tmp/fio_test

# Clean up
rm /tmp/fio_test
```

---

### Example 10 — Check Disk Errors and Health

```bash
# Check for disk errors in kernel messages
dmesg | grep -i "error\|I/O error\|ata\|reset"

# Check S.M.A.R.T. disk health
sudo apt install smartmontools
sudo smartctl -a /dev/sda

# Check for bad sectors
sudo smartctl -t short /dev/sda    # Run short test
sudo smartctl -l selftest /dev/sda # View test results
```

```bash
# dmesg error example:
[12345.678] ata1.00: failed command: READ FPDMA QUEUED
[12345.679] ata1.00: error: { UNC }
# → Hardware read error — disk may be failing
```

---

### Example 11 — Historical Disk I/O with `sar`

```bash
# Historical disk I/O for today
sar -d

# Disk I/O sampled every 2 seconds, 5 times
sar -d 2 5

# Historical data from a specific date
sar -d -f /var/log/sa/sa21

# Enable sysstat for historical collection
sudo systemctl enable --now sysstat
```

```bash
# sar -d output:
10:35:22    DEV  tps  rkB/s  wkB/s  areq-sz  aqu-sz  await  svctm  %util
10:35:24    sda  165  3600   9600     80.0     2.1    18.4    4.8   79.0
```

---

### Example 12 — Diagnose High I/O Wait

```bash
#!/bin/bash
# Diagnose high I/O wait

echo "=== I/O Wait Check ==="
IOWAIT=$(vmstat 1 3 | tail -1 | awk '{print $16}')
echo "Current I/O wait: ${IOWAIT}%"

if [ "$IOWAIT" -gt 10 ]; then
  echo "⚠️  HIGH I/O WAIT — investigating..."

  echo -e "\n=== Top I/O Consuming Processes ==="
  sudo iotop -b -n 1 -o 2>/dev/null | head -10

  echo -e "\n=== Disk Utilization ==="
  iostat -x 1 2 | grep -v "^$\|^Linux" | tail -10

  echo -e "\n=== Processes Blocked on I/O ==="
  ps aux | awk '$8 == "D" {print "PID:", $2, "CMD:", $11}'
fi
```

---

## 🗂️ Disk I/O Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `iostat -x` | Sampled | Per-device latency, IOPS, utilization |
| `iotop` | Interactive | Per-process I/O identification |
| `vmstat` | Sampled | System-wide I/O wait and blocked processes |
| `dstat` | Sampled | Combined disk + CPU + network view |
| `dd` | Benchmark | Quick sequential read/write speed |
| `fio` | Benchmark | Production-quality I/O testing |
| `sar -d` | Historical | Past disk performance investigation |
| `smartctl` | Health check | Disk hardware health and errors |

---

## 🔢 Disk Latency Reference

| Storage Type | Expected Latency | Warning | Critical |
|-------------|-----------------|---------|---------|
| NVMe SSD | < 0.1 ms | > 1 ms | > 5 ms |
| SATA SSD | < 1 ms | > 5 ms | > 20 ms |
| HDD (7200 RPM) | < 10 ms | > 20 ms | > 50 ms |
| Network storage | < 5 ms | > 20 ms | > 100 ms |

---

## 🚀 Full Workflow — Diagnose a Disk I/O Bottleneck

```bash
# Step 1: Check for high I/O wait
vmstat 1 5    # Look for high 'wa' column

# Step 2: Identify which disk is busy
iostat -x 1   # Look for %util > 70%, high r/w_await, large aqu-sz

# Step 3: Find which process is doing the I/O
sudo iotop -o

# Step 4: Check if it's read or write heavy
iostat -xm 1    # Compare rMB/s vs wMB/s

# Step 5: Check for processes blocked on I/O
ps aux | awk '$8 == "D" {print "Blocked:", $2, $11}'

# Step 6: Check disk health
dmesg | grep -i "error\|I/O error"
sudo smartctl -a /dev/sda

# Step 7: Historical context
sar -d

# Step 8: Take action
# Tune application (reduce writes, add caching)
# Upgrade storage (HDD → SSD → NVMe)
# Distribute I/O across multiple disks
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| First `iostat` report shows since-boot averages | Skip first sample or use `-y` flag |
| High `%util` but low latency | Normal for SSDs — NVMe can be 100% utilized with sub-ms latency |
| High CPU load but disk looks fine | Check `wa` in `vmstat` — I/O wait doesn't show as CPU usage |
| `iotop` not installed | `sudo apt install iotop` or `sudo dnf install iotop` |
| `iostat` not available | Install sysstat: `sudo apt install sysstat` |
| `dd` shows good speed but app is slow | `dd` tests sequential I/O — use `fio` for random I/O patterns |

---

## 🔑 Key Takeaways

- `iostat -x 1` is the primary disk performance tool — watch **`%util`**, **`r_await`**, **`w_await`**, and **`aqu-sz`**.
- **`%util > 90%`** means the disk is saturated — it is the bottleneck.
- **High `wa` in `vmstat`** means the CPU is waiting on disk — the I/O system is the constraint, not CPU.
- `iotop -o` immediately shows **which process is driving the disk I/O**.
- The **first `iostat` sample** shows since-boot averages — skip it for real-time metrics.
- Processes in **`D` state** (uninterruptible sleep) are blocked on disk I/O — check with `ps aux | awk '$8 == "D"'`.
- Use **`dd`** for quick sequential benchmarks; use **`fio`** for realistic production-like testing.

---

## 📚 Related Concepts

- [fio Documentation](https://fio.readthedocs.io/)
- [sysstat (iostat, sar)](https://github.com/sysstat/sysstat)

</details>
