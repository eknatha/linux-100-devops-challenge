# Challenge 40: Text Processing with sed

## 🧩 Task
Replace a word in a file.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Replace a word in a file
```bash
sed 's/old/new/g' file.txt
```

---

### 🔹 Example input (`file.txt`)
```text
hello world
hello linux
hello devops
```

### 🔹 Output:
```text
hello world
hello linux
hello devops
```

### 🔹 Example with replacement
```bash
sed 's/hello/hi/g' file.txt
```

**Output:**
```text
hi world
hi linux
hi devops
```

---

### 📖 Explanation
- `sed` → Stream editor used for text transformation  
- `s/old/new/g` → Substitute:
  - `s` → substitution command  
  - `old` → word to replace  
  - `new` → replacement word  
  - `g` → global (replace all occurrences in each line)  

---

### 💡 Tips
- Replace only first occurrence per line (omit `g`):
```bash
sed 's/hello/hi/' file.txt
```

- Replace in-place (modify file):
```bash
sed -i 's/hello/hi/g' file.txt
```

- Case-insensitive replacement:
```bash
sed 's/hello/hi/Ig' file.txt
```

---

### 🚀 Use Cases
- Log transformation  
- Configuration updates  
- Bulk text replacements in files  

</details>
