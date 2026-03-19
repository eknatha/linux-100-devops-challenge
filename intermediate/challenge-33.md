# Challenge 33: Monitor Logs in Real-Time

## 🧩 Task
View logs in real-time.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Monitor logs in real-time
```bash
tail -f /var/log/messages
```

### 🔹 Example output:
```text
Mar 19 10:15:01 server sshd[1203]: Accepted password for user from 10.0.0.1 port 54321
Mar 19 10:15:05 server systemd[1]: Started Session 5 of user user.
Mar 19 10:15:10 server nginx[890]: 200 GET /index.html
```

---

### 📖 Explanation
- `tail` → Displays the last part of a file  
- `-f` → Follows the file in real-time (updates continuously)  
- Useful for monitoring logs as events happen  

---

### ⚠️ System Differences
- **RHEL / CentOS / Amazon Linux:**
```bash
tail -f /var/log/messages
```

- **Ubuntu / Debian:**
```bash
tail -f /var/log/syslog
```

---

### 💡 Tips
- Press `Ctrl + C` to stop monitoring  
- Use `tail -n 50 -f file.log` to view last 50 lines + follow  
- Combine with `grep` for filtering logs:
```bash
tail -f /var/log/syslog | grep "error"
```

</details>
