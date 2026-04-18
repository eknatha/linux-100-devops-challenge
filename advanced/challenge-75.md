# Challenge 75: Debug Network Latency in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**Network latency** is the time it takes for a packet to travel from your system to a destination and back — measured in milliseconds (ms). High latency causes slow API responses, database timeouts, choppy video calls, and degraded user experience.

Diagnosing network latency requires identifying **where** in the network path the delay occurs — whether it is your local machine, a gateway, an intermediate router, or the destination server. The combination of `ping` (end-to-end measurement) and `mtr` (per-hop analysis) gives you a complete picture.

---

## 🧩 Task

Identify where network delay is occurring between your system and a destination host.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Basic end-to-end latency check
ping google.com

# Per-hop latency analysis (combines ping + traceroute)
mtr google.com
```

### How it works

| Command | Description |
|---------|-------------|
| `ping` | Sends ICMP echo requests and measures **round-trip time (RTT)** |
| `mtr` | **My Traceroute** — traces each hop and measures latency per hop continuously |

---

## 🔢 Understanding `ping` Output

```bash
ping google.com
```

```
PING google.com (142.250.80.46): 56 data bytes
64 bytes from 142.250.80.46: icmp_seq=1 ttl=116 time=12.4 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=116 time=11.8 ms
64 bytes from 142.250.80.46: icmp_seq=3 ttl=116 time=13.1 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 11.8/12.4/13.1/0.5 ms
```

| Field | Description |
|-------|-------------|
| `icmp_seq` | Sequence number — gaps indicate packet loss |
| `ttl` | Time To Live — hops remaining (reduced by 1 at each router) |
| `time` | **Round-trip time in milliseconds** — the key latency metric |
| `0% packet loss` | All packets returned — 0% is healthy |
| `min/avg/max/mdev` | Minimum, average, maximum, and mean deviation of RTT |
| `mdev` | **Jitter** — variation in latency (high mdev = unstable connection) |

---

## 🔢 Understanding `mtr` Output

```bash
mtr google.com
```

```
Host                        Loss%  Snt  Last   Avg  Best  Wrst  StDev
1. 192.168.1.1 (gateway)    0.0%   10   1.2    1.3   1.0   1.8   0.2
2. 10.10.0.1 (ISP router)   0.0%   10   5.4    5.6   5.1   6.3   0.3
3. 203.0.113.1               0.0%   10  10.2   10.5   9.8  11.4   0.4
4. 72.14.198.100             0.0%   10  11.1   11.3  10.9  12.0   0.3
5. 142.250.80.46 (google)   0.0%   10  12.4   12.6  11.8  13.1   0.4
```

| Column | Description |
|--------|-------------|
| `Host` | IP address or hostname of each hop |
| `Loss%` | **Packet loss at this hop** — key indicator of network problems |
| `Snt` | Total packets sent |
| `Last` | RTT of the most recent packet (ms) |
| `Avg` | Average RTT across all samples (ms) |
| `Best` | Minimum RTT recorded (ms) |
| `Wrst` | Maximum RTT recorded (ms) |
| `StDev` | **Standard deviation — jitter at this hop** |

---

## 📖 Extended Examples

### Example 1 — Basic Ping

```bash
ping google.com
```

```bash
# Linux — runs until Ctrl+C
# macOS — runs 5 packets by default
```

---

### Example 2 — Ping with Limited Count

```bash
# Send exactly 10 packets then stop
ping -c 10 google.com
```

```bash
--- google.com ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9012ms
rtt min/avg/max/mdev = 11.8/12.4/13.1/0.5 ms
```

---

### Example 3 — Ping with Custom Interval and Packet Size

```bash
# Ping every 0.2 seconds (flood-like — use carefully)
ping -i 0.2 -c 20 google.com

# Large packet size (tests MTU and fragmentation)
ping -s 1400 -c 5 google.com

# Minimum packet size for pure latency measurement
ping -s 1 -c 5 google.com
```

---

### Example 4 — Detect Packet Loss

```bash
ping -c 50 google.com | tail -2
```

```bash
# Healthy — no loss:
50 packets transmitted, 50 received, 0% packet loss

