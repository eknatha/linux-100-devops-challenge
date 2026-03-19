# Challenge 30: Create Symbolic Link

## 🧩 Task
Create a symbolic link.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Create a symbolic (soft) link
```bash
ln -s file.txt link.txt
```

### 🔹 Example usage
```bash
ls -l
```

**Example output:**
```text
-rw-r--r-- 1 user user  20 Mar 19 10:00 file.txt
lrwxrwxrwx 1 user user   8 Mar 19 10:01 link.txt -> file.txt
```

### 📖 Explanation
- `ln` → Command used to create links  
- `-s` → Creates a symbolic (soft) link  
- `file.txt` → Original file (source)  
- `link.txt` → Link pointing to the original file  

**Key points:**
- Symbolic links act as shortcuts to files/directories  
- If the original file is deleted, the symlink becomes broken  
- Useful for configuration files, shared libraries, and shortcuts  

</details>
