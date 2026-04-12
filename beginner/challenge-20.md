# Challenge 20: Command History in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

The **command history** feature in Linux records every command you type in the terminal, allowing you to revisit, reuse, and search previous commands without retyping them. This dramatically speeds up your workflow — especially when dealing with long, complex commands you need to run repeatedly.

History is stored in the **`~/.bash_history`** file (for bash) and is managed by the `history` built-in command. Linux saves hundreds or thousands of commands by default, making it a powerful productivity and audit tool.

---

## 🧩 Task

View previously executed commands and reuse them efficiently.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# View full command history
history

# View last 20 commands
history 20

# Search history interactively
Ctrl + R
```

### How it works

| Command / Shortcut | Description |
|--------------------|-------------|
| `history` | Display all saved command history |
| `history N` | Display last **N** commands |
| `Ctrl + R` | **Reverse search** — search history interactively |
| `!!` | Repeat the **last** command |
| `!N` | Run command number **N** from history |

---

## 🔢 Understanding `history` Output

```bash
history
```

```
  498  ls -la /etc/
  499  sudo systemctl status nginx
  500  grep -i error /var/log/syslog
  501  df -h
  502  free -h
  503  history
```

| Column | Description |
|--------|-------------|
| Number | Command **sequence number** in history |
| Command | The actual command that was run |

> 💡 The number on the left lets you re-run any command with `!N` — for example `!500` reruns `grep -i error /var/log/syslog`.

---

## 📖 Extended Examples

### Example 1 — View Full History

```bash
history
```

---

### Example 2 — View Last N Commands

```bash
# Last 10 commands
history 10

# Last 20 commands
history 20

# Last 50 commands
history 50
```

---

### Example 3 — Search History with `Ctrl + R`

Press `Ctrl + R` to enter reverse-search mode:

```bash
(reverse-i-search)`nginx': sudo systemctl restart nginx
```

| Action | Description |
|--------|-------------|
| `Ctrl + R` | Open reverse search |
| Type characters | Narrows the search |
| `Ctrl + R` again | Go to **previous** match |
| `Enter` | **Run** the found command |
| `Ctrl + G` | **Cancel** search |
| `→` or `End` | **Edit** the found command before running |

> 💡 `Ctrl + R` is the **fastest way** to find and rerun a previous command — type just a few characters and it finds the most recent match.

---

### Example 4 — Rerun Previous Commands

```bash
# Rerun the very last command
!!

# Rerun command number 500 from history
!500

# Rerun the last command starting with "sudo"
!sudo

# Rerun the last command starting with "git"
!git
```

```bash
# Example:
$ sudo systemctl restart nginx
# ... runs command

$ !!
sudo systemctl restart nginx    ← repeats it exactly
```

---

### Example 5 — Search History with `grep`

```bash
# Find all commands containing "nginx"
history | grep nginx

# Find all sudo commands
history | grep sudo

# Find all git commands
history | grep "^[0-9]* *git"

# Find commands with a specific file
history | grep "/etc/ssh"
```

```bash
# history | grep nginx output:
  499  sudo systemctl status nginx
  512  sudo nginx -t
  518  sudo systemctl restart nginx
```

---

### Example 6 — History Expansion — Reuse Arguments

```bash
# Run ls on a path, then cd into it using !! tricks
ls /var/log/nginx/
cd !$           # !$ = last argument of previous command
# cd /var/log/nginx/

# Use the entire previous command as argument
echo !!
# Prints the last command

# Get specific argument from last command
ls -la /etc/nginx/nginx.conf
vi !:2          # !:2 = second argument = /etc/nginx/nginx.conf
```

| Expansion | Description |
|-----------|-------------|
| `!!` | Entire last command |
| `!$` | Last **argument** of last command |
| `!^` | First **argument** of last command |
| `!:N` | Nth argument of last command |
| `!*` | All arguments of last command |

---

### Example 7 — History File Location and Size

```bash
# View the history file directly
cat ~/.bash_history

# Check how many commands are stored
wc -l ~/.bash_history

# Check history size settings
echo $HISTSIZE        # Max commands in memory
echo $HISTFILESIZE    # Max commands saved to file
```

```bash
# Default values:
HISTSIZE=1000         # Keep 1000 commands in memory
HISTFILESIZE=2000     # Save 2000 commands to file
```

---

### Example 8 — Configure History Behavior

Add these to `~/.bashrc` for a better history experience:

```bash
# ~/.bashrc — history configuration

# Increase history size
HISTSIZE=10000
HISTFILESIZE=20000

# Add timestamps to history
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  "

# Ignore duplicate commands and commands starting with space
HISTCONTROL=ignoreboth

# Append to history instead of overwriting
shopt -s histappend

# Ignore common commands from history
HISTIGNORE="ls:ll:cd:pwd:clear:history"
```

```bash
# Apply changes
source ~/.bashrc

# Now history shows timestamps:
history 5
```

