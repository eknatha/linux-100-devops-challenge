# Challenge 36: Check Open Ports in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Open ports** are network endpoints where a process is actively listening for incoming connections. Knowing which ports are open and which services own them is essential for:

- **Security auditing** — identifying unexpected or unauthorized listening services
- **Troubleshooting** — confirming a service is actually listening on the expected port
- **Firewall configuration** — knowing which ports to allow or block
- **Debugging connection failures** — verifying the service is listening before blaming the network

The main tools are `ss` (modern, recommended) and `netstat` (older but still widely used).

---

## 🧩 Task

List all ports that are currently open and listening for connections.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Modern tool (recommended)
ss -tlnp

# Legacy tool
netstat -tlnp

# Check a specific port
ss -tlnp | grep ':80'
```

### How it works

| Flag | Description |
|------|-------------|
| `-t` | **TCP** sockets only |
| `-l` | **Listening** sockets only |
| `-n` | **Numeric** — show port numbers, not service names |
| `-p` | Show the **process** (PID and name) owning the socket |
| `-u` | **UDP** sockets |

---

## 🔢 Understanding `ss -tlnp` Output

```bash
ss -tlnp
```

```
State    Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN   0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=801,fd=3))
LISTEN   0       511     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=5201,fd=6))
LISTEN   0       128     127.0.0.1:3306      0.0.0.0:*          users:(("mysqld",pid=2341,fd=21))
LISTEN   0       128     0.0.0.0:443         0.0.0.0:*          users:(("nginx",pid=5201,fd=7))
```

| Column | Description |
|--------|-------------|
| `State` | `LISTEN` = waiting for connections |
| `Local Address:Port` | IP and port the service is listening on |
| `0.0.0.0:port` | Listening on **all interfaces** (accessible from anywhere) |
| `127.0.0.1:port` | Listening on **localhost only** (local access only) |
| `:::port` | Listening on all IPv6 interfaces |
| `Process` | PID and name of the process owning the socket |

---

## 📖 Extended Examples

### Example 1 — List All Listening TCP Ports

```bash
ss -tlnp
```

---

### Example 2 — List Both TCP and UDP Listening Ports

```bash
# TCP + UDP
ss -tulnp

# TCP only
ss -tlnp

# UDP only
ss -ulnp
```

---

### Example 3 — List All Ports (Including Established Connections)

```bash
# All TCP connections (listening + established)
ss -tnp

# All sockets (TCP + UDP + Unix)
ss -anp
```

---

### Example 4 — Check a Specific Port

```bash
# Is anything listening on port 80?
ss -tlnp | grep ':80'

# Is anything listening on port 443?
ss -tlnp | grep ':443'

# Is anything listening on port 3306 (MySQL)?
ss -tlnp | grep ':3306'

# Check a custom application port
ss -tlnp | grep ':8080'
```

```bash
# Output for port 80:
LISTEN 0 511 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=5201,fd=6))
```

---

### Example 5 — Use `netstat` (Legacy Alternative)

```bash
# Install if not available
sudo apt install net-tools    # Debian/Ubuntu
sudo dnf install net-tools    # RHEL

# List all listening TCP ports
netstat -tlnp

# List all ports
netstat -anp
```

```bash
# netstat output:
Proto  Recv-Q  Send-Q  Local Address    Foreign Address  State   PID/Program
tcp    0       0       0.0.0.0:22       0.0.0.0:*        LISTEN  801/sshd
tcp    0       0       0.0.0.0:80       0.0.0.0:*        LISTEN  5201/nginx
tcp    0       0       127.0.0.1:3306   0.0.0.0:*        LISTEN  2341/mysqld
```

---

### Example 6 — Check Which Process Uses a Port with `lsof`

```bash
# Find which process is using port 80
sudo lsof -i :80

# Find which process is using port 443
sudo lsof -i :443

# Find all listening ports with lsof
sudo lsof -i -P -n | grep LISTEN
```

```bash
# lsof -i :80 output:
COMMAND  PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   5201  www-data   6u  IPv4  12345      0t0  TCP *:http (LISTEN)
nginx   5202  www-data   6u  IPv4  12346      0t0  TCP *:http (LISTEN)
```

---

### Example 7 — Check a Port on a Remote Host

```bash
# Test if a port is open on a remote host
nc -zv remote-host 80

# Test multiple ports
nc -zv remote-host 22 80 443

# Test with a timeout
nc -zv -w 3 remote-host 3306
```

```bash
# nc output:
Connection to remote-host 80 port [tcp/http] succeeded!
Connection to remote-host 443 port [tcp/https] succeeded!
nc: connect to remote-host port 3306 failed: Connection refused
```

---

### Example 8 — Scan Open Ports with `nmap`

```bash
# Install nmap
sudo apt install nmap    # Debian/Ubuntu
sudo dnf install nmap    # RHEL

# Scan most common ports on localhost
nmap localhost

# Scan all 65535 ports
nmap -p- localhost

# Scan specific ports
nmap -p 22,80,443,3306 localhost

