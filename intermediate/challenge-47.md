# Challenge 47: Monitor Network Traffic

## 🧩 Task
Monitor bandwidth usage.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Monitor traffic using iftop
```bash
iftop
```

**Example output:**
```text
TX: 1.25Mb  RX: 980Kb
192.168.1.10 => 142.250.183.206   500Kb   300Kb   200Kb
```

---

### 🔹 Monitor bandwidth using nload
```bash
nload
```

**Example output:**
```text
Incoming: 1.20 Mbit/s
Outgoing: 800 Kbit/s
```

---

### 📖 Explanation
- `iftop` → Displays real-time network connections and bandwidth usage per host  
- `nload` → Shows overall incoming and outgoing traffic in a simple interface  

---

### 💡 Tips
- Install tools if not available:
```bash
sudo apt install iftop nload   # Debian/Ubuntu
sudo yum install iftop nload   # RHEL/CentOS
```

- Run with sudo for full visibility:
```bash
sudo iftop
sudo nload
```

---

### 🚀 Use Cases
- Detecting high bandwidth usage  
- Identifying suspicious network activity  
- Monitoring traffic during deployments or incidents  

</details>
