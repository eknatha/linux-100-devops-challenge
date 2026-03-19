# Challenge 28: System Information

## 🧩 Task
Display system information.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Display system information
```bash
uname -a
```

**Example output:**
```text
Linux myhost 5.15.0-91-generic #101-Ubuntu SMP x86_64 GNU/Linux
```

### 📖 Explanation
- **Linux** → Kernel name  
- **myhost** → Hostname  
- **5.15.0-91-generic** → Kernel version  
- **#101-Ubuntu SMP** → Build and distribution info  
- **x86_64** → System architecture (64-bit)  
- **GNU/Linux** → Operating system type  

**Tip:**
Other useful variations:
```bash
uname -r   # Kernel version only
uname -n   # Hostname
uname -m   # Machine architecture
```

</details>
