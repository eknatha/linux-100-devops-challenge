# Challenge 76: Capture Network Traffic with tcpdump

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`tcpdump`** is the most powerful and widely used command-line packet analyzer on Linux. It captures raw network traffic at the interface level, allowing you to inspect every packet passing through — from TCP handshakes to DNS queries to HTTP requests.

It is an essential tool for:
- Debugging application-level network issues
- Verifying firewall rules are working correctly
- Diagnosing slow connections at the packet level
- Investigating security incidents
- Capturing traffic for analysis in Wireshark

---

## 🧩 Task

Capture packets on a network interface to inspect live network traffic.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
sudo tcpdump -i eth0
```

### How it works

| Part | Description |
|------|-------------|
| `sudo` | Root privileges required — only root can put an interface in promiscuous mode |
| `tcpdump` | Packet capture tool — reads directly from the network interface |
| `-i eth0` | **Interface** to capture on — `eth0` is the first Ethernet interface |

---

## 🔢 Understanding `tcpdump` Output

```bash
sudo tcpdump -i eth0
```

```
10:35:22.123456 IP 192.168.1.5.54312 > 142.250.80.46.443: Flags [S], seq 1234567890, win 65535, length 0
10:35:22.135821 IP 142.250.80.46.443 > 192.168.1.5.54312: Flags [S.], seq 987654321, ack 1234567891, win 65535, length 0
10:35:22.135900 IP 192.168.1.5.54312 > 142.250.80.46.443: Flags [.], ack 1, win 65535, length 0
```

| Field | Example | Description |
|-------|---------|-------------|
| Timestamp | `10:35:22.123456` | Time packet was captured (microseconds) |
| Protocol | `IP` | Layer 3 protocol (IP, ARP, IPv6) |
| Source | `192.168.1.5.54312` | Source IP + port |
| Direction | `>` | Packet direction |
| Destination | `142.250.80.46.443` | Destination IP + port |
| Flags | `[S]` | TCP flags — see table below |
| Sequence | `seq 1234567890` | TCP sequence number |
| Length | `length 0` | Payload data length in bytes |

### TCP Flag Reference

| Flag | Code | Meaning |
|------|------|---------|
| `[S]` | SYN | Connection initiation |
| `[S.]` | SYN-ACK | Connection acknowledged |
| `[.]` | ACK | Acknowledgment |
| `[P.]` | PSH-ACK | Data push + acknowledgment |
| `[F.]` | FIN-ACK | Connection close |
| `[R]` | RST | Connection reset (abort) |
| `[R.]` | RST-ACK | Reset with acknowledgment |

> A healthy TCP handshake: `[S]` → `[S.]` → `[.]`
> A connection reset: `[R]` or `[R.]` — investigate why

---

## 📖 Extended Examples

### Example 1 — Capture on a Specific Interface

```bash
# Capture on eth0
sudo tcpdump -i eth0

# Capture on all interfaces
sudo tcpdump -i any

# List available interfaces
sudo tcpdump -D
```

```bash
# tcpdump -D output:
1.eth0 [Up, Running, Connected]
2.lo   [Up, Running, Loopback]
3.wlan0 [Up, Running, Connected]
```

---

### Example 2 — Show Human-Readable Output (No DNS Lookups)

```bash
# -n: no DNS resolution (faster, no false hostname mappings)
# -nn: no DNS AND no port name resolution (shows port numbers instead of 'https')
sudo tcpdump -i eth0 -nn
```

```bash
10:35:22.123456 IP 192.168.1.5.54312 > 142.250.80.46.443: Flags [S]
```

> 💡 Always use `-nn` in production — DNS lookups slow down capture and can alter timing.

---

### Example 3 — Limit Packet Count

```bash
# Capture exactly 50 packets then stop
sudo tcpdump -i eth0 -c 50

# Capture 100 packets with no DNS resolution
sudo tcpdump -i eth0 -nn -c 100
```

---

### Example 4 — Filter by Host

```bash
# Capture all traffic to/from a specific IP
sudo tcpdump -i eth0 host 192.168.1.100

# Only traffic FROM a host
sudo tcpdump -i eth0 src host 192.168.1.100

# Only traffic TO a host
sudo tcpdump -i eth0 dst host 192.168.1.100

# Capture traffic to/from a domain name
sudo tcpdump -i eth0 host google.com
```

---

### Example 5 — Filter by Port

```bash
# Capture only HTTP traffic (port 80)
sudo tcpdump -i eth0 port 80

# Capture only HTTPS traffic (port 443)
sudo tcpdump -i eth0 port 443

# Capture SSH traffic (port 22)
sudo tcpdump -i eth0 port 22

# Capture a range of ports
sudo tcpdump -i eth0 portrange 8000-8100

# Capture DNS queries (UDP port 53)
sudo tcpdump -i eth0 udp port 53
```

---

### Example 6 — Filter by Protocol

```bash
# Capture only TCP traffic
sudo tcpdump -i eth0 tcp

