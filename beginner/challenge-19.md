# Challenge 19: Kill a Process

## 🧩 Task
Terminate a process.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Terminate a process gracefully
```bash
kill <PID>
```

### 🔹 Force kill a process
```bash
kill -9 <PID>
```

**Example:**
```bash
kill 2450
kill -9 2450
```

### 📖 Explanation
- **kill <PID>** → Sends a default signal (SIGTERM) to allow the process to exit gracefully  
- **kill -9 <PID>** → Sends SIGKILL, forcefully terminating the process immediately (cannot be ignored)  
- **PID** → Process ID obtained using commands like `ps aux`, `top`, etc.  

</details>
