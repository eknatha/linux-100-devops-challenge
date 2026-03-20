# Challenge 42: Check Network Connectivity

## 🧩 Task
Test connectivity to a host.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Check connectivity using ping
```bash
ping google.com
```

**Example output:**
```text
PING google.com (142.250.183.206) 56(84) bytes of data.
64 bytes from 142.250.183.206: icmp_seq=1 ttl=115 time=20.5 ms
64 bytes from 142.250.183.206: icmp_seq=2 ttl=115 time=19.8 ms
```

---

### 🔹 Trace network path
```bash
traceroute www.amazon.com
```

**Example output:**
```text
1  192.168.1.1     1.123 ms
2  10.0.0.1        5.456 ms
3  172.16.0.1     10.789 ms
...
```

---

### 📖 Explanation
- `ping` → Tests connectivity by sending ICMP packets to a host  
- Shows:
  - Response time (latency)  
  - Packet loss  
- `traceroute` → Displays the path packets take to reach a destination  
- Helps identify network hops and delays  

---

### 💡 Tips
- Stop `ping` using `Ctrl + C`  
- Send limited packets:
```bash
ping -c 4 google.com
```

- If `traceroute` is not installed:
```bash
sudo apt install traceroute   # Debian/Ubuntu
sudo yum install traceroute   # RHEL/CentOS
```

---

### 🚀 Use Cases
- Debugging network connectivity issues  
- Identifying slow network paths  
- Verifying external service availability  

</details>
