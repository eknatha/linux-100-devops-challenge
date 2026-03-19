# Challenge 38: Monitor System Performance

## 🧩 Task
Monitor system stats in real time.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Monitor system performance
```bash
vmstat 1
```

---

### 🔹 Example output:
```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 500000  20000 300000    0    0    10    20  100  200 10  5 80  5  0
```

---

### 📖 Explanation
- `vmstat` → Reports system performance (CPU, memory, I/O, processes)  
- `1` → Refresh interval in seconds  

**Key fields:**
- **r** → Processes waiting for CPU  
- **b** → Processes in uninterruptible sleep (I/O wait)  
- **swpd** → Swap memory used  
- **free** → Free memory  
- **buff/cache** → Memory used for buffers/cache  
- **bi/bo** → Blocks in/out (disk I/O)  
- **in/cs** → Interrupts / context switches  
- **us** → User CPU time  
- **sy** → System CPU time  
- **id** → Idle CPU time  
- **wa** → Time waiting for I/O  

---

### 💡 Tips
- High **r** → CPU contention (too many processes waiting)  
- High **wa** → Disk I/O bottleneck  
- Low **free memory** + high swap → Memory pressure  
- Useful for diagnosing:
  - CPU bottlenecks  
  - Memory issues  
  - Disk I/O problems  

</details>
