# Challenge 42: Check Network Connectivity in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

**Network connectivity testing** is one of the most fundamental Linux administration skills. Before debugging complex network issues, you need to verify the basics — can the system reach a host at all? Is DNS working? Is a specific port open?

These simple connectivity checks help you quickly isolate whether a problem is:
- A local network configuration issue
- A DNS resolution failure
- A remote host or service being unreachable
- A firewall blocking traffic

---

## 🧩 Task

Test network connectivity to a remote host.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Test basic connectivity (ICMP ping)
ping google.com

# Limit to 4 packets
ping -c 4 google.com

# Test if a specific port is open
nc -zv google.com 80
```

### How it works

| Command | Description |
|---------|-------------|
| `ping hostname` | Send ICMP echo requests — tests basic connectivity |
| `-c N` | Send exactly N packets then stop |
| `nc -zv host port` | Test if a specific **TCP port** is reachable |

---

## 🔢 Understanding `ping` Output

```bash
ping -c 4 google.com
```

```
PING google.com (142.250.80.46): 56 data bytes
64 bytes from 142.250.80.46: icmp_seq=1 ttl=116 time=12.4 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=116 time=11.8 ms
64 bytes from 142.250.80.46: icmp_seq=3 ttl=116 time=13.1 ms
64 bytes from 142.250.80.46: icmp_seq=4 ttl=116 time=12.6 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 11.8/12.5/13.1/0.5 ms
```

| Field | Description |
|-------|-------------|
| `64 bytes` | Response received from host |
| `icmp_seq` | Sequence number — gaps indicate packet loss |
| `ttl` | Time To Live — decremented at each router |
| `time` | **Round-trip latency in milliseconds** |
| `0% packet loss` | All packets returned — good connectivity |
| `min/avg/max/mdev` | Latency statistics |

---

## 📖 Extended Examples

### Example 1 — Basic Ping

```bash
# Ping continuously (press Ctrl+C to stop)
ping google.com

# Send exactly 4 packets
ping -c 4 google.com
```

---

### Example 2 — Ping by IP Address (Bypass DNS)

```bash
# Ping using IP directly
ping 8.8.8.8

# If ping by hostname fails but IP works → DNS problem
ping google.com    # Fails
ping 142.250.80.46 # Works → DNS is the issue
```

---

### Example 3 — Test with Specific Packet Count and Interval

```bash
# Send 10 packets, one every 0.5 seconds
ping -c 10 -i 0.5 google.com

# Flood ping (use carefully — sends as fast as possible)
sudo ping -f -c 100 192.168.1.1

# Ping with larger packet size (test MTU)
ping -c 4 -s 1400 google.com
```

---

### Example 4 — Check Connectivity to Local Network

```bash
# Ping the default gateway (router)
ping $(ip route | awk '/default/ {print $3}')

# Ping another host on the same network
ping 192.168.1.100

# Ping localhost (test local stack)
ping 127.0.0.1
ping localhost
```

---

### Example 5 — Test a Specific Port with `nc` (netcat)

```bash
# Test if HTTP port 80 is open
nc -zv google.com 80

# Test HTTPS port 443
nc -zv google.com 443

# Test SSH port 22
nc -zv 192.168.1.5 22

# Test with timeout (3 seconds)
nc -zvw 3 google.com 443
```

```bash
# nc output when port is OPEN:
Connection to google.com 443 port [tcp/https] succeeded!

# nc output when port is CLOSED:
nc: connect to google.com port 81: Connection refused
```

---

### Example 6 — Check Connectivity with `curl`

```bash
# Test HTTP connectivity
curl -I http://google.com

# Test with timing breakdown
curl -o /dev/null -s -w "DNS: %{time_namelookup}s | Connect: %{time_connect}s | Total: %{time_total}s\n" https://google.com
```

```bash
# Timing output:
DNS: 0.008s | Connect: 0.025s | Total: 0.125s
```

---

### Example 7 — Trace the Network Path with `traceroute`

```bash
# Trace route to a host
traceroute google.com

# Numeric output (no DNS)
traceroute -n google.com

# Use ICMP instead of UDP
traceroute -I google.com
```

```bash
# traceroute output:
 1  192.168.1.1    1.2 ms   1.1 ms   1.0 ms    ← Local gateway
 2  10.10.0.1      5.4 ms   5.3 ms   5.5 ms    ← ISP router
 3  203.0.113.1   10.2 ms  10.1 ms  10.3 ms    ← Upstream
 4  * * *                                       ← ICMP blocked
 5  142.250.80.46 12.4 ms  12.3 ms  12.5 ms    ← Google
```

---

### Example 8 — Check DNS Resolution

```bash
# Test DNS resolution
nslookup google.com

