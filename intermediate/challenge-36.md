# Challenge 36: Check Open Ports

## 🧩 Task
List all listening ports.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 List all listening ports
```bash
ss -tulnp
```

---

### 🔹 Example output:
```text
Netid State   Recv-Q Send-Q Local Address:Port   Peer Address:Port Process
tcp   LISTEN  0      128    0.0.0.0:22           0.0.0.0:*         users:(("sshd",pid=890))
tcp   LISTEN  0      128    0.0.0.0:80           0.0.0.0:*         users:(("nginx",pid=1203))
udp   UNCONN  0      0      0.0.0.0:68           0.0.0.0:*         users:(("dhclient",pid=500))
```

---

### 📖 Explanation
- `ss` → Utility to display socket (network) information  
- `-t` → Show TCP sockets  
- `-u` → Show UDP sockets  
- `-l` → Show listening sockets only  
- `-n` → Show numeric addresses (no DNS resolution)  
- `-p` → Show process using the port  

---

### 💡 Tips
- Check a specific port:
```bash
ss -tulnp | grep :80
```

- Alternative (older systems):
```bash
netstat -tulnp
```

- Useful for:
  - Debugging services not accessible  
  - Verifying if a port is open  
  - Identifying which process is using a port  

</details>
