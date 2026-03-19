# Challenge 39: Process Data with awk

## 🧩 Task
Print the first column from a file.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Print first column
```bash
awk '{print $1}' file.txt
```

---

### 🔹 Example input (`file.txt`)
```text
apple red 10
banana yellow 20
grape purple 15
```

### 🔹 Output:
```text
apple
banana
grape
```

---

### 📖 Explanation
- `awk` → Text processing tool used for pattern scanning and data extraction  
- `{print $1}` → Prints the first field (column) of each line  
- `$1` → Refers to the first column (fields are separated by whitespace by default)  

---

### 💡 Tips
- Print second column:
```bash
awk '{print $2}' file.txt
```

- Print multiple columns:
```bash
awk '{print $1, $3}' file.txt
```

- Change delimiter (e.g., comma-separated file):
```bash
awk -F ',' '{print $1}' file.csv
```

---

### 🚀 Use Cases
- Log parsing  
- CSV/structured data processing  
- Extracting specific fields from command outputs  

</details>