# Problem — packet loss detected:
50 packets transmitted, 43 received, 14% packet loss
```

> 💡 Even 1–2% packet loss on a connection causes **significant TCP retransmissions** and noticeable performance degradation. 5%+ is severe.

---

### Example 5 — Run `mtr` Interactively

```bash
mtr google.com
```

**Key `mtr` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `d` | Toggle DNS resolution on/off |
| `n` | Show IP addresses instead of hostnames |
| `p` | Pause/resume |
| `r` | Reset statistics |
| `q` | Quit |
| `?` | Help |

---

### Example 6 — `mtr` Report Mode (Non-Interactive)

```bash
# Generate a static report (100 packets per hop)
mtr --report --report-cycles 100 google.com
```

```bash
Host                        Loss%  Snt  Last  Avg  Best  Wrst  StDev
1. 192.168.1.1               0.0%  100   1.2  1.3   1.0   1.8   0.2
2. 10.10.0.1                 0.0%  100   5.4  5.6   5.1   6.3   0.3
3. 203.0.113.1               0.0%  100  10.2  10.5   9.8  11.4   0.4
4. ???                      100.0%  100   --   --    --    --    --
5. 72.14.198.100             0.0%  100  11.1  11.3  10.9  12.0   0.3
6. 142.250.80.46             0.0%  100  12.4  12.6  11.8  13.1   0.4
```

> 💡 Hop 4 shows **100% loss** — this is almost always a router configured to **block ICMP** (not a real network problem). If the next hops show 0% loss, the path is healthy.

---

### Example 7 — `mtr` in JSON or CSV Format (For Scripting)

```bash
# JSON output
mtr --json --report --report-cycles 10 google.com

# CSV output
mtr --csv --report --report-cycles 10 google.com
```

---

### Example 8 — `traceroute` Alternative

```bash
# Classic traceroute (UDP by default on Linux)
traceroute google.com

# ICMP traceroute (like Windows tracert)
traceroute -I google.com

# TCP traceroute on port 80 (bypasses ICMP blocks)
sudo traceroute -T -p 80 google.com

# Faster — no DNS resolution
traceroute -n google.com
```

```bash
traceroute to google.com (142.250.80.46), 30 hops max
 1  192.168.1.1     1.241 ms  1.102 ms  1.056 ms
 2  10.10.0.1       5.512 ms  5.421 ms  5.389 ms
 3  203.0.113.1    10.234 ms  10.112 ms  10.301 ms
 4  * * *                                           ← ICMP blocked
 5  72.14.198.100  11.105 ms  10.987 ms  11.012 ms
 6  142.250.80.46  12.412 ms  12.301 ms  12.288 ms
```

---

### Example 9 — Test DNS Latency Separately

```bash
# Measure DNS resolution time
time nslookup google.com

# Or with dig
dig google.com | grep "Query time"

# Test against a specific DNS server
dig @8.8.8.8 google.com | grep "Query time"
```

```bash
# dig output:
;; Query time: 8 msec
;; SERVER: 192.168.1.1#53
```

> 💡 High DNS latency (> 50ms) adds to every connection — if DNS is slow but `ping` is fast, the culprit is your DNS resolver, not the network path.

---

### Example 10 — Test TCP Connection Latency

```bash
# Test TCP latency to a port (not just ICMP)
# Useful when ICMP is blocked

# Using curl (time to first byte)
curl -o /dev/null -s -w "DNS: %{time_namelookup}s | Connect: %{time_connect}s | TTFB: %{time_starttransfer}s | Total: %{time_total}s\n" https://google.com
```

```bash
DNS: 0.008s | Connect: 0.025s | TTFB: 0.124s | Total: 0.125s
```

| `curl` Timing Field | Description |
|--------------------|-------------|
| `time_namelookup` | Time to resolve DNS |
| `time_connect` | Time to establish TCP connection |
| `time_starttransfer` | Time to first byte (TTFB) |
| `time_total` | Total request time |

---

### Example 11 — Check for Local Network Congestion

```bash
# Check interface statistics (errors, drops, collisions)
ip -s link show eth0
```

```bash
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    RX:  bytes    packets  errors  dropped  overrun  mcast
    1048576000  1000000       0        5         0  12000
    TX:  bytes    packets  errors  dropped  carrier  collisions
    524288000    500000       0        0         0       0
```

```bash
# Watch interface stats live
watch -n 1 'ip -s link show eth0'

