# Challenge 88: Monitor Real-Time System Metrics in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Real-time system metrics monitoring** gives you a live, continuously updating view of every critical resource on a Linux system — CPU, memory, disk I/O, network throughput, and running processes. Rather than running individual commands one at a time, dedicated monitoring tools aggregate all metrics into a single dashboard that updates every second.

This is essential for:
- Diagnosing performance problems as they happen
- Watching a system during a load test or deployment
- Identifying resource bottlenecks in real time
- Detecting unexpected spikes before they become outages

---

## 🧩 Task

Monitor CPU, memory, disk, network, and process metrics in real time using Linux tools.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Classic real-time monitor
top

# Enhanced color-coded monitor (recommended)
htop

# All-in-one comprehensive dashboard
glances

# CPU + memory + disk + network combined
dstat -cdnm 1
```

### How it works

| Command | Description |
|---------|-------------|
| `top` | Built-in real-time process and resource monitor |
| `htop` | Enhanced color-coded interactive monitor with mouse support |
| `glances` | All-in-one dashboard covering all subsystems |
| `dstat` | Customizable real-time stats for all subsystems |

---

## 🔢 Key Metrics to Monitor

```
┌─────────────────────────────────────────────────────────────┐
│                    Real-Time Metrics                        │
├──────────────┬──────────────┬──────────────┬───────────────┤
│     CPU      │    Memory    │     Disk     │    Network    │
├──────────────┼──────────────┼──────────────┼───────────────┤
│ %user (%us)  │ Total RAM    │ Read MB/s    │ Recv MB/s     │
│ %system(%sy) │ Used RAM     │ Write MB/s   │ Send MB/s     │
│ %iowait(%wa) │ Available    │ IOPS (r/s)   │ Connections   │
│ %idle (%id)  │ Swap used    │ %util        │ Errors/drops  │
│ Load avg     │ Buff/cache   │ Latency(ms)  │ Retransmits   │
└──────────────┴──────────────┴──────────────┴───────────────┘
```

---

## 📖 Extended Examples

### Example 1 — `top` — Classic Real-Time Monitor

```bash
top
```

```bash
top - 10:35:22 up 5 days, 3:17, 2 users, load average: 1.42, 1.21, 0.98
Tasks: 215 total,   2 running, 213 sleeping,   0 stopped,   0 zombie
%Cpu(s): 18.2 us,  3.3 sy,  0.0 ni, 76.8 id,  1.5 wa,  0.0 hi,  0.2 si
MiB Mem : 15,820.0 total,  5,812.0 free,  9,341.0 used,    667.0 buff/cache
MiB Swap:  2,048.0 total,  2,048.0 free,      0.0 used.  6,021.0 avail Mem

  PID USER     PR  NI    VIRT    RES %CPU %MEM     TIME+ COMMAND
12345 appuser  20   0 2145000 512000 85.2  3.2  45:12 python3
 2341 mysql    20   0 1892000 324000 14.3  2.0  15:21 mysqld
```

**`top` real-time keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `P` | Sort by **CPU** |
| `M` | Sort by **Memory** |
| `1` | Toggle **per-core** CPU bars |
| `k` | **Kill** a process |
| `r` | **Renice** a process |
| `u` | Filter by **user** |
| `d` | Change **refresh interval** |
| `q` | **Quit** |

---

### Example 2 — `htop` — Enhanced Visual Monitor

```bash
htop
```

```bash
# Install if needed
sudo apt install htop    # Debian/Ubuntu
sudo dnf install htop    # RHEL/CentOS
```

**What `htop` adds over `top`:**
- **Color-coded per-CPU bars** at the top
- **Visual memory and swap bars** with color legend
- **Mouse support** — click column headers to sort
- **Process tree** view (`F5`)
- **Filter** by name (`F4`)
- **Kill** without knowing PID (`F9`)

**`htop` memory bar colors:**
- 🟢 **Green** — used by processes
- 🔵 **Blue** — buffer memory
- 🟡 **Yellow** — cache
- 🔴 **Red** — swap in use

**`htop` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `F2` | **Setup** — configure columns |
| `F4` | **Filter** by name |
| `F5` | Toggle **tree** view |
| `F6` | **Sort** by column |
| `F9` | **Kill** selected process |
| `u` | Filter by **user** |
| `q` | **Quit** |

---

### Example 3 — `glances` — All-in-One Dashboard

```bash
# Install
sudo apt install glances    # Debian/Ubuntu
sudo dnf install glances    # RHEL
pip install glances          # via pip

# Launch
glances
```

**What `glances` shows in one screen:**
- CPU usage with per-core breakdown
- Memory, swap, and available RAM
- Disk I/O — read/write per device
- Network — recv/sent per interface
- Process list with color alerts
- Filesystem usage with alert thresholds
- System uptime and load average
- GPU usage (if applicable)
- Temperature sensors (if available)

```bash
# Remote monitoring — run as web server
glances -w
# Access at: http://localhost:61208

