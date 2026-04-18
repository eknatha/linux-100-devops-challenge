# Challenge 43: Trace Network Route in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Route tracing** shows the path that network packets take from your system to a destination host — hop by hop, through every router in between. It is one of the most powerful network diagnostic tools, revealing exactly where connectivity breaks down, where latency is introduced, and how traffic is routed across networks.

Two primary tools:
- **`traceroute`** — the classic hop-by-hop path tracer
- **`mtr`** — combines traceroute and ping for continuous per-hop monitoring

---

## 🧩 Task

Trace the network route to a remote host to see every hop along the path.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Classic traceroute
traceroute google.com

# Modern continuous tracer (recommended)
mtr google.com

# Numeric output (no DNS lookups — faster)
traceroute -n google.com
```

### How it works

| Command | Description |
|---------|-------------|
| `traceroute hostname` | Trace the network path — shows each router hop |
| `mtr hostname` | **My Traceroute** — combines ping + traceroute in real-time |
| `-n` | No DNS resolution — show IP addresses instead of hostnames |

---

## 🔢 Understanding `traceroute` Output

```bash
traceroute google.com
```

```
traceroute to google.com (142.250.80.46), 30 hops max, 60 byte packets
 1  192.168.1.1        1.241 ms  1.102 ms  1.056 ms   ← Local gateway
 2  10.10.0.1          5.512 ms  5.421 ms  5.389 ms   ← ISP router
 3  203.0.113.1       10.234 ms 10.112 ms 10.301 ms   ← Upstream
 4  * * *                                              ← ICMP blocked
 5  72.14.198.100     11.105 ms 10.987 ms 11.012 ms   ← Google edge
 6  142.250.80.46     12.412 ms 12.301 ms 12.288 ms   ← Destination
```

| Column | Description |
|--------|-------------|
| Hop number | Order in the network path |
| Hostname/IP | The router at this hop |
| 3 time values | **Round-trip latency** for 3 probe packets (ms) |
| `* * *` | Router **does not respond** to ICMP — not necessarily a problem |

> 💡 `* * *` at a middle hop does NOT mean the path is broken — many routers block ICMP but still forward traffic. If subsequent hops respond, the path is fine.

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
4. ???                      100.0%  10   --     --    --    --    --
5. 72.14.198.100             0.0%   10  11.1   11.3  10.9  12.0   0.3
6. 142.250.80.46             0.0%   10  12.4   12.6  11.8  13.1   0.4
```

| Column | Description |
|--------|-------------|
| `Host` | Router IP or hostname at this hop |
| `Loss%` | **Packet loss** at this hop |
| `Snt` | Packets sent |
| `Last` | Most recent RTT (ms) |
| `Avg` | Average RTT |
| `Best` | Minimum RTT |
| `Wrst` | Maximum RTT |
| `StDev` | **Jitter** — variation in latency |

---

## 📖 Extended Examples

### Example 1 — Basic Traceroute

```bash
traceroute google.com
```

---

### Example 2 — Numeric Output (No DNS — Faster)

```bash
traceroute -n google.com
```

```bash
# Output shows only IP addresses — no DNS resolution delay:
 1  192.168.1.1    1.2 ms  1.1 ms  1.1 ms
 2  10.10.0.1      5.5 ms  5.4 ms  5.4 ms
 3  203.0.113.1   10.2 ms 10.1 ms 10.3 ms
```

---

### Example 3 — Use ICMP Instead of UDP (Like Windows `tracert`)

```bash
# Linux traceroute uses UDP by default — ICMP is often less filtered
sudo traceroute -I google.com

# TCP traceroute on port 80 (bypasses most firewall ICMP blocks)
sudo traceroute -T -p 80 google.com
```

---

### Example 4 — Set Maximum Hops

```bash
# Limit to 15 hops (default is 30)
traceroute -m 15 google.com

# Show only first 5 hops (local path)
traceroute -m 5 google.com
```

---

### Example 5 — `mtr` Interactive Mode

```bash
# Launch interactive mtr
mtr google.com
```

**`mtr` keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `d` | Toggle **DNS** resolution on/off |
| `n` | Show **IP addresses** only |
| `p` | **Pause** / resume |
| `r` | **Reset** statistics |
| `q` | **Quit** |

---

### Example 6 — `mtr` Report Mode (Non-Interactive)

```bash
# Generate a report with 100 packets per hop
mtr --report --report-cycles 100 google.com
```

```bash
Start: 2025-03-21T10:35:22+0000
HOST: myserver                    Loss%  Snt  Last  Avg  Best  Wrst StDev
  1.|-- 192.168.1.1               0.0%  100   1.2  1.3   1.0   1.8   0.2
  2.|-- 10.10.0.1                 0.0%  100   5.4  5.6   5.1   6.3   0.3
  3.|-- 203.0.113.1               0.0%  100  10.2 10.5   9.8  11.4   0.4
  4.|-- ???                      100.0%  100   --   --    --    --    --
  5.|-- 72.14.198.100             0.0%  100  11.1 11.3  10.9  12.0   0.3
  6.|-- 142.250.80.46             0.0%  100  12.4 12.6  11.8  13.1   0.4
```

---

### Example 7 — `mtr` JSON Output (For Scripting)

