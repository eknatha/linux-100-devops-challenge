# Challenge 26: Create Alias

## 🧩 Task
Create a shortcut command.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Create an alias
```bash
alias ll='ls -la'
```

### 🔹 Example 1: List files using alias
```bash
ll
```

**Output:**
```text
total 12
drwxr-xr-x  3 user user 4096 Mar 19 10:00 .
drwxr-xr-x 10 user user 4096 Mar 19 09:50 ..
-rw-r--r--  1 user user   20 Mar 19 10:00 file.txt
```

---

### 🔹 Example 2: Create a custom alias for updating system (DevOps use)
```bash
alias update='sudo apt update && sudo apt upgrade'
```

**Usage:**
```bash
update
```

---

### 🔹 Example 3: Shortcut for frequently used command
```bash
alias gs='git status'
```

**Usage:**
```bash
gs
```

### 📖 Explanation
- `alias` → Creates a shortcut for a longer command  
- Helps improve productivity by reducing repetitive typing  
- Commonly used in DevOps workflows for tools like `git`, `kubectl`, etc.  

**Tip:**
To make aliases permanent, add them to:
- `~/.bashrc`
- `~/.zshrc`

</details>
