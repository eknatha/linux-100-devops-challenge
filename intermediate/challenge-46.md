# Challenge 46: Download File

## 🧩 Task
Download a file.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Download a file using wget
```bash
wget http://example.com/file.zip
```

---

### 🔹 Example output:
```text
--2026-03-19--  http://example.com/file.zip
Resolving example.com... 93.184.216.34
Connecting to example.com... connected.
HTTP request sent, awaiting response... 200 OK
Length: 102400 (100K) [application/zip]
Saving to: ‘file.zip’

file.zip      100%[===================>] 100K  --.-KB/s   in 0.01s
```

---

### 📖 Explanation
- `wget` → Command-line tool to download files from the web  
- `http://example.com/file.zip` → URL of the file to download  
- Automatically saves the file in the current directory  

---

### 💡 Tips
- Download and save with a custom name:
```bash
wget -O newname.zip http://example.com/file.zip
```

- Resume interrupted download:
```bash
wget -c http://example.com/file.zip
```

- Download in background:
```bash
wget -b http://example.com/file.zip
```

---

### 🚀 Use Cases
- Downloading artifacts in CI/CD pipelines  
- Fetching backups or logs  
- Automating file downloads in scripts  

</details>