# Connect to a remote glances server
glances -c remote-server-ip

# Export metrics to a file
glances --export csv --export-csv-file /tmp/metrics.csv
```

---

### Example 4 — `dstat` — Customizable Combined Stats

```bash
# Install
sudo apt install dstat

# CPU + disk + network + memory (all at once)
dstat -cdnm 1

# All statistics
dstat -a 1

# Custom: CPU + memory + network
dstat -cmn 1
```

```bash
# dstat -cdnm 1 output:
----total-usage---- -dsk/total- -net/total- ------memory-usage-----
usr sys idl wai stl| read  writ| recv  send| used  free  buff  cach
 18   3  77   1   0| 512k 3500k|12.3k 45.1k|9341M 5812M  262M  642M
 15   2  82   1   0|   0  1200k|8.1k  32.4k|9342M 5811M  262M  642M
```

**`dstat` plugin options:**

| Flag | Shows |
|------|-------|
| `-c` | CPU usage |
| `-d` | Disk I/O |
| `-n` | Network I/O |
| `-m` | Memory |
| `-s` | Swap |
| `-p` | Process stats |
| `-l` | Load average |
| `-a` | All of the above |

---

### Example 5 — `vmstat` — Classic System Stats

```bash
# Sample every 1 second
vmstat 1

# Sample every 2 seconds, 10 times
vmstat 2 10
```

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 5812000 262000 642000   0    0     8    22  210  430 18  3 77  1  0
```

**Key columns to monitor:**

| Column | Warning Sign |
|--------|-------------|
| `r` | Run queue > CPU cores = CPU overloaded |
| `b` | > 2 = processes blocked on I/O |
| `si`/`so` | > 0 = active swap = memory pressure |
| `wa` | > 10% = disk bottleneck |
| `id` | < 10% = system heavily loaded |

---

### Example 6 — `iostat` — Real-Time Disk Metrics

```bash
# Extended disk stats every 1 second
iostat -x 1

# In MB/s
iostat -xm 1
```

```bash
Device  r/s   w/s  rMB/s  wMB/s  r_await  w_await  aqu-sz  %util
sda     10.0  45.0    0.5    3.5     1.2      8.4     0.4    42.0
nvme0n1  0.5   2.0    0.1    0.5     0.1      0.2     0.0     0.5
```

---

### Example 7 — `sar` — Historical + Real-Time All Metrics

```bash
# Install sysstat
sudo apt install sysstat

# Real-time CPU
sar -u 1 10

# Real-time memory
sar -r 1 10

# Real-time disk
sar -d 1 10

# Real-time network
sar -n DEV 1 10

# All metrics at once
sar -A 1 5
```

---

### Example 8 — `nmon` — Interactive All-in-One Monitor

```bash
# Install nmon
sudo apt install nmon    # Debian/Ubuntu
sudo dnf install nmon    # RHEL

# Launch
nmon
```

**`nmon` keyboard shortcuts:**

| Key | Toggles |
|-----|---------|
| `c` | **CPU** stats |
| `m` | **Memory** stats |
| `d` | **Disk** I/O |
| `n` | **Network** stats |
| `t` | **Top** processes |
| `q` | **Quit** |

---

### Example 9 — `watch` — Refresh Any Command Periodically

```bash
# Refresh free memory every 2 seconds
watch -n 2 free -h

# Watch disk usage
watch -n 5 'df -h | grep -v tmpfs'

# Watch top processes
watch -n 2 'ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -10'

# Watch failed services
watch -n 5 'systemctl --failed'

# Highlight changes between refreshes
watch -n 2 -d free -h
```

---

### Example 10 — Custom Real-Time Dashboard Script

```bash
#!/bin/bash
# Lightweight real-time dashboard

clear_and_header() {
  clear
  echo "╔══════════════════════════════════════════════════╗"
  echo "║     REAL-TIME SYSTEM METRICS — $(date '+%H:%M:%S')     ║"
  echo "╚══════════════════════════════════════════════════╝"
}

while true; do
  clear_and_header

  echo -e "\n── Load & CPU ───────────────────────────────────────"
  uptime
  top -bn1 | grep "Cpu(s)" | awk '{
    printf "CPU:  usr=%-6s sys=%-6s iow=%-6s idle=%s\n", $2, $4, $8, $12
  }'

  echo -e "\n── Memory ───────────────────────────────────────────"
  free -h | awk 'NR==2{printf "RAM:  total=%-8s used=%-8s avail=%s\n", $2, $3, $7}'
  free -h | awk 'NR==3{printf "Swap: total=%-8s used=%-8s free=%s\n", $2, $3, $4}'

  echo -e "\n── Disk Usage ───────────────────────────────────────"
  df -h | awk 'NR>1 && $1 !~ /tmpfs/ {printf "%-20s %s used of %s (%s)\n", $6, $3, $2, $5}'

  echo -e "\n── Top 5 Processes by CPU ───────────────────────────"
  ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -6

  echo -e "\n── Network ──────────────────────────────────────────"
  ip -s link show eth0 2>/dev/null | awk '/RX:/{getline; printf "RX: %s bytes\n", $1} /TX:/{getline; printf "TX: %s bytes\n", $1}'

  sleep 3
done
```

