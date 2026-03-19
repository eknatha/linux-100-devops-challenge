# Challenge 04: Change File Permissions

## 🧩 Task
Give read, write, execute permissions to the owner.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Set permissions
```bash
chmod 700 file.txt
```

### 🔹 Verify permissions
```bash
ls -lrth file.txt
# Output:
# -rwx------ Owner group size date file.txt
```

</details>