```bash
  500  2025-03-21 10:00:01  sudo systemctl restart nginx
  501  2025-03-21 10:01:15  df -h
  502  2025-03-21 10:02:30  free -h
```

---

### Example 9 — Delete Specific History Entries

```bash
# Delete a specific history entry by number
history -d 500

# Delete the last command from history
history -d $(history | tail -2 | head -1 | awk '{print $1}')

# Clear entire history (current session)
history -c

# Clear history and delete the file
history -c && rm ~/.bash_history
```

> ⚠️ Use `history -c` carefully — it removes all history from the current session permanently.

---

### Example 10 — Prevent Sensitive Commands from Being Saved

```bash
# Start a command with a SPACE to exclude it from history
# (requires HISTCONTROL=ignorespace or ignoreboth)
 export API_KEY="secret123"    ← leading space prevents saving

# Verify it was not saved
history | tail -5
# The command with the leading space does not appear
```

---

### Example 11 — Share History Across Multiple Terminals

By default, each terminal has its own history. This syncs it:

```bash
# Add to ~/.bashrc to sync history across terminals
export PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"
```

| Command | Action |
|---------|--------|
| `history -a` | **Append** current session to history file |
| `history -c` | **Clear** current session's history |
| `history -r` | **Read** history file into current session |

---

### Example 12 — Use `fzf` for Fuzzy History Search

```bash
# Install fzf (fuzzy finder)
sudo apt install fzf    # Debian/Ubuntu
sudo dnf install fzf    # RHEL

# Fuzzy search history (much more powerful than Ctrl+R)
history | fzf

# Bind fzf to Ctrl+R for enhanced history search
# Add to ~/.bashrc:
source /usr/share/doc/fzf/examples/key-bindings.bash
```

> `fzf` replaces the basic `Ctrl+R` with a full interactive fuzzy finder — type partial words in any order to find commands.

---

## 🗂️ History Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `history` | Show all history |
| `history N` | Show last N commands |
| `history -c` | Clear current session history |
| `history -d N` | Delete entry number N |
| `history -a` | Append session to history file |
| `history -r` | Read history file into session |
| `history -w` | Write current history to file |
| `!!` | Run last command |
| `!N` | Run command number N |
| `!string` | Run last command starting with string |
| `!$` | Last argument of previous command |
| `Ctrl + R` | Reverse interactive search |
| `Ctrl + G` | Cancel reverse search |

---

## 🗂️ History Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `HISTSIZE` | 1000 | Commands kept in memory |
| `HISTFILESIZE` | 2000 | Commands saved to file |
| `HISTFILE` | `~/.bash_history` | History file location |
| `HISTTIMEFORMAT` | (unset) | Timestamp format for history |
| `HISTCONTROL` | (unset) | `ignoredups`, `ignorespace`, `ignoreboth` |
| `HISTIGNORE` | (unset) | Colon-separated patterns to ignore |

---

## 🚀 Full Workflow — Find and Reuse a Previous Command

```bash
# Step 1: View recent history
history 20

# Step 2: Search for a specific command
history | grep "docker"

# Step 3: Use Ctrl+R to search interactively
# Press Ctrl+R and type "docker"

# Step 4: Rerun by number
!502

# Step 5: Edit before running (press → or End to edit)
# Ctrl+R finds: docker run -p 8080:80 nginx
# Press → to edit: docker run -p 9090:80 nginx

# Step 6: Reuse last argument
ls /var/log/nginx/
cd !$    # Navigate to /var/log/nginx/

# Step 7: Check timestamps (if configured)
HISTTIMEFORMAT="%F %T " history 10
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| History not saved between sessions | Check `shopt histappend` and `HISTFILESIZE` |
| Sensitive commands saved to history | Use a leading space or `HISTIGNORE` |
| History from other terminals not visible | Use `PROMPT_COMMAND` to sync across terminals |
| `!!` runs unexpected command | Run `history 5` first to confirm what `!!` will execute |
| History cleared on logout | Check `HISTFILESIZE` — set to a higher value |
| Duplicate commands cluttering history | Set `HISTCONTROL=ignoredups` |

---

## 🔑 Key Takeaways

- `history` shows all saved commands; `history N` shows the last N.
- **`Ctrl + R`** is the fastest way to find and reuse a previous command — type a few characters to search.
- `!!` repeats the last command; `!N` runs command number N; `!string` runs the last command starting with `string`.
- **`!$`** reuses the last argument of the previous command — extremely useful for navigating and editing the same path.
- Set `HISTTIMEFORMAT` in `~/.bashrc` to add **timestamps** to history — essential for security auditing.
- Use a **leading space** before a command to prevent it from being saved to history — great for passwords and secrets.
- Increase `HISTSIZE` and `HISTFILESIZE` in `~/.bashrc` for a longer, more useful history.

---

## 📚 Related Concepts

- [GNU Bash History Manual](https://www.gnu.org/software/bash/manual/bash.html#Using-History-Interactively)
- [fzf — Fuzzy Finder](https://github.com/junegunn/fzf)

</details>