---

### Example 11 — `bpytop` / `btop` — Modern Beautiful Monitor

```bash
# Install btop (modern replacement for htop/top)
sudo apt install btop    # Ubuntu 22.04+
sudo dnf install btop    # RHEL 9+

# Or build from source / snap
sudo snap install btop

# Launch
btop
```

> `btop` is a modern, visually rich monitor with responsive layout, mouse support, themes, and detailed per-core graphs — the most visually impressive real-time monitor available.

---

### Example 12 — Prometheus + Node Exporter (Production Monitoring)

For production environments, use dedicated monitoring stacks:

```bash
# Install Node Exporter (exposes system metrics)
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz
tar -xzf node_exporter-*.tar.gz
sudo mv node_exporter*/node_exporter /usr/local/bin/

# Start Node Exporter
node_exporter &

# Metrics available at:
curl http://localhost:9100/metrics | grep "node_cpu\|node_memory\|node_disk" | head -20
```

---

## 🗂️ Monitoring Tools — Complete Comparison

| Tool | CPU | RAM | Disk | Net | Process | Historical | GUI-like |
|------|-----|-----|------|-----|---------|------------|---------|
| `top` | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | Basic |
| `htop` | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ Color |
| `glances` | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ Rich |
| `dstat` | ✅ | ✅ | ✅ | ✅ | Partial | ❌ | ❌ |
| `vmstat` | ✅ | ✅ | Partial | ❌ | Partial | ❌ | ❌ |
| `iostat` | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| `nmon` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `sar` | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| `btop` | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ Rich |

---

## 🔢 Performance Health Reference

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Load / CPU cores | < 0.7 | 0.7–1.0 | > 1.0 |
| CPU idle | > 50% | 20–50% | < 20% |
| I/O wait | < 5% | 5–20% | > 20% |
| Available RAM | > 30% | 15–30% | < 15% |
| Swap used | 0% | < 50% | > 50% |
| Disk `%util` | < 60% | 60–90% | > 90% |

---

## 🚀 Full Workflow — Set Up Real-Time Monitoring

```bash
# Step 1: Quick health check
uptime && free -h && df -h

# Step 2: Launch htop for interactive monitoring
htop

# Step 3: If investigating disk or network:
dstat -cdnm 1    # Combined view

# Step 4: For disk specifically
iostat -x 1

# Step 5: For per-process I/O
sudo iotop -o

# Step 6: For comprehensive dashboard
glances

# Step 7: For historical analysis
sar -u -r -d

# Step 8: Set up continuous logging
sar -o /var/log/sa/sa$(date +%d) 5 &    # Log every 5 seconds
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `htop`/`glances` not installed | `sudo apt install htop glances` |
| High load but low CPU | Check `wa` (I/O wait) — disk is the real bottleneck |
| Memory looks free but system is slow | Check `available` column and swap `si`/`so` in vmstat |
| First `vmstat`/`iostat` line misleading | Shows since-boot average — skip first line |
| `sar` shows no data | Enable: `sudo systemctl enable --now sysstat` |
| Monitoring tool itself using too much CPU | Use `sar` for low-overhead background collection |

---

## 🔑 Key Takeaways

- **`htop`** is the best interactive all-around monitor — color-coded, mouse-aware, and shows everything.
- **`glances`** gives the most complete single-screen view — CPU, RAM, disk, network, temps, and processes.
- **`dstat -cdnm 1`** combines all four subsystems — perfect for correlating CPU spikes with disk or network.
- **`vmstat 1`** is the fastest way to determine the bottleneck subsystem — look at `r`, `b`, `wa`, `si`/`so`.
- **High `wa` (I/O wait)** means the bottleneck is disk — not CPU, even if load average is high.
- **`sar`** is the only tool with **historical data** — essential for post-incident investigation.
- For production, use **Prometheus + Grafana** or **Datadog** for persistent, alertable metrics dashboards.

---

## 📚 Related Concepts


- [glances Documentation](https://nicolargo.github.io/glances/)
- [btop GitHub](https://github.com/aristocratos/btop)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)

</details>
