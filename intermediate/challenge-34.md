# Challenge 34: Configure Cron Job

## 🧩 Task
Schedule a job to run every minute.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Edit crontab
```bash
crontab -e
```

### 🔹 Add cron job (runs every minute)
```bash
* * * * * echo "Hello" >> /tmp/test.log
```

---

### 🔹 Example output (inside `/tmp/test.log`)
```text
Hello
Hello
Hello
```

---

### 📖 Explanation
- `crontab -e` → Opens the cron configuration file for editing  
- `* * * * *` → Schedule format:
  - Minute (0–59)
  - Hour (0–23)
  - Day of month (1–31)
  - Month (1–12)
  - Day of week (0–7)  
- `echo "Hello"` → Command to execute  
- `>> /tmp/test.log` → Appends output to a file  

---

### 💡 Tips
- Use `crontab -l` to list current cron jobs  
- Use `crontab -r` to remove all cron jobs  
- Always use full paths in cron jobs (e.g., `/usr/bin/echo`) for reliability  

</details>