# Capture only UDP traffic
sudo tcpdump -i eth0 udp

# Capture only ICMP (ping) traffic
sudo tcpdump -i eth0 icmp

# Capture only ARP traffic
sudo tcpdump -i eth0 arp
```

---

### Example 7 — Combine Filters with Boolean Operators

```bash
# HTTP or HTTPS traffic
sudo tcpdump -i eth0 'port 80 or port 443'

# Traffic from a specific host on port 443
sudo tcpdump -i eth0 'host 192.168.1.5 and port 443'

# All traffic EXCEPT SSH
sudo tcpdump -i eth0 'not port 22'

# HTTP traffic from a specific subnet
sudo tcpdump -i eth0 'src net 192.168.1.0/24 and port 80'
```

---

### Example 8 — Show Packet Contents (Payload)

```bash
# Show ASCII payload (readable text)
sudo tcpdump -i eth0 -A port 80

# Show HEX and ASCII payload
sudo tcpdump -i eth0 -X port 80

# Show full packet contents (no truncation)
sudo tcpdump -i eth0 -X -s 0 port 80
```

```bash
10:35:22.123456 IP 192.168.1.5.54312 > 93.184.216.34.80: Flags [P.], length 78
        0x0000:  4500 0076 1234 4000 4006 ...  E..v.4@.@.
        0x0010:  c0a8 0105 5db8 d822 0050 ...  ....].."P
        GET / HTTP/1.1
        Host: example.com
        User-Agent: curl/7.74.0
```

> 💡 `-s 0` sets the snap length to unlimited — captures the **full packet payload** instead of just the first 68 bytes (default).

---

### Example 9 — Save Capture to a File

```bash
# Save to a .pcap file for later analysis (e.g., in Wireshark)
sudo tcpdump -i eth0 -w /tmp/capture.pcap

# Save with a timestamp in the filename
sudo tcpdump -i eth0 -w /tmp/capture_$(date +"%Y%m%d_%H%M%S").pcap

# Save with filter applied
sudo tcpdump -i eth0 -w /tmp/http_traffic.pcap port 80

# Read back a saved capture
sudo tcpdump -r /tmp/capture.pcap

# Read and filter a saved capture
sudo tcpdump -r /tmp/capture.pcap -nn host 192.168.1.5
```

> 💡 `.pcap` files can be opened in **Wireshark** for full GUI-based protocol dissection and analysis.

---

### Example 10 — Capture DNS Traffic

```bash
# Watch all DNS queries and responses
sudo tcpdump -i eth0 -nn udp port 53
```

```bash
10:35:22.123456 IP 192.168.1.5.49521 > 8.8.8.8.53: A? google.com.
10:35:22.131200 IP 8.8.8.8.53 > 192.168.1.5.49521: A google.com. 142.250.80.46
```

> 💡 Watching DNS traffic reveals DNS-based security issues, slow resolvers, and unexpected name lookups from suspicious processes.

---

### Example 11 — Capture HTTP Requests (Unencrypted)

```bash
# Capture HTTP GET requests
sudo tcpdump -i eth0 -A -s 0 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420)'
```

> The filter `0x47455420` is the hex encoding of `GET ` — this captures only packets containing HTTP GET requests.

```bash
# Simpler approach — capture all port 80 and grep for GET
sudo tcpdump -i eth0 -A -s 0 port 80 | grep -E "GET|POST|Host:"
```

```bash
GET /index.html HTTP/1.1
Host: example.com
```

---

### Example 12 — Detect Port Scans and SYN Floods

```bash
# Look for large numbers of SYN packets (potential port scan or SYN flood)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'
```

```bash
10:35:22 IP 203.0.113.10.49000 > 192.168.1.5.22:  Flags [S]
10:35:22 IP 203.0.113.10.49001 > 192.168.1.5.80:  Flags [S]
10:35:22 IP 203.0.113.10.49002 > 192.168.1.5.443: Flags [S]
10:35:22 IP 203.0.113.10.49003 > 192.168.1.5.3306: Flags [S]
```

> A single source IP sending SYN packets to many different ports in rapid succession is a classic **port scan**.

---

### Example 13 — Capture with Rotation (Long-term Capture)

```bash
# Rotate capture file every 100MB, keep last 10 files
sudo tcpdump -i eth0 \
  -w /var/captures/capture.pcap \
  -C 100 \        # Rotate at 100MB
  -W 10           # Keep max 10 files (circular buffer)
