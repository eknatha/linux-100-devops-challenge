# Challenge 39: Process Data with awk in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`awk`** is one of the most powerful text processing tools in Linux. Named after its creators (Aho, Weinberger, and Kernighan), it is a full programming language designed for processing structured text — log files, CSV data, command output, and configuration files.

While `grep` finds lines and `sed` substitutes text, `awk` operates on **fields** (columns) within each line — making it perfect for extracting specific columns, performing calculations, filtering rows, and generating formatted reports.

---

## 🧩 Task

Extract and print specific columns from a file or command output.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Print the first column of a file
awk '{print $1}' filename.txt

# Print multiple columns
awk '{print $1, $3}' filename.txt

# Print with a custom separator
awk -F: '{print $1}' /etc/passwd
```

### How it works

| Part | Description |
|------|-------------|
| `awk` | Text processing tool |
| `'{print $1}'` | **Action** — print the first field of each line |
| `$1`, `$2`... | Field (column) references — `$1` = first, `$2` = second |
| `$0` | The **entire line** |
| `$NF` | The **last field** on the line |
| `-F:` | **Field separator** — use `:` instead of whitespace |
| `filename.txt` | File to process |

---

## 🔢 How `awk` Processes Data

```
Input line: "Alice   Engineering   75000"
            ────────────────────────────
            $1=Alice  $2=Engineering  $3=75000
                                     $NF=75000 (last field)
            $0 = entire line

Default field separator: whitespace (spaces and tabs)
```

---

## 📖 Extended Examples

### Example 1 — Print First Column

```bash
# Print first column (username) from /etc/passwd
awk -F: '{print $1}' /etc/passwd
```

```bash
root
daemon
bin
sys
devuser
mysql
```

---

### Example 2 — Print Multiple Columns

```bash
cat employees.txt
# Alice  Engineering  75000
# Bob    Marketing    62000
# Carol  Engineering  85000

# Print name and salary
awk '{print $1, $3}' employees.txt
```

```bash
Alice 75000
Bob 62000
Carol 85000
```

---

### Example 3 — Custom Output Format

```bash
# Format output with labels
awk '{print "Name:", $1, "| Salary:", $3}' employees.txt
```

```bash
Name: Alice | Salary: 75000
Name: Bob | Salary: 62000
Name: Carol | Salary: 85000
```

```bash
# Use printf for aligned columns
awk '{printf "%-15s %-15s %s\n", $1, $2, $3}' employees.txt
```

```bash
Alice           Engineering     75000
Bob             Marketing       62000
Carol           Engineering     85000
```

---

### Example 4 — Custom Field Separator with `-F`

```bash
# Print usernames from /etc/passwd (colon-separated)
awk -F: '{print $1}' /etc/passwd

# Print IP addresses from a log
awk -F'"' '{print $2}' /var/log/nginx/access.log

# CSV file with comma separator
awk -F',' '{print $1, $3}' data.csv

# Use tab as separator
awk -F'\t' '{print $2}' data.tsv

# Multi-character separator
awk -F'::' '{print $1}' file.txt
```

---

### Example 5 — Filter Rows with Conditions

```bash
# Print only rows where salary ($3) is above 70000
awk '$3 > 70000 {print $0}' employees.txt

# Print rows where department ($2) is "Engineering"
awk '$2 == "Engineering" {print $1, $3}' employees.txt

# Print rows where name starts with "A"
awk '$1 ~ /^A/ {print $0}' employees.txt

# Print rows matching a pattern (like grep)
awk '/Engineering/ {print $0}' employees.txt
```

```bash
# $3 > 70000 output:
Alice  Engineering  75000
Carol  Engineering  85000
```

---

### Example 6 — Special Variables

```bash
# NR — current line number
awk '{print NR, $1}' employees.txt

# NF — number of fields on current line
awk '{print "Fields:", NF, "| Last:", $NF}' employees.txt

# Skip the header line (NR > 1)
awk 'NR > 1 {print $1, $3}' employees.txt

# Print only lines 3 to 7
awk 'NR >= 3 && NR <= 7 {print}' filename.txt
```

```bash
# NR output:
1 Alice
2 Bob
3 Carol
```

| Variable | Description |
|----------|-------------|
| `NR` | **Current line number** (Number of Records) |
| `NF` | **Number of fields** on the current line |
| `$0` | **Entire line** |
| `$1`..`$NF` | Field references |
| `FS` | **Field separator** (same as `-F`) |
| `RS` | **Record separator** (default: newline) |
| `OFS` | **Output field separator** |
| `ORS` | **Output record separator** |

---

### Example 7 — Perform Calculations

```bash
# Sum all values in column 3
awk '{sum += $3} END {print "Total:", sum}' employees.txt

# Calculate average
awk '{sum += $3; count++} END {print "Average:", sum/count}' employees.txt

# Find maximum
awk 'BEGIN {max=0} $3 > max {max=$3} END {print "Max:", max}' employees.txt

# Count lines matching a condition
awk '$3 > 70000 {count++} END {print "High earners:", count}' employees.txt
```

```bash
# Sum output:
Total: 222000

# Average output:
Average: 74000
```

---

### Example 8 — BEGIN and END Blocks

```bash
# BEGIN: runs before processing any lines
# END: runs after processing all lines