# Check for packet drops in real time
netstat -i
```

> ⚠️ Non-zero **errors** or **dropped** packets on a local interface indicate hardware issues, driver problems, or network congestion at the NIC level — not a remote network problem.

---

## 🗂️ Interpreting `mtr` Results — Pattern Guide

| `mtr` Pattern | Meaning | Action |
|--------------|---------|--------|
| Loss% at one hop, 0% at subsequent hops | Router blocking ICMP — **not a real problem** | Ignore that hop |
| Loss% starts at one hop and continues | Real packet loss at that router or link | Investigate that hop |
| Latency jumps sharply at one hop | Bottleneck or congested link at that point | Contact ISP or check routing |
| All hops OK but destination unreachable | Firewall at the destination blocking traffic | Check destination firewall |
| High `StDev` (jitter) at one hop | Unstable link at that hop | Check physical connection or ISP |
| `???` for a hop | Router is not responding to ICMP — not a real loss | Continue checking subsequent hops |

---

## 🛠️ Network Latency Tools — Comparison

| Tool | Type | Best For |
|------|------|----------|
| `ping` | RTT measurement | Quick end-to-end latency and packet loss check |
| `mtr` | Per-hop analysis | Pinpointing where in the path latency is introduced |
| `traceroute` | Path mapping | One-time hop-by-hop path discovery |
| `dig` / `nslookup` | DNS latency | Isolating DNS resolution delays |
| `curl -w` | HTTP timing | Full HTTP request timing breakdown |
| `ss` / `netstat` | Connection state | Checking local socket states and retransmissions |
| `iperf3` | Bandwidth test | Measuring actual throughput between two hosts |
| `tcpdump` | Packet capture | Deep-dive packet-level latency analysis |

---

## 🔢 Latency Reference

| Connection Type | Expected RTT | Warning | Critical |
|----------------|-------------|---------|----------|
| Same data center | < 1 ms | > 5 ms | > 20 ms |
| Same country | 5–30 ms | > 50 ms | > 100 ms |
| Cross-continent | 80–200 ms | > 250 ms | > 400 ms |
| Satellite | 500–700 ms | — | Expected |
| DNS resolution | < 10 ms | > 50 ms | > 200 ms |

---

## 🚀 Full Workflow — Diagnose a Network Latency Problem

```bash
# Step 1: Quick end-to-end latency check
ping -c 20 google.com

# Step 2: Check for packet loss
ping -c 100 google.com | tail -3

# Step 3: Find where in the path the latency is introduced
mtr --report --report-cycles 50 google.com

# Step 4: Check if DNS is slow
dig google.com | grep "Query time"

# Step 5: If ICMP is blocked, test TCP latency instead
curl -o /dev/null -s -w "Connect: %{time_connect}s TTFB: %{time_starttransfer}s\n" https://google.com

# Step 6: Check local interface for errors/drops
ip -s link show eth0

# Step 7: Test bandwidth to rule out saturation
iperf3 -c iperf.he.net      # Test against a public iperf server

# Step 8: Set up ongoing monitoring
# Add to cron — check every 5 minutes
*/5 * * * * /usr/local/bin/latency_monitor.sh >> /var/log/network_latency.log 2>&1
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| 100% loss at a mid-path hop | Check if **subsequent hops are fine** — ICMP is likely blocked at that router |
| `ping` succeeds but app is slow | Test TCP latency with `curl -w` — app-level timeouts differ from ICMP |
| Low average RTT but high `mdev` | **Jitter** is the problem — variable latency wrecks real-time applications |
| `mtr` not installed | Install: `sudo apt install mtr` or `sudo dnf install mtr` |
| `ping` blocked by destination firewall | Use `curl`, `traceroute -T`, or `nmap` to test TCP connectivity instead |
| DNS added to RTT measurement | Use `ping -n` (numeric) to avoid DNS lookups skewing the first packet RTT |

---

## 🔑 Key Takeaways

- `ping` measures **end-to-end round-trip time** and packet loss — always the first step.
- `mtr` combines traceroute and ping — it shows **per-hop latency and loss** to pinpoint exactly where problems occur.
- **100% loss at a single middle hop** with 0% loss afterwards is **not a problem** — it just means that router blocks ICMP.
- Real latency problems show as **increasing latency and loss that persists from a hop onwards**.
- **`mdev` (jitter)** is as important as average latency — high jitter degrades real-time applications even if average RTT is acceptable.
- If ICMP is blocked, use `curl -w`, `traceroute -T`, or `nmap` for TCP-based latency testing.
- Separate **DNS latency** from **network latency** — they are different problems with different fixes.

---

## 📚 Related Concepts

- [iperf3](https://iperf.fr/) — bandwidth measurement between two hosts
- [tcpdump](https://www.tcpdump.org/) — packet capture for detailed network analysis
- [Wireshark](https://www.wireshark.org/) — GUI packet analyzer
- [nmap](https://nmap.org/) — network discovery and TCP-level connectivity testing


</details>
