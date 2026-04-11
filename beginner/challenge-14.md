# Challenge 14: Sort File Content in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)


## 📌 Overview

The `sort` command arranges the lines of a file or input stream in a specified order — alphabetically, numerically, by a specific column, or in reverse. It is one of the most frequently used text processing tools in Linux, especially when combined with pipes for data analysis, log processing, and reporting.

Common real-world uses:
- Sort log entries alphabetically or by timestamp
- Sort a list of usernames or hostnames
- Find the largest files by sorting `du` output numerically
- Remove duplicates from sorted data using `uniq`
- Sort CSV data by a specific column

---

## 🧩 Task

Sort the lines of a file in alphabetical or numerical order.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Sort alphabetically (default)
sort filename.txt

# Sort in reverse order
sort -r filename.txt

# Sort numerically
sort -n filename.txt
```

### How it works

| Flag | Description |
|------|-------------|
| `sort` (no flags) | Sort lines **alphabetically** (A–Z) |
| `-r` | Sort in **reverse** order (Z–A or descending) |
| `-n` | Sort **numerically** (treats values as numbers, not text) |

---

## 🔢 Alphabetical vs Numerical Sorting

```bash
# File content:
cat numbers.txt
10
9
100
2
20

# Alphabetical sort (default) — wrong for numbers!
sort numbers.txt
10
100
2
20
9

# Numerical sort — correct!
sort -n numbers.txt
2
9
10
20
100
```

> ⚠️ Always use `-n` when sorting numbers — alphabetical sort treats `10` as less than `9` because `1` comes before `9`.

---

## 📖 Extended Examples

### Example 1 — Basic Alphabetical Sort

```bash
cat fruits.txt
banana
apple
cherry
date
mango

sort fruits.txt
```

```bash
apple
banana
cherry
date
mango
```

---

### Example 2 — Sort in Reverse Order

```bash
sort -r fruits.txt
```

```bash
mango
date
cherry
banana
apple
```

---

### Example 3 — Numerical Sort

```bash
cat scores.txt
45
12
89
3
67
100

sort -n scores.txt
```

```bash
3
12
45
67
89
100
```

---

### Example 4 — Sort and Save to a File

```bash
# Sort and save to a new file
sort filename.txt > sorted.txt

# Sort in place (overwrite original)
sort -o filename.txt filename.txt
```

---

### Example 5 — Sort Case-Insensitively

```bash
cat mixed.txt
Banana
apple
Cherry
date

# Default sort (case-sensitive — uppercase first)
sort mixed.txt
# Banana
# Cherry
# apple
# date

# Case-insensitive sort
sort -f mixed.txt
# apple
# Banana
# Cherry
# date
```

---

### Example 6 — Sort by a Specific Column (`-k`)

```bash
cat employees.txt
Alice  Engineering  75000
Bob    Marketing    62000
Carol  Engineering  85000
Dave   Marketing    58000

# Sort by column 2 (department)
sort -k2 employees.txt
```

```bash
Alice  Engineering  75000
Carol  Engineering  85000
Bob    Marketing    62000
Dave   Marketing    58000
```

```bash
# Sort by column 3 (salary) numerically
sort -k3 -n employees.txt
```

```bash
Dave   Marketing    58000
Bob    Marketing    62000
Alice  Engineering  75000
Carol  Engineering  85000
```

---

### Example 7 — Sort by Multiple Columns

```bash
# Sort by column 2 (department), then by column 3 (salary) numerically
sort -k2 -k3n employees.txt
```

```bash
Alice  Engineering  75000
Carol  Engineering  85000
Dave   Marketing    58000
Bob    Marketing    62000
```

---

### Example 8 — Sort and Remove Duplicates (`-u`)

```bash
cat duplicates.txt
apple
banana
apple
cherry
banana
date

