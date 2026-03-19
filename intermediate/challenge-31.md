# Challenge 31: Top CPU Consuming Processes

## 🧩 Task
Find processes consuming the most CPU.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Find top CPU consuming processes
```bash
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
```

### 🔹 Example output:
```text
  PID  PPID CMD                         %CPU
 2450     1 /usr/bin/python app.py       35.2
 1203     1 /usr/bin/java -jar app.jar   22.8
  890     1 nginx: worker process        10.5
  450     1 /usr/sbin/sshd               2.1
  101     0 /sbin/init                   1.0
```

### 📖 Explanation
- `ps` → Displays process information  
- `-e` → Shows all processes  
- `-o pid,ppid,cmd,%cpu` → Custom output format:
  - **PID** → Process ID  
  - **PPID** → Parent Process ID  
  - **CMD** → Command that started the process  
  - **%CPU** → CPU usage percentage  
- `--sort=-%cpu` → Sorts processes by CPU usage in descending order  
- `| head` → Shows top results only  

**Tip:**
- This command is very useful for identifying **CPU bottlenecks** in real systems  
- Often used in DevOps/SRE troubleshooting scenarios  

</details>
