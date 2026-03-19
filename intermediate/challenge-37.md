# Challenge 37: Analyze Disk I/O

## 🧩 Task
Check disk performance.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Monitor disk I/O in real-time
```bash
iostat -x 1
```

---

### 🔹 Example output:
```text
Device            r/s     w/s   rkB/s   wkB/s  %util
sda              10.5     5.2   120.0    80.0   35.0
sdb               2.0     1.0    20.0    10.0    5.0
```

---

### 📖 Explanation
- `iostat` → Reports CPU and disk I/O statistics  
- `-x` → Shows extended statistics (detailed disk metrics)  
- `1` → Refresh interval in seconds  

**Key fields:**
- **r/s** → Read requests per second  
- **w/s** → Write requests per second  
- **rkB/s / wkB/s** → Data read/written per second  
- **%util** → Percentage of time the disk is busy  

---

### 💡 Tips
- High **%util (~100%)** → Disk is fully utilized (possible bottleneck)  
- High read/write values → Heavy disk activity  
- Useful for diagnosing:
  - Slow applications  
  - Database performance issues  
  - High disk latency  

**Note:**
If `iostat` is not installed:
```bash
sudo apt install sysstat     # Debian/Ubuntu
sudo yum install sysstat     # RHEL/CentOS
```

</details>