sort -u duplicates.txt
```

```bash
apple
banana
cherry
date
```

> 💡 `-u` (unique) removes duplicate lines after sorting — cleaner than using `sort | uniq` separately.

---

### Example 9 — Sort Human-Readable Sizes (`-h`)

```bash
# Sort du output by human-readable size
du -sh /var/log/* 2>/dev/null | sort -hr
```

```bash
 18G  /var/log/journal
6.0G  /var/log/nginx
3.0G  /var/log/mysql
512M  /var/log/syslog
 40M  /var/log/auth.log
```

> 💡 `-h` understands human-readable sizes like `18G`, `6.0G`, `512M` — essential for sorting `du` and `ls -lh` output correctly.

---

### Example 10 — Sort with Custom Field Separator (`-t`)

```bash
cat data.csv
Alice,Engineering,75000
Bob,Marketing,62000
Carol,Engineering,85000
Dave,Marketing,58000

# Sort by the 3rd column (salary) using comma as separator
sort -t',' -k3 -n data.csv
```

```bash
Dave,Marketing,58000
Bob,Marketing,62000
Alice,Engineering,75000
Carol,Engineering,85000
```

---

### Example 11 — Sort Piped Output

```bash
# Sort running processes by CPU usage
ps aux --sort=-%cpu | head -10

# Sort directory contents by file size
ls -la | sort -k5 -n

# Find top 5 largest files
du -sh /var/log/* 2>/dev/null | sort -hr | head -5

# Sort unique IP addresses from a log
awk '{print $1}' /var/log/nginx/access.log | sort -u
```

---

### Example 12 — Sort and Count with `uniq -c`

```bash
# Count occurrences of each value — must sort first!
cat colors.txt
red
blue
red
green
blue
red

sort colors.txt | uniq -c | sort -rn
```

```bash
      3 red
      2 blue
      1 green
```

> 💡 **`sort | uniq -c | sort -rn`** is one of the most powerful text analysis patterns in Linux — sort first, count occurrences, then sort by count descending.

---

## 🗂️ `sort` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-n` | **Numeric** sort (numbers treated as numbers) |
| `-r` | **Reverse** sort order |
| `-f` | **Fold** case — case-insensitive sort |
| `-u` | **Unique** — remove duplicate lines |
| `-k N` | Sort by **column N** |
| `-t CHAR` | Use CHAR as **field separator** (default: whitespace) |
| `-h` | **Human-readable** sort (understands K, M, G suffixes) |
| `-V` | **Version** sort (natural sort for version numbers) |
| `-o FILE` | Write output to FILE (sort in place) |
| `-c` | **Check** if file is already sorted (no output if sorted) |
| `-R` | **Random** shuffle (randomize line order) |
| `-b` | Ignore **leading blanks** |
| `-M` | Sort by **month name** (Jan, Feb, ...) |

---

## 🔄 Common Sort Patterns

| Goal | Command |
|------|---------|
| Sort alphabetically | `sort file.txt` |
| Sort numerically | `sort -n file.txt` |
| Sort in reverse | `sort -r file.txt` |
| Sort largest first | `sort -rn file.txt` |
| Remove duplicates | `sort -u file.txt` |
| Sort by column 2 | `sort -k2 file.txt` |
| Sort CSV by column 3 | `sort -t',' -k3 -n file.csv` |
| Sort disk usage | `du -sh * \| sort -hr` |
| Count and rank | `sort file \| uniq -c \| sort -rn` |
| Sort version numbers | `sort -V versions.txt` |

---

## 🚀 Full Workflow — Analyze Log Data

```bash
# Step 1: Sort log file alphabetically
sort /var/log/auth.log > sorted_auth.log

# Step 2: Find unique IP addresses that connected
awk '{print $11}' /var/log/auth.log | sort -u

# Step 3: Count and rank most common IPs
awk '{print $11}' /var/log/auth.log | sort | uniq -c | sort -rn | head -10

# Step 4: Find top 10 largest log files
du -sh /var/log/* 2>/dev/null | sort -hr | head -10

# Step 5: Sort error messages by frequency
grep -i "error" /var/log/syslog | awk '{print $6}' | sort | uniq -c | sort -rn | head -5
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Sorting numbers without `-n` | Always use `-n` for numerical data |
| Sorting human-readable sizes without `-h` | Use `sort -h` for K/M/G suffixes |
| `uniq` without sorting first | `uniq` only removes **adjacent** duplicates — always `sort` first |
| `-k` column counting is 1-based | First column is `-k1`, not `-k0` |
| Overwriting input file with `>` | Use `-o filename filename` to sort in place safely |

---

## 🔑 Key Takeaways

- `sort` by default sorts **alphabetically** — use `-n` for numbers to avoid incorrect ordering.
- `-r` reverses the sort order — combine with `-n` as `sort -rn` for largest-first numeric sorting.
- `-u` removes **duplicate lines** — cleaner than piping to `uniq` separately.
- `-k N` sorts by **column N** — combine with `-t` to specify a custom delimiter like `,` for CSV.
- `-h` handles **human-readable sizes** — essential for sorting `du -h` and `ls -lh` output.
- **`sort | uniq -c | sort -rn`** is the classic pattern for **counting and ranking** text occurrences.
- Use `-o` to sort a file **in place** — never use `sort file > file` as it truncates the file before sorting.

---

## 📚 Related Concepts

- [GNU sort Manual](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html)
- [GNU uniq Manual](https://www.gnu.org/software/coreutils/manual/html_node/uniq-invocation.html)

</details>
