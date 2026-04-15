# Challenge 40: Text Processing with sed in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`sed`** (**S**tream **Ed**itor) is a powerful text processing tool that reads input line by line, applies transformations, and writes the result. It is the standard tool for:

- Replacing text in files and command output
- Deleting specific lines
- Inserting or appending text
- Extracting portions of text
- Batch editing multiple files

Unlike a text editor, `sed` processes files **non-interactively** — perfect for scripts, automation, and processing large files.

---

## 🧩 Task

Replace a word or pattern in a file using `sed`.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Replace first occurrence on each line
sed 's/oldword/newword/' filename.txt

# Replace ALL occurrences on each line (global)
sed 's/oldword/newword/g' filename.txt

# Edit the file in place (modify the actual file)
sed -i 's/oldword/newword/g' filename.txt
```

### How it works

| Part | Description |
|------|-------------|
| `sed` | Stream editor — processes text line by line |
| `'s/old/new/'` | **Substitute** command — replace `old` with `new` |
| `s` | **s**ubstitute |
| `/old/` | Pattern to find |
| `/new/` | Replacement text |
| `/g` | **g**lobal — replace all occurrences per line |
| `-i` | **i**n-place — edit the file directly |

---

## 🔢 The `sed` Substitute Command Structure

```
s / pattern / replacement / flags
│   │         │             │
│   │         │             └── g=global, i=case-insensitive, p=print
│   │         └──────────────── What to replace with
│   └────────────────────────── What to find (regex supported)
└────────────────────────────── Substitute command
```

---

## 📖 Extended Examples

### Example 1 — Basic Word Replacement

```bash
echo "Hello World" | sed 's/World/Linux/'
```

```bash
Hello Linux
```

---

### Example 2 — Replace in a File (Print to Screen)

```bash
cat config.txt
# server=old-server.example.com
# port=8080
# debug=false

sed 's/old-server/new-server/' config.txt
```

```bash
server=new-server.example.com
port=8080
debug=false
```

> The original file is **not changed** — output goes to stdout. Use `-i` to edit in place.

---

### Example 3 — Replace All Occurrences on Each Line (`g` flag)

```bash
echo "cat and cat and cat" | sed 's/cat/dog/'
# cat and cat and cat → dog and cat and cat  (only first replaced)

echo "cat and cat and cat" | sed 's/cat/dog/g'
# dog and dog and dog  (all replaced)
```

---

### Example 4 — Edit File In Place (`-i`)

```bash
# Edit the file directly
sed -i 's/old-server/new-server/g' config.txt

# Edit in place with a backup (creates config.txt.bak)
sed -i.bak 's/old-server/new-server/g' config.txt

# Verify the change
cat config.txt
```

> 💡 Always use `-i.bak` (with backup extension) when editing important files — you can restore the original if something goes wrong.

---

### Example 5 — Case-Insensitive Replacement (`I` flag)

```bash
# Replace regardless of case (Error, error, ERROR all matched)
sed 's/error/WARNING/gI' logfile.txt
```

---

### Example 6 — Delete Lines

```bash
# Delete lines matching a pattern
sed '/^#/d' config.txt           # Delete comment lines

# Delete empty lines
sed '/^$/d' config.txt

# Delete a specific line number
sed '5d' filename.txt             # Delete line 5

# Delete a range of lines
sed '3,7d' filename.txt           # Delete lines 3 to 7

# Delete last line
sed '$d' filename.txt
```

---

### Example 7 — Print Specific Lines

```bash
# Print only lines matching a pattern
sed -n '/ERROR/p' logfile.txt

# Print only line 5
sed -n '5p' filename.txt

# Print lines 10 to 20
sed -n '10,20p' filename.txt

# Print lines from pattern to end
sed -n '/START/,/END/p' filename.txt
```

> 💡 `-n` suppresses default output — only lines explicitly selected with `p` are printed.

---

### Example 8 — Insert and Append Lines

```bash
# Insert a line BEFORE line 3
sed '3i\New line before line 3' filename.txt

# Append a line AFTER line 3
sed '3a\New line after line 3' filename.txt

# Insert before lines matching a pattern
sed '/pattern/i\Inserted before this line' filename.txt

# Append after lines matching a pattern
sed '/\[server\]/a\timeout=30' config.txt
```

---

### Example 9 — Multiple Commands with `-e`

```bash
# Run multiple sed commands at once
sed -e 's/foo/bar/g' -e 's/hello/world/g' filename.txt

# Or use semicolons
sed 's/foo/bar/g; s/hello/world/g' filename.txt

# Multiple substitutions in one sed call
sed -i \
  -e 's/DEBUG=false/DEBUG=true/g' \
  -e 's/PORT=8080/PORT=9090/g' \
  -e 's/HOST=localhost/HOST=0.0.0.0/g' \
  config.txt
