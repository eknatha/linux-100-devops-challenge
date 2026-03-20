# Challenge 73: Disk Space Issue

## 🧩 Task
Find what is filling disk.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Check overall disk usage
```bash
df -h
```

**Example output:**
```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   45G  3.5G  93% /
/dev/sdb1       100G   20G   80G  20% /data
```

---

### 🔹 Find largest directories
```bash
du -sh /* | sort -hr | head
```

**Example output:**
```text
40G /var
3G  /home
1G  /usr
500M /etc
```

---

### 📖 Explanation
- `df -h` → Shows disk usage of mounted filesystems in human-readable format  
  - Helps identify which partition is full  
- `du -sh /*` → Shows disk usage of top-level directories  
  - `-s` → Summary per directory  
  - `-h` → Human-readable sizes  
- `sort -hr` → Sorts results by size in reverse (largest first)  
- `head` → Displays top entries  

---

### 💡 Tips
- Drill down into a specific directory:
```bash
du -sh /var/*
```

- Ignore permission errors:
```bash
du -sh /* 2>/dev/null | sort -hr | head
```

- Find very large files:
```bash
find / -type f -size +100M 2>/dev/null
```

---

### 🚀 Use Cases
- Identifying disk space leaks  
- Cleaning up logs and temporary files  
- Troubleshooting “No space left on device” errors  

</details>