# Detailed DNS info
dig google.com

# Short answer only
dig +short google.com

# Test against a specific DNS server
dig @8.8.8.8 google.com
```

---

### Example 9 — Check Open Ports with `ss`

```bash
# Show all listening ports
ss -tlnp

# Show all established connections
ss -tnp state established

# Check if a specific port is listening
ss -tlnp | grep ':80'
ss -tlnp | grep ':22'
```

---

### Example 10 — Quick Connectivity Script

```bash
#!/bin/bash

HOSTS=("google.com" "8.8.8.8" "192.168.1.1")
TIMEOUT=3

echo "Network Connectivity Check — $(date)"
echo "──────────────────────────────────────"

for HOST in "${HOSTS[@]}"; do
  if ping -c 1 -W $TIMEOUT "$HOST" &>/dev/null; then
    RTT=$(ping -c 1 -W $TIMEOUT "$HOST" | grep 'time=' | awk -F'time=' '{print $2}')
    echo "✅ $HOST — reachable (${RTT})"
  else
    echo "❌ $HOST — unreachable"
  fi
done
```

```bash
# Output:
Network Connectivity Check — Thu Mar 21 10:35:22 2025
──────────────────────────────────────
✅ google.com — reachable (12.4 ms)
✅ 8.8.8.8 — reachable (8.2 ms)
✅ 192.168.1.1 — reachable (1.1 ms)
```

---

## 🗂️ Connectivity Testing Tools — Quick Reference

| Tool | Tests | Use Case |
|------|-------|----------|
| `ping` | ICMP reachability | Basic host reachability |
| `ping IP` | ICMP bypassing DNS | Isolate DNS vs network issues |
| `nc -zv host port` | TCP port reachability | Test if a service port is open |
| `curl -I url` | HTTP/HTTPS response | Test web service connectivity |
| `traceroute` | Path to host | Find where packets fail |
| `dig / nslookup` | DNS resolution | Test name resolution |
| `ss -tlnp` | Local open ports | Check what is listening |

---

## 🔢 Connectivity Issue Diagnosis Path

```
Can't reach a service?
      │
      ▼
ping IP address directly
      │
  ┌───┴───┐
  │ Fails │ → Local network/gateway issue
  │       │   → ping 192.168.1.1 (gateway)
  └───┬───┘
      │ Works
      ▼
ping hostname
      │
  ┌───┴───┐
  │ Fails │ → DNS issue
  │       │   → dig @8.8.8.8 hostname
  └───┬───┘
      │ Works
      ▼
nc -zv host port
      │
  ┌───┴───┐
  │ Fails │ → Firewall or service not running
  │       │   → Check firewall rules
  └───┬───┘
      │ Works
      ▼
Service is reachable ✅
```

---

## 🚀 Full Workflow — Diagnose a Connectivity Problem

```bash
# Step 1: Test local network stack
ping -c 2 127.0.0.1

# Step 2: Test gateway (router)
ping -c 2 $(ip route | awk '/default/ {print $3}')

# Step 3: Test external IP (bypass DNS)
ping -c 2 8.8.8.8

# Step 4: Test DNS resolution
ping -c 2 google.com
dig +short google.com

# Step 5: Test specific service port
nc -zvw 3 google.com 443

# Step 6: Trace the network path
traceroute -n google.com

# Step 7: Check local listening services
ss -tlnp
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `ping` succeeds but app can't connect | Test the specific port with `nc -zv host port` |
| `ping hostname` fails but `ping IP` works | DNS problem — check `/etc/resolv.conf` and `dig` |
| `ping` blocked by firewall | Use `nc -zv` or `curl` for TCP-based testing |
| Linux `ping` runs forever | Always use `-c N` to limit packets |
| Port test with `nc` always fails | Ensure `ncat` or `netcat` is installed: `sudo apt install netcat` |

---

## 🔑 Key Takeaways

- `ping -c 4 hostname` tests **basic ICMP reachability** — always the first connectivity check.
- If `ping IP` works but `ping hostname` fails — the problem is **DNS**, not the network.
- `nc -zv host port` tests if a **specific TCP port** is reachable — essential for service testing.
- Use `traceroute` to find **where** in the network path packets are being dropped.
- `dig @8.8.8.8 hostname` tests DNS resolution using a **known public resolver** — bypasses local DNS.
- Always test in order: **localhost → gateway → external IP → hostname → service port**.
- `0% packet loss` in ping output confirms reliable connectivity; any loss indicates a problem.

---

## 📚 Related Concepts

- [ping Manual](https://linux.die.net/man/8/ping)
- [GNU netcat Manual](https://linux.die.net/man/1/nc)

</details>
