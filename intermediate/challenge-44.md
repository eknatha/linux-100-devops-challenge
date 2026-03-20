# Challenge 44: DNS Resolution

## 🧩 Task
Check DNS resolution.

---

<details>
<summary>💡 Click to view solution</summary>

### 🔹 Check DNS resolution
```bash
nslookup google.com
```

---

### 🔹 Example output:
```text
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.183.206
```

---

### 📖 Explanation
- `nslookup` → Queries DNS servers to resolve domain names  
- **Server** → DNS server used for the query  
- **Name** → Domain name queried  
- **Address** → IP address returned  

---

### 💡 Tips
- Use a specific DNS server:
```bash
nslookup google.com 8.8.8.8
```

- Alternative tool (more detailed):
```bash
dig google.com
```

---

### 🚀 Use Cases
- Debugging DNS issues  
- Verifying domain resolution  
- Checking DNS server responses  

</details>