# Scan a remote host
nmap 192.168.1.100
```

```bash
# nmap output:
Starting Nmap 7.93 ( https://nmap.org )
Nmap scan report for localhost (127.0.0.1)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql
```

---

### Example 9 — Filter Ports by Accessibility

```bash
# Ports accessible from anywhere (0.0.0.0 = all interfaces)
ss -tlnp | grep "0.0.0.0"

# Ports accessible locally only (127.0.0.1 = localhost)
ss -tlnp | grep "127.0.0.1"

# Ports on all IPv6 interfaces
ss -tlnp | grep ":::"
```

---

### Example 10 — Monitor Port Activity in Real-Time

```bash
# Watch for changes in listening ports every 2 seconds
watch -n 2 'ss -tlnp'

# Monitor specific port activity
watch -n 1 'ss -tnp | grep ":80"'
```

---

### Example 11 — Common Port Reference Check

```bash
# Check all common service ports at once
for PORT in 22 25 53 80 443 3306 5432 6379 8080 8443 27017; do
  result=$(ss -tlnp | grep ":$PORT ")
  if [ -n "$result" ]; then
    echo "✅ Port $PORT: OPEN"
    echo "   $result"
  else
    echo "❌ Port $PORT: closed"
  fi
done
```

```bash
✅ Port 22: OPEN
   LISTEN 0 128 0.0.0.0:22 0.0.0.0:* users:(("sshd",pid=801))
✅ Port 80: OPEN
   LISTEN 0 511 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=5201))
❌ Port 25: closed
❌ Port 3306: closed
```

---

### Example 12 — Security Audit — Find Unexpected Listening Ports

```bash
#!/bin/bash
# Find all listening ports and compare against expected list

EXPECTED_PORTS=(22 80 443)    # Define expected ports
echo "=== Open Port Security Audit ==="
echo ""

# Get all listening ports
OPEN_PORTS=$(ss -tlnp | awk 'NR>1 {split($4, a, ":"); print a[length(a)]}' | sort -un)

for PORT in $OPEN_PORTS; do
  EXPECTED=false
  for EXP in "${EXPECTED_PORTS[@]}"; do
    [ "$PORT" -eq "$EXP" ] && EXPECTED=true && break
  done

  PROCESS=$(ss -tlnp | grep ":${PORT} " | awk '{print $6}')
  if $EXPECTED; then
    echo "✅ Port $PORT — expected — $PROCESS"
  else
    echo "⚠️  Port $PORT — UNEXPECTED! — $PROCESS"
  fi
done
```

---

## 🗂️ Common Well-Known Ports

| Port | Protocol | Service |
|------|----------|---------|
| 22 | TCP | SSH |
| 25 | TCP | SMTP (email) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | Alternative HTTP |
| 27017 | TCP | MongoDB |

---

## 🗂️ Port Monitoring Tools — Quick Reference

| Tool | Description |
|------|-------------|
| `ss -tlnp` | Modern — list listening TCP ports with process |
| `ss -tulnp` | Modern — list TCP + UDP listening ports |
| `netstat -tlnp` | Legacy — equivalent to ss -tlnp |
| `lsof -i :PORT` | Show process using a specific port |
| `nc -zv host PORT` | Test if a remote port is open |
| `nmap localhost` | Scan all common ports |
| `fuser PORT/tcp` | Find process ID using a port |

---

## 🚀 Full Workflow — Verify a Service is Listening

```bash
# Step 1: Start the service
sudo systemctl start nginx

# Step 2: Check it is listening on the expected port
ss -tlnp | grep ':80'

# Step 3: Verify the firewall allows the port
sudo firewall-cmd --list-all | grep 80   # firewalld
sudo ufw status | grep 80                 # ufw

# Step 4: Test locally
curl http://localhost

# Step 5: Test the specific port
nc -zv localhost 80

# Step 6: Check from a remote host (if applicable)
nc -zv server-ip 80

# Step 7: If not accessible remotely but open locally
# — check firewall rules
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Port open but service not reachable remotely | Check firewall — port may be blocked by `firewalld`/`ufw` |
| `ss` requires root to see process names | Use `sudo ss -tlnp` for process info |
| `netstat` not installed | Install `net-tools` or switch to `ss` (built-in) |
| Port shows as listening on `127.0.0.1` | Service is localhost-only — change bind address in config |
| Service configured but port not open | Service may have failed to start — check `systemctl status` |
| Port conflict — two services on same port | One fails to start — check `systemctl status` for the error |

---

## 🔑 Key Takeaways

- `ss -tlnp` is the modern, recommended tool — `netstat -tlnp` is the legacy alternative.
- `0.0.0.0:port` means the service accepts connections from **any interface** — accessible remotely.
- `127.0.0.1:port` means **localhost only** — not accessible from outside the machine.
- Use `sudo ss -tlnp` to see the **process name and PID** — without sudo, the process column is empty.
- `lsof -i :PORT` identifies which process is using a specific port.
- `nc -zv host PORT` quickly tests if a remote port is reachable.
- An open port on the server **does not guarantee accessibility** if a firewall is blocking it.

---

## 📚 Related Concepts

- [ss Manual](https://linux.die.net/man/8/ss)
- [nmap Documentation](https://nmap.org/book/man.html)

</details>
