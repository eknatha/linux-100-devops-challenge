# Challenge 25: Environment Variables in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**Environment variables** are named values stored in the shell's environment that programs and scripts can read. They are used to configure application behavior, store system information, and pass data between processes — all without hardcoding values into code.

Common uses:
- Store database credentials and API keys
- Set the default text editor (`EDITOR`)
- Configure the command search path (`PATH`)
- Define the user's home directory (`HOME`)
- Pass configuration to applications (`DEBUG=true`)

---

## 🧩 Task

Set an environment variable and use it in the shell and scripts.

---

<details>
<summary>💡 Click to view solution</summary>


## ✅ Solution

```bash
# Set a variable (shell variable — current session only)
MYVAR="Hello, World!"

# Export it as an environment variable (available to child processes)
export MYVAR="Hello, World!"

# Use the variable
echo $MYVAR
```

### How it works

| Command | Description |
|---------|-------------|
| `MYVAR="value"` | Create a **shell variable** (current shell only) |
| `export MYVAR` | **Export** — makes it available to child processes |
| `export MYVAR="value"` | Set and export in one step |
| `echo $MYVAR` | **Print** the variable's value |
| `unset MYVAR` | **Remove** the variable |

---

## 🔢 Shell Variable vs Environment Variable

```
Shell Variable (MYVAR="value")
├── Visible in current shell ✅
├── NOT visible in child processes ❌
└── NOT visible in scripts called from this shell ❌

Environment Variable (export MYVAR="value")
├── Visible in current shell ✅
├── Visible in child processes ✅
└── Visible in scripts called from this shell ✅
```

---

## 📖 Extended Examples

### Example 1 — Set and Use a Variable

```bash
# Set a variable
NAME="Alice"

# Use it
echo "Hello, $NAME"
echo "Hello, ${NAME}!"   # Braces for clarity
```

```bash
Hello, Alice
Hello, Alice!
```

---

### Example 2 — Export as Environment Variable

```bash
# Export so child processes can see it
export DATABASE_URL="postgresql://localhost/mydb"

# Verify it is set
echo $DATABASE_URL

# Show in environment
env | grep DATABASE_URL
```

```bash
postgresql://localhost/mydb
DATABASE_URL=postgresql://localhost/mydb
```

---

### Example 3 — View All Environment Variables

```bash
# Show all environment variables
env

# Or using printenv
printenv

# Show a specific variable
printenv HOME
printenv PATH

# Show all with grep filter
env | grep -i user
```

```bash
# env output (partial):
HOME=/home/devuser
USER=devuser
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
SHELL=/bin/bash
LANG=en_US.UTF-8
```

---

### Example 4 — Important Built-in Environment Variables

```bash
echo $HOME      # Home directory of current user
echo $USER      # Current username
echo $PATH      # Directories searched for commands
echo $SHELL     # Current shell
echo $PWD       # Current working directory
echo $EDITOR    # Default text editor
echo $LANG      # System language/locale
echo $TERM      # Terminal type
echo $PS1       # Shell prompt format
echo $?         # Exit code of last command (0 = success)
echo $$         # PID of current shell
```

| Variable | Example Value | Description |
|----------|--------------|-------------|
| `$HOME` | `/home/devuser` | User's home directory |
| `$USER` | `devuser` | Current username |
| `$PATH` | `/usr/bin:/bin:/usr/sbin` | Command search path |
| `$SHELL` | `/bin/bash` | Default shell |
| `$PWD` | `/home/devuser/projects` | Current directory |
| `$EDITOR` | `vim` | Default text editor |
| `$?` | `0` | Last command exit code |
| `$$` | `4821` | Current shell's PID |

---

### Example 5 — Modify the `PATH` Variable

```bash
# Add a directory to PATH (temporarily)
export PATH="$PATH:/opt/myapp/bin"

# Verify
echo $PATH

# Add permanently (add to ~/.bashrc)
echo 'export PATH="$PATH:/opt/myapp/bin"' >> ~/.bashrc
source ~/.bashrc
```

```bash
# PATH before:
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# PATH after:
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/myapp/bin
```

---

### Example 6 — Make Variables Permanent

Variables set with `export` are **lost when the session ends**. To make them permanent, add them to shell startup files:

```bash
# For the current user — add to ~/.bashrc
echo 'export MYVAR="Hello, World!"' >> ~/.bashrc

# Apply immediately
source ~/.bashrc
# or
. ~/.bashrc

# For all users — add to /etc/environment (no export needed)
sudo echo 'MYVAR="Hello, World!"' >> /etc/environment
```

| File | Scope | When Loaded |
|------|-------|-------------|
| `~/.bashrc` | Current user | Interactive bash shells |
| `~/.bash_profile` | Current user | Login shells |
| `~/.profile` | Current user | Login shells (sh-compatible) |
| `/etc/environment` | All users | System-wide, all shells |
| `/etc/profile` | All users | Login shells |
| `/etc/profile.d/*.sh` | All users | Login shells (modular) |

---

### Example 7 — Unset a Variable

```bash
# Remove a variable from the environment
unset MYVAR

# Verify it is gone
echo $MYVAR    # Prints nothing
env | grep MYVAR  # No output
```

---

### Example 8 — Use Variables in Scripts