awk '
BEGIN {
  print "=== Employee Report ==="
  print "Name\t\tDept\t\tSalary"
  print "────────────────────────────────"
}
{
  printf "%-15s %-15s %s\n", $1, $2, $3
}
END {
  print "────────────────────────────────"
  print "Total employees:", NR
}
' employees.txt
```

```bash
=== Employee Report ===
Name            Dept            Salary
────────────────────────────────
Alice           Engineering     75000
Bob             Marketing       62000
Carol           Engineering     85000
────────────────────────────────
Total employees: 3
```

---

### Example 9 — Process Command Output with `awk`

```bash
# Extract usernames of processes
ps aux | awk '{print $1}' | sort -u

# Get disk usage percentage only
df -h | awk '{print $5, $6}' | grep -v Use

# Get IPs from network interface
ip addr | awk '/inet / {print $2}'

# Get the 5th column of ps output
ps aux | awk '{print $11}'

# Extract field from /proc/meminfo
awk '/MemAvailable/ {print $2, $3}' /proc/meminfo
```

---

### Example 10 — String Operations

```bash
# Convert field to uppercase
awk '{print toupper($1)}' employees.txt

# Convert to lowercase
awk '{print tolower($2)}' employees.txt

# Get string length
awk '{print $1, length($1)}' employees.txt

# Concatenate fields
awk '{print $1 "_" $2}' employees.txt

# Substring
awk '{print substr($1, 1, 3)}' employees.txt   # First 3 chars
```

```bash
# toupper output:
ALICE Engineering 75000
BOB Marketing 62000
CAROL Engineering 85000
```

---

### Example 11 — Multiple Conditions and Logic

```bash
# AND condition
awk '$2 == "Engineering" && $3 > 70000 {print $1}' employees.txt

# OR condition
awk '$2 == "Engineering" || $3 > 80000 {print $1, $3}' employees.txt

# NOT condition
awk '$2 != "Marketing" {print $1}' employees.txt

# if-else inside awk
awk '{
  if ($3 > 80000) print $1, "Senior"
  else if ($3 > 70000) print $1, "Mid"
  else print $1, "Junior"
}' employees.txt
```

```bash
# if-else output:
Alice Mid
Bob Junior
Carol Senior
```

---

### Example 12 — Real-World: Parse Log Files

```bash
# Count HTTP status codes in nginx access log
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Find top 10 IP addresses making requests
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Show only 404 errors
awk '$9 == 404 {print $1, $7}' /var/log/nginx/access.log

# Calculate average response size
awk '{sum += $10; count++} END {print "Avg response size:", sum/count, "bytes"}' /var/log/nginx/access.log

# Find lines where response time > 1 second
awk '$NF > 1.0 {print $0}' /var/log/nginx/access.log
```

---

## 🗂️ `awk` Quick Reference

### Field References

| Expression | Meaning |
|-----------|---------|
| `$1` | First field |
| `$2` | Second field |
| `$NF` | Last field |
| `$(NF-1)` | Second-to-last field |
| `$0` | Entire line |

### Common Patterns

| Goal | `awk` Command |
|------|--------------|
| Print column 1 | `awk '{print $1}'` |
| Print columns 1 and 3 | `awk '{print $1, $3}'` |
| Print last column | `awk '{print $NF}'` |
| Filter rows | `awk '$2 == "value" {print}'` |
| Sum a column | `awk '{sum += $3} END {print sum}'` |
| Count lines | `awk 'END {print NR}'` |
| Skip header | `awk 'NR > 1 {print}'` |
| CSV with commas | `awk -F',' '{print $1}'` |
| Custom separator | `awk -F: '{print $1}'` |
| Print with format | `awk '{printf "%-10s %s\n", $1, $2}'` |

---

## 🚀 Full Workflow — Analyze a Log File

```bash
# Step 1: Preview the log format
head -3 /var/log/nginx/access.log

# Step 2: Extract specific columns
awk '{print $1, $7, $9}' /var/log/nginx/access.log | head -10

# Step 3: Filter for errors (5xx status codes)
awk '$9 >= 500 {print $0}' /var/log/nginx/access.log

# Step 4: Count status codes
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Step 5: Calculate total bytes served
awk '{sum += $10} END {printf "Total: %.2f GB\n", sum/1073741824}' /var/log/nginx/access.log

# Step 6: Find top 5 busiest IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -5

# Step 7: Save report
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn > status_report.txt
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `$1` splits on any whitespace by default | Use `-F','` for CSV or `-F':'` for colon-delimited files |
| Arithmetic with strings returns 0 | Ensure the field contains numbers; use `+0` to force numeric |
| `print $1 $2` runs them together | Use `print $1, $2` (comma adds OFS separator) |
| awk single quotes conflict with shell | Put awk program in a file: `awk -f program.awk file` |
| `NR` vs line count | `NR` includes all input lines; for `wc -l` equivalent use `END {print NR}` |

---

## 🔑 Key Takeaways

- `awk '{print $1}'` prints the **first column** — `$1`, `$2`... reference fields by number.
- `$NF` is always the **last field**; `$0` is the **entire line**.
- Use `-F` to set the **field separator** — `-F:` for `/etc/passwd`, `-F,` for CSV files.
- `NR` is the **current line number** — use `NR > 1` to skip headers.
- `BEGIN {}` runs **before** any lines; `END {}` runs **after** all lines — use for totals and reports.
- `awk` can do **arithmetic** — sum, average, max — across all lines in a file.
- The `sort | uniq -c | sort -rn` pattern with `awk` is the classic recipe for **counting and ranking** any field.

---

## 📚 Related Concepts

- [GNU awk Manual](https://www.gnu.org/software/gawk/manual/)
- [awk One-Liners](https://www.pement.org/awk/awk1line.txt)

</details>