```

```bash
# Files created:
/var/captures/capture.pcap0
/var/captures/capture.pcap1
...
/var/captures/capture.pcap9   ← wraps back to pcap0
```

---

## 🗂️ `tcpdump` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-i interface` | Interface to capture on (`any` for all) |
| `-n` | No DNS resolution |
| `-nn` | No DNS and no port name resolution |
| `-c N` | Stop after capturing N packets |
| `-A` | Print packet payload in ASCII |
| `-X` | Print payload in HEX and ASCII |
| `-s N` | Snap length — bytes to capture per packet (0 = full packet) |
| `-w file` | Write raw packets to a `.pcap` file |
| `-r file` | Read packets from a `.pcap` file |
| `-C N` | Rotate output file every N MB |
| `-W N` | Limit number of rotated files (circular buffer) |
| `-t` | No timestamp |
| `-tttt` | Full date+time timestamp |
| `-v` | Verbose output |
| `-vv` | More verbose |
| `-vvv` | Maximum verbosity |
| `-D` | List available interfaces |
| `-e` | Show Ethernet (link-layer) header |
| `-q` | Quiet — less protocol info |

---

## 🔍 Common BPF Filter Expressions

| Filter | Description |
|--------|-------------|
| `host 1.2.3.4` | Traffic to or from IP |
| `src host 1.2.3.4` | Traffic from IP |
| `dst host 1.2.3.4` | Traffic to IP |
| `net 192.168.1.0/24` | Traffic to/from subnet |
| `port 80` | Traffic on port 80 (TCP or UDP) |
| `tcp port 443` | TCP traffic on port 443 |
| `udp port 53` | UDP DNS traffic |
| `portrange 8000-9000` | Traffic in port range |
| `not port 22` | All traffic except SSH |
| `icmp` | ICMP packets only |
| `tcp[tcpflags] & tcp-syn != 0` | TCP SYN packets |
| `greater 1000` | Packets larger than 1000 bytes |
| `less 64` | Packets smaller than 64 bytes |

---

## 🛠️ Network Capture Tools — Comparison

| Tool | Interface | Best For |
|------|-----------|----------|
| `tcpdump` | CLI | Fast packet capture, filtering, scripting |
| `Wireshark` | GUI | Deep protocol analysis, visual inspection |
| `tshark` | CLI | Wireshark's CLI — advanced dissection |
| `ngrep` | CLI | `grep`-style pattern matching on packets |
| `nethogs` | Interactive | Per-process bandwidth usage |
| `iftop` | Interactive | Per-connection bandwidth monitoring |
| `ss` / `netstat` | CLI | Socket and connection state inspection |

---

## 🚀 Full Workflow — Debug a Network Issue with tcpdump

```bash
# Step 1: Identify the interface carrying the traffic
ip link show
sudo tcpdump -D

# Step 2: Start a broad capture to see what is happening
sudo tcpdump -i eth0 -nn -c 200

# Step 3: Narrow down to the relevant host and port
sudo tcpdump -i eth0 -nn host 192.168.1.100 and port 443

# Step 4: Capture payload for inspection
sudo tcpdump -i eth0 -A -s 0 host 192.168.1.100 and port 80

# Step 5: Save for deeper analysis in Wireshark
sudo tcpdump -i eth0 -w /tmp/debug_$(date +%Y%m%d_%H%M%S).pcap \
  host 192.168.1.100 and port 443

# Step 6: Open in Wireshark (local or transfer and open)
# scp /tmp/debug_*.pcap user@workstation:~/

# Step 7: Read back and filter from command line
sudo tcpdump -r /tmp/debug_*.pcap -nn 'tcp[tcpflags] & tcp-rst != 0'
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Capturing too much traffic | Always apply filters — `host`, `port`, `not port 22` |
| DNS lookups slow down capture | Always use `-nn` in busy environments |
| Default snap length truncates payload | Use `-s 0` to capture full packet payload |
| Capture file grows without bound | Use `-C` and `-W` for rotation or `-c N` to limit count |
| Not enough privileges | `tcpdump` requires `root` or `CAP_NET_RAW` capability |
| Missing packets at high traffic rates | Increase buffer: `sudo tcpdump -B 4096 -i eth0` |

---

## 🔑 Key Takeaways

- `sudo tcpdump -i eth0` starts capturing — always **filter immediately** to avoid overwhelming output.
- Use `-nn` to suppress DNS and port-name lookups — faster and avoids false hostname mappings.
- **BPF filters** (`host`, `port`, `net`, `and`, `or`, `not`) are applied at the kernel level before packets reach `tcpdump` — extremely efficient.
- Use `-w capture.pcap` to save traffic for later analysis in **Wireshark** — the most powerful workflow.
- `-s 0` captures the **full packet payload** — essential for inspecting HTTP, DNS, and application data.
- A `[R]` or `[RST]` in the flags column means a connection was abruptly closed — always investigate why.
- For long-term captures, use `-C` and `-W` to implement **automatic file rotation**.

---

## 📚 Related Concepts

- [Wireshark](https://www.wireshark.org/) — GUI companion to tcpdump for deep packet analysis
- [tshark](https://www.wireshark.org/docs/man-pages/tshark.html) — Wireshark's CLI for advanced protocol dissection
- [BPF Filter Syntax](https://www.tcpdump.org/manpages/pcap-filter.7.html) — full BPF expression reference


</details>