```bash
# Export as JSON
mtr --json --report --report-cycles 10 google.com

# Export as CSV
mtr --csv --report --report-cycles 10 google.com
```

---

### Example 8 — Identify Where Latency Increases

```bash
mtr --report google.com
```

Looking at the Avg column:

```
Hop 1:  1.3ms   ← Local gateway (fast)
Hop 2:  5.6ms   ← ISP router (+4ms — normal)
Hop 3:  10.5ms  ← Upstream (+5ms — normal)
Hop 4:  ???     ← ICMP blocked (skip)
Hop 5:  11.3ms  ← Google edge (+1ms — fine)
Hop 6:  12.6ms  ← Destination (+1ms — fine)
```

> 💡 A sudden large increase in latency at one hop — for example, jumping from 10ms to 150ms — identifies the **bottleneck link** in the network.

---

### Example 9 — Diagnose Packet Loss

```bash
mtr --report --report-cycles 50 google.com
```

**Interpreting `Loss%`:**

```
Hop 3: Loss 50.0%  → Hop 4: Loss 0.0%
# Loss at hop 3 but NOT at hop 4 = hop 3 is rate-limiting ICMP (not a real problem)

Hop 3: Loss 50.0%  → Hop 4: Loss 50.0%
# Loss persists from hop 3 onwards = REAL packet loss at hop 3's link
```

> 💡 **Loss that starts at a hop and continues to the destination** is real. Loss at a middle hop that clears at the next hop is just ICMP rate limiting.

---

### Example 10 — Install and Use on Different Systems

```bash
# Install traceroute
sudo apt install traceroute    # Debian/Ubuntu
sudo dnf install traceroute    # RHEL/CentOS

# Install mtr
sudo apt install mtr           # Debian/Ubuntu
sudo dnf install mtr           # RHEL/CentOS

# macOS equivalent
traceroute google.com          # Built-in
brew install mtr               # via Homebrew
```

---

## 🗂️ Traceroute Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `traceroute hostname` | Basic UDP traceroute |
| `traceroute -n hostname` | No DNS resolution (faster) |
| `traceroute -I hostname` | ICMP mode (requires root) |
| `traceroute -T -p 80 hostname` | TCP mode on port 80 (requires root) |
| `traceroute -m N hostname` | Set max hops to N |
| `mtr hostname` | Interactive real-time tracer |
| `mtr --report hostname` | Non-interactive report mode |
| `mtr -n hostname` | Numeric only (no DNS) |
| `mtr --report-cycles N hostname` | Set number of packets per hop |
| `mtr --json hostname` | JSON output |

---

## 🔄 `traceroute` vs `mtr` — When to Use Each

| Feature | `traceroute` | `mtr` |
|---------|-------------|-------|
| One-time snapshot | ✅ | ✅ |
| Continuous monitoring | ❌ | ✅ |
| Per-hop packet loss | ❌ | ✅ |
| Jitter (StDev) | ❌ | ✅ |
| Interactive display | ❌ | ✅ |
| JSON/CSV export | ❌ | ✅ |
| Requires install | Sometimes | Usually |
| Best for | Quick path check | Detailed diagnostics |

---

## 🚀 Full Workflow — Diagnose a Network Path Issue

```bash
# Step 1: Basic connectivity check
ping -c 4 google.com

# Step 2: Trace the path
traceroute -n google.com

# Step 3: Run mtr for detailed per-hop stats
mtr --report --report-cycles 50 google.com

# Step 4: Look for:
# - Where does latency increase significantly?
# - Where does packet loss start and persist?
# - Are * * * hops followed by normal hops? (ICMP block — not a real problem)

# Step 5: If ICMP is blocked, try TCP traceroute
sudo traceroute -T -p 443 google.com

# Step 6: Save report for sharing with ISP or network team
mtr --report --report-cycles 100 google.com > /tmp/mtr_report_$(date +%Y%m%d).txt
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `* * *` means broken path | Not necessarily — many routers block ICMP. Check if subsequent hops respond |
| Loss at one middle hop = real problem | Only real if loss **persists** to subsequent hops |
| traceroute uses UDP (may be filtered) | Try `-I` (ICMP) or `-T -p 80` (TCP) modes |
| mtr not installed | `sudo apt install mtr` or `sudo dnf install mtr` |
| Traceroute incomplete (stops mid-path) | Destination may block ICMP — try TCP mode |
| First hop shows high latency | Check local network — may indicate WiFi or gateway issue |

---

## 🔑 Key Takeaways

- `traceroute` shows the **hop-by-hop network path** — each line is a router in the path.
- `* * *` at middle hops means **ICMP is blocked** at that router — not necessarily a real problem.
- **`mtr`** is more powerful than `traceroute` — it shows continuous per-hop packet loss, latency, and jitter.
- **Packet loss that persists** from a hop to the destination is real; loss that clears at the next hop is just ICMP rate limiting.
- A **sudden large latency increase** at one hop identifies the bottleneck link.
- Use `mtr --report --report-cycles 100` to generate a **shareable report** for network team analysis.
- Use **`-n`** flag to skip DNS resolution — results come faster and aren't confused by hostname lookups.

---

## 📚 Related Concepts

- [mtr Manual](https://linux.die.net/man/8/mtr)
- [traceroute Manual](https://linux.die.net/man/8/traceroute)

</details>
