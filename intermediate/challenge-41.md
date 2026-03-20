# Challenge 41: Find Large Files

## 🧩 Task
Find files larger than 100MB.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Find files larger than 100MB
```bash
find / -type f -size +100M
```

---

### 🔹 Find and list details (human-readable)
```bash
find / -type f -size +100M -exec ls -lrth {} \;
```

---

### 🔹 Example output:
```text
-rw-r--r-- 1 user user 120M Mar 19 10:00 /var/log/bigfile.log
-rw-r--r-- 1 user user 250M Mar 18 09:30 /home/user/archive.tar.gz
```

---

### 📖 Explanation
- `find /` → Search starting from root directory  
- `-type f` → Look for files only  
- `-size +100M` → Files larger than 100MB  
- `-exec ls -lrth {} \;` → Executes `ls -lrth` on each result:
  - `-l` → Long listing  
  - `-r` → Reverse order  
  - `-t` → Sort by time  
  - `-h` → Human-readable sizes  

---

### 💡 Tips
- Search in current directory only:
```bash
find . -type f -size +100M
```

- Suppress permission errors:
```bash
find / -type f -size +100M 2>/dev/null
```

- Find files larger than 1GB:
```bash
find / -type f -size +1G
```

---

### 🚀 Use Cases
- Disk cleanup and storage optimization  
- Finding large log or backup files  
- Troubleshooting disk space issues  

</details>
