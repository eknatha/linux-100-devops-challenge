# Challenge 18: View Running Processes

## 🧩 Task
List running processes.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 List running processes
```bash
ps aux
```

**Example output:**
```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  170000  9000 ?        Ss   10:00   0:01 /sbin/init
user      2345  0.2  1.5  500000 60000 ?        S    10:05   0:02 nginx
user      2450  0.5  2.0  800000 80000 pts/0    R+   10:10   0:00 python app.py
```

### 📖 Explanation
- **USER** → Owner of the process  
- **PID** → Process ID  
- **%CPU** → CPU usage percentage  
- **%MEM** → Memory usage percentage  
- **VSZ** → Virtual memory size  
- **RSS** → Resident Set Size (actual memory used)  
- **TTY** → Terminal associated with the process  
- **STAT** → Process state (R=running, S=sleeping, Z=zombie, etc.)  
- **START** → Start time of the process  
- **TIME** → Total CPU time consumed  
- **COMMAND** → Command that started the process  

</details>