```bash
#!/bin/bash

# Variables defined in script
APP_NAME="MyApp"
APP_VERSION="1.0"
LOG_FILE="/var/log/${APP_NAME}.log"

echo "Starting $APP_NAME version $APP_VERSION"
echo "Logs: $LOG_FILE"

# Use environment variables passed from outside
echo "Database: ${DATABASE_URL:-not set}"    # With default value
echo "Debug mode: ${DEBUG:-false}"
```

```bash
# Run with inline environment variable
DATABASE_URL="postgresql://localhost/mydb" ./script.sh
```

---

### Example 9 — Default Values for Variables

```bash
# ${VAR:-default} — use default if VAR is unset or empty
echo ${DATABASE_URL:-"postgresql://localhost/default"}

# ${VAR:=default} — set VAR to default if unset, then use it
echo ${CONFIG_FILE:="/etc/myapp/config.conf"}

# ${VAR:?error message} — exit with error if VAR is unset
echo ${API_KEY:?"API_KEY must be set"}
```

```bash
# Useful in scripts for required variables
#!/bin/bash
: "${DATABASE_URL:?ERROR: DATABASE_URL is required}"
: "${API_KEY:?ERROR: API_KEY is required}"
echo "All required variables are set"
```

---

### Example 10 — Set for a Single Command (Inline)

```bash
# Set variable only for this one command
DEBUG=true ./myscript.sh

# Multiple inline variables
DATABASE_URL="postgres://localhost/db" DEBUG=true ./myscript.sh

# Set for a command without exporting
env DATABASE_URL="postgres://localhost/db" python3 app.py
```

> 💡 Inline variables are only visible to that single command — they do not affect the current shell or other commands.

---

### Example 11 — Store Secrets Safely

```bash
# BAD: visible in process list and shell history
export DB_PASSWORD="mysecretpassword"

# BETTER: load from a file
export DB_PASSWORD=$(cat /etc/myapp/db_password)

# BEST: use a .env file and load it
# .env file:
# DB_PASSWORD=mysecretpassword
# API_KEY=abc123

# Load .env in a script
set -a    # Automatically export all variables
source .env
set +a    # Stop auto-exporting

# Or use export with command substitution
export $(grep -v '^#' .env | xargs)
```

---

### Example 12 — Use `envsubst` to Template Files

```bash
# Template file: config.template
# server_name = ${APP_HOST}
# port = ${APP_PORT}

# Set variables
export APP_HOST="myserver.com"
export APP_PORT="8080"

# Substitute and create config
envsubst < config.template > config.conf
cat config.conf
```

```bash
# Output:
server_name = myserver.com
port = 8080
```

---

## 🗂️ Variable Operations — Quick Reference

| Syntax | Description |
|--------|-------------|
| `VAR=value` | Set a shell variable |
| `export VAR=value` | Set and export as environment variable |
| `echo $VAR` | Print variable value |
| `printenv VAR` | Print environment variable |
| `unset VAR` | Remove variable |
| `env` | List all environment variables |
| `${VAR:-default}` | Use default if VAR is unset |
| `${VAR:=default}` | Set to default if unset |
| `${VAR:?message}` | Error if VAR is unset |
| `${#VAR}` | Length of VAR's value |
| `${VAR^^}` | Uppercase value |
| `${VAR,,}` | Lowercase value |

---

## 🚀 Full Workflow — Configure an Application with Environment Variables

```bash
# Step 1: Create a .env file with your settings
cat > .env << EOF
APP_NAME=MyWebApp
APP_PORT=8080
DATABASE_URL=postgresql://localhost/mydb
API_KEY=your_api_key_here
DEBUG=false
EOF

# Step 2: Load the .env file
export $(grep -v '^#' .env | xargs)

# Step 3: Verify variables are set
env | grep -E "APP_|DATABASE_|API_|DEBUG"

# Step 4: Run your application
./myapp

# Step 5: Make permanent by adding to ~/.bashrc
echo 'export APP_PORT=8080' >> ~/.bashrc
echo 'export DATABASE_URL=postgresql://localhost/mydb' >> ~/.bashrc
source ~/.bashrc
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Spaces around `=` break assignment | `VAR=value` ✅ — `VAR = value` ❌ |
| Variable set but child process can't see it | Use `export VAR` — shell variables are not automatically exported |
| Variable persists only for current session | Add `export VAR=value` to `~/.bashrc` for persistence |
| Secrets in shell history | Use leading space or load from file — avoid `export PASSWORD=secret` |
| Quoting issues with spaces | Always quote: `VAR="value with spaces"` |
| `$VAR` vs `${VAR}` | Use `${VAR}` when variable is followed by text: `echo "${HOME}dir"` |

---

## 🔑 Key Takeaways

- `VAR=value` creates a **shell variable** — `export VAR` makes it an **environment variable** visible to child processes.
- Always `export` variables that scripts or programs need to read.
- Variables set in a session are **lost when the session ends** — add to `~/.bashrc` for persistence.
- `${VAR:-default}` provides a **default value** if the variable is not set — essential for robust scripts.
- **Never put secrets** (passwords, API keys) directly in shell history — load them from files.
- Use `env | grep VARNAME` or `printenv VARNAME` to verify a variable is set and exported.
- `source .env` or `export $(grep -v '^#' .env | xargs)` loads a `.env` file into the current shell.

---

## 📚 Related Concepts

- [GNU Bash Variables Manual](https://www.gnu.org/software/bash/manual/bash.html#Shell-Variables)
- [Linux Environment Variables Guide](https://www.gnu.org/software/bash/manual/bash.html#Environment)

</details>
