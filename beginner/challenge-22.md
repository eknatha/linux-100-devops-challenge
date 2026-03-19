# Challenge 22: Locate Files

## 🧩 Task
Find files quickly using `locate`.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Find a file using locate
```bash
locate file.txt
```

**Example output:**
```text
/home/user/file.txt
/var/log/file.txt
/usr/share/doc/file.txt
```

### 📖 Explanation
- `locate` → Searches a pre-built database of files on the system  
- It is much faster than `find` because it does not scan the filesystem in real time  
- Output depends on the database updated by `updatedb`  

**Important:**
If results are outdated or missing, update the database:
```bash
sudo updatedb
```

**Tip:**
- Use `locate -i file.txt` for case-insensitive search  

</details>
