# Challenge 51: Pass Arguments in Bash Script

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

## 📌 Overview

Bash scripts can accept **command-line arguments** passed by the user at runtime. This makes scripts dynamic and reusable without modifying the code itself.

---

## 🧩 Task

Write a script that accepts arguments from the command line and prints them.

---

<details>
<summary>💡 Click to view solution</summary>
  
## ✅ Solution

```bash
#!/bin/bash
echo "First argument: $1"
```

### How it works

| Variable | Description |
|----------|-------------|
| `$0` | The script name itself |
| `$1` | First argument passed to the script |
| `$2` | Second argument |
| `$N` | Nth argument |
| `$@` | All arguments as separate words |
| `$*` | All arguments as a single string |
| `$#` | Total number of arguments passed |

---

## 📖 Extended Examples

### Example 1 — Basic Single Argument

```bash
#!/bin/bash
echo "First argument: $1"
```

```bash
$ ./script.sh Hello
First argument: Hello
```

---

### Example 2 — Multiple Arguments

```bash
#!/bin/bash
echo "First argument : $1"
echo "Second argument: $2"
echo "Third argument : $3"
```

```bash
$ ./script.sh Apple Banana Cherry
First argument : Apple
Second argument: Banana
Third argument : Cherry
```

---

### Example 3 — Print All Arguments

```bash
#!/bin/bash
echo "Total arguments: $#"
echo "All arguments  : $@"
```

```bash
$ ./script.sh one two three
Total arguments: 3
All arguments  : one two three
```

---

### Example 4 — Loop Over All Arguments

```bash
#!/bin/bash
echo "Arguments received:"
for arg in "$@"; do
  echo "  - $arg"
done
```

```bash
$ ./script.sh red green blue
Arguments received:
  - red
  - green
  - blue
```

---

### Example 5 — Using Arguments with Default Values

```bash
#!/bin/bash
NAME=${1:-"World"}
echo "Hello, $NAME!"
```

```bash
$ ./script.sh Alice
Hello, Alice!

$ ./script.sh
Hello, World!
```

> 💡 `${1:-"default"}` uses the first argument if provided, otherwise falls back to the default value.

---

### Example 6 — Validating Arguments

```bash
#!/bin/bash
if [ $# -eq 0 ]; then
  echo "❌ Error: No arguments provided."
  echo "Usage: $0 <name> [greeting]"
  exit 1
fi

GREETING=${2:-"Hello"}
echo "$GREETING, $1!"
```

```bash
$ ./script.sh
❌ Error: No arguments provided.
Usage: ./script.sh <name> [greeting]

$ ./script.sh Alice Hi
Hi, Alice!
```

---

## 🚀 How to Run

```bash
# Step 1: Create the script file
touch script.sh

# Step 2: Make it executable
chmod +x script.sh

# Step 3: Run it with arguments
./script.sh arg1 arg2 arg3
```

---

## 🔑 Key Takeaways

- Arguments are accessed using **positional parameters**: `$1`, `$2`, `$3`, etc.
- `$#` gives the **count** of arguments — useful for validation.
- `$@` lets you **iterate** over all arguments cleanly.
- Use `${1:-default}` to set **default values** when an argument is omitted.
- Always validate input with `if [ $# -eq 0 ]` for robust scripts.

---

## 📚 Related Concepts

- [Special Variables in Bash](https://www.gnu.org/software/bash/manual/bash.html#Special-Parameters)
- [Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/bash.html#Bash-Conditional-Expressions)
- [Shift Command](https://www.gnu.org/software/bash/manual/bash.html#index-shift) — shifts positional parameters to the left

</details>
