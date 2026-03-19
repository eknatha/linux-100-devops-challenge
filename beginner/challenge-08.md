# Challenge 08: Logged-in Users

## 🧩 Task
Check logged-in users.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Basic check
```bash
who
```

**Example output:**
```bash
root     pts/0        2026-03-18 19:10 (10.X.X.177)
```

### 🔹 Display all details of current logged-in users
```bash
who -a
```

**Example output:**
```text
           system boot  YYYY-MM-DD HH:MM
LOGIN      tty1         YYYY-MM-DD HH:MM              #### id=tty1
           run-level 5  YYYY-MM-DD HH:MM
<user>    + tty7         YYYY-MM-DD HH:MM   ..        #### (:0)
```

</details>