```

---

### Example 10 — Use Different Delimiters

When the pattern contains `/`, use a different delimiter:

```bash
# Replace a file path — using | as delimiter
sed 's|/old/path|/new/path|g' script.sh

# Using # as delimiter
sed 's#/var/log#/data/logs#g' config.txt

# Any character can be the delimiter after s
sed 's:/old/path:/new/path:g' script.sh
```

---

### Example 11 — Address Ranges

```bash
# Apply substitution only on lines 5 to 10
sed '5,10s/foo/bar/g' filename.txt

# Apply only on lines matching a pattern
sed '/server/s/old/new/g' config.txt

# Apply from pattern to end of file
sed '/START/,$s/old/new/g' filename.txt

# Apply on even lines (every 2nd line starting from 2)
sed '0~2s/old/new/g' filename.txt
```

---

### Example 12 — Real-World: Update Configuration Files

```bash
#!/bin/bash
# Update multiple config values in a file

CONFIG="/etc/myapp/config.conf"

# Backup first
cp "$CONFIG" "${CONFIG}.bak"

# Apply multiple changes
sed -i \
  -e 's/^debug=.*/debug=false/' \
  -e 's/^port=.*/port=9090/' \
  -e 's/^host=.*/host=production.example.com/' \
  "$CONFIG"

echo "Configuration updated:"
grep -E "^debug|^port|^host" "$CONFIG"
```

```bash
# Output:
Configuration updated:
debug=false
port=9090
host=production.example.com
```

---

## 🗂️ `sed` Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `s/old/new/` | **Substitute** first occurrence |
| `s/old/new/g` | Substitute **all** occurrences |
| `s/old/new/gi` | Substitute all, **case-insensitive** |
| `s/old/new/2` | Substitute **2nd** occurrence only |
| `/pattern/d` | **Delete** lines matching pattern |
| `/^$/d` | Delete **empty** lines |
| `Nd` | Delete line **N** |
| `N,Md` | Delete lines **N to M** |
| `-n '/pattern/p'` | **Print** only matching lines |
| `-n 'N,Mp'` | Print lines **N to M** |
| `Ni\text` | **Insert** text before line N |
| `Na\text` | **Append** text after line N |
| `y/abc/xyz/` | **Transliterate** (replace chars) |
| `q` | **Quit** after first match |

---

## 🔄 `sed` vs `awk` vs `grep` — When to Use Each

| Tool | Best For |
|------|----------|
| `grep` | Finding lines that match a pattern |
| `sed` | Transforming text — substitute, delete, insert |
| `awk` | Processing structured columns — extract, calculate |

---

## 🚀 Full Workflow — Batch Update Configuration Files

```bash
# Step 1: Preview change (no -i flag — dry run)
sed 's/localhost/production.example.com/g' /etc/myapp/config.conf

# Step 2: Confirm it looks right, then apply with backup
sed -i.bak 's/localhost/production.example.com/g' /etc/myapp/config.conf

# Step 3: Verify the change
grep "production.example.com" /etc/myapp/config.conf

# Step 4: If something went wrong, restore backup
cp /etc/myapp/config.conf.bak /etc/myapp/config.conf

# Step 5: Apply to multiple files at once
find /etc/myapp/ -name "*.conf" -exec sed -i.bak 's/localhost/prod.example.com/g' {} \;
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Forgetting `g` — only first occurrence replaced | Add `/g` flag: `s/old/new/g` |
| `-i` with no backup | Always use `-i.bak` for important files |
| Pattern contains `/` | Use different delimiter: `s\|/old\|/new\|g` |
| Regex special chars in pattern | Escape them: `s/1\.0/2\.0/g` (dot needs escaping) |
| Editing piped output with `-i` | `-i` only works on files, not stdin — use without `-i` |
| macOS vs Linux `sed` differences | macOS requires `-i ''` for in-place: `sed -i '' 's/old/new/g'` |

---

## 🔑 Key Takeaways

- `sed 's/old/new/g'` replaces **all** occurrences — without `g`, only the first per line is replaced.
- Always **preview first** (without `-i`) before using `-i` to edit files in place.
- Use **`-i.bak`** to create a backup before in-place editing.
- Use **alternative delimiters** (`|`, `#`, `:`) when the pattern contains `/`.
- **`-n` with `p`** is the `sed` way to print only matching lines (like `grep`).
- **`^` and `$`** anchor patterns to the start and end of a line — use them for precise matching.
- `sed` supports **regex** — `.`, `*`, `^`, `$`, `[]`, `\+`, `\?` all work in patterns.

---

## 📚 Related Concepts

- [GNU sed Manual](https://www.gnu.org/software/sed/manual/sed.html)
- [sed One-Liners](https://www.pement.org/sed/sed1line.txt)

</details>
