# Challenge 53: Conditional Statements in Bash

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

Conditional statements in Bash let your script **make decisions** based on conditions — checking whether files exist, comparing numbers, evaluating strings, and more. The primary construct is the `if` statement, which can be extended with `elif` and `else` for branching logic.

---

## 🧩 Task

Check if a file exists and print an appropriate message.

---

<details>
<summary>💡 Click to view solution</summary>
  
## ✅ Solution

```bash
#!/bin/bash
if [ -f file.txt ]; then
  echo "File exists"
fi
```

```bash
$ ./script.sh
File exists
```

### How it works

| Part | Description |
|------|-------------|
| `if` | Starts the conditional block |
| `[ -f file.txt ]` | Test expression — checks if `file.txt` exists and is a regular file |
| `-f` | File test flag — true if the path exists and is a regular file |
| `then` | Marks the start of the block to execute when condition is true |
| `echo "File exists"` | Command that runs when the condition is met |
| `fi` | Closes the `if` block (`fi` is `if` spelled backwards) |

---

## 📖 Extended Examples

### Example 1 — Basic File Check

```bash
#!/bin/bash
if [ -f file.txt ]; then
  echo "File exists"
fi
```

```bash
$ ./script.sh
File exists
```

---

### Example 2 — `if / else` — File Exists or Not

```bash
#!/bin/bash
if [ -f file.txt ]; then
  echo "✅ file.txt exists"
else
  echo "❌ file.txt does not exist"
fi
```

```bash
# When file exists:
✅ file.txt exists

# When file is missing:
❌ file.txt does not exist
```

---

### Example 3 — `if / elif / else` — Multiple Branches

```bash
#!/bin/bash
FILE="data"

if [ -f "$FILE" ]; then
  echo "$FILE is a regular file"
elif [ -d "$FILE" ]; then
  echo "$FILE is a directory"
else
  echo "$FILE does not exist"
fi
```

```bash
data is a directory
```

---

### Example 4 — Comparing Numbers

```bash
#!/bin/bash
SCORE=85

if [ $SCORE -ge 90 ]; then
  echo "Grade: A"
elif [ $SCORE -ge 75 ]; then
  echo "Grade: B"
elif [ $SCORE -ge 60 ]; then
  echo "Grade: C"
else
  echo "Grade: F"
fi
```

```bash
Grade: B
```

---

### Example 5 — Comparing Strings

```bash
#!/bin/bash
NAME="Alice"

if [ "$NAME" = "Alice" ]; then
  echo "Hello, Alice! 👋"
elif [ "$NAME" = "Bob" ]; then
  echo "Hey, Bob!"
else
  echo "Who are you?"
fi
```

```bash
Hello, Alice! 👋
```

> 💡 Always **quote string variables** (`"$NAME"`) to avoid errors when the value is empty.

---

### Example 6 — Check if a Directory Exists

```bash
#!/bin/bash
DIR="./logs"

if [ -d "$DIR" ]; then
  echo "Directory '$DIR' exists"
else
  echo "Creating directory '$DIR'..."
  mkdir "$DIR"
  echo "Done!"
fi
```

```bash
Creating directory './logs'...
Done!
```

---

### Example 7 — Check if a Variable is Empty

```bash
#!/bin/bash
INPUT=""

if [ -z "$INPUT" ]; then
  echo "❌ Input is empty"
else
  echo "✅ Input: $INPUT"
fi
```

```bash
❌ Input is empty
```

---

### Example 8 — Combining Conditions with `&&` and `||`

```bash
#!/bin/bash
AGE=20
HAS_ID=true

if [ $AGE -ge 18 ] && [ "$HAS_ID" = "true" ]; then
  echo "✅ Access granted"
else
  echo "❌ Access denied"
fi
```

```bash
✅ Access granted
```

---

### Example 9 — Negating a Condition with `!`

```bash
#!/bin/bash
if [ ! -f "config.txt" ]; then
  echo "Config file not found. Creating default config..."
  echo "theme=dark" > config.txt
fi
```

```bash
Config file not found. Creating default config...
```

> 💡 `!` negates the condition — the block runs when the test is **false**.

---

### Example 10 — One-liner with `&&` and `||`

```bash
#!/bin/bash
[ -f file.txt ] && echo "Exists" || echo "Not found"
```

```bash
# If file.txt exists:
Exists

# If file.txt is missing:
Not found
```

---

## 🗂️ File Test Flags — Quick Reference

| Flag | True When |
|------|-----------|
| `-f file` | File exists and is a regular file |
| `-d file` | File exists and is a directory |
| `-e file` | File exists (any type) |
| `-r file` | File exists and is readable |
| `-w file` | File exists and is writable |
| `-x file` | File exists and is executable |
| `-s file` | File exists and is not empty |
| `-L file` | File exists and is a symbolic link |

---

## 🔢 Numeric Comparison Operators

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |

---

## 🔤 String Comparison Operators

| Operator | Meaning |
|----------|---------|
| `=` | Strings are equal |
| `!=` | Strings are not equal |
| `-z "$s"` | String is empty |
| `-n "$s"` | String is not empty |

---

## 🚀 How to Run

```bash
# Step 1: Create a test file (optional)
touch file.txt

# Step 2: Create the script
touch script.sh

# Step 3: Make it executable
chmod +x script.sh

# Step 4: Run it
./script.sh
```

---

## 🔑 Key Takeaways

- `[ ]` is the **test command** — always include spaces inside the brackets.
- Use `-f` to check for files, `-d` for directories, `-e` for either.
- Use `-z` / `-n` to test whether a string is **empty or not**.
- Always **quote variables** inside conditions to handle empty values safely.
- `!` negates a condition; `&&` combines conditions (both must be true); `||` means either must be true.
- `fi` closes every `if` block — it's simply `if` reversed.

---

## 📚 Related Concepts

- [Bash Conditional Constructs](https://www.gnu.org/software/bash/manual/bash.html#Conditional-Constructs)
- [Bash File Test Operators](https://www.gnu.org/software/bash/manual/bash.html#Bash-Conditional-Expressions)
- [Bash String Comparison](https://www.gnu.org/software/bash/manual/bash.html#index-test)

</details>
