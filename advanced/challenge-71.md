# Challenge 71: Diagnose High CPU Usage

## 🧩 Task
Identify processes causing high CPU usage.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Real-time CPU monitoring
```bash
top
```

**Example output:**
```text
top - 10:30:01 up 2 days,  3:45,  2 users,  load average: 1.25, 0.98, 0.75
%Cpu(s): 35.0 us,  5.0 sy,  0.0 ni, 60.0 id
PID   USER   PR  NI  VIRT   RES   SHR S %CPU %MEM TIME+ COMMAND
2450  user   20   0  500m  60m  10m R 35.2  1.5  0:10 python
1203  user   20   0  800m 120m  15m S 22.8  2.5  0:20 java
```

---

### 🔹 Find top CPU consuming processes (snapshot)
```bash
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

**Example output:**
```text
 PID CMD                         %CPU
2450 python app.py               35.2
1203 java -jar app.jar           22.8
 890 nginx: worker process       10.5
```

---

### 📖 Explanation
- `top` → Displays real-time system usage including CPU, memory, and running processes  
- `%CPU` → Percentage of CPU used by each process  
- `load average` → System load over 1, 5, and 15 minutes  
- `ps -eo ... --sort=-%cpu` → Lists processes sorted by highest CPU usage  
- `head` → Shows top results  

**When to use:**
- Use `top` for **live monitoring**  
- Use `ps` for a **quick snapshot**  

**Tip:**
- Press `q` to quit `top`  
- Press `P` inside `top` to sort by CPU usage  

</details>
