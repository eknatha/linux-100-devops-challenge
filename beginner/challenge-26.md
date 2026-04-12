# Challenge 26: Create Aliases in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-brightgreen)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

An **alias** is a custom shortcut for a command or a series of commands. Instead of typing a long command repeatedly, you define a short memorable name for it. Aliases make your workflow faster, reduce typos, and let you apply your preferred flags automatically — without changing any system files.

Common uses:
- Shorten long commands (`ll` instead of `ls -la`)
- Add safety flags automatically (`rm -i` instead of `rm`)
- Create memorable shortcuts for complex commands
- Override default commands with improved versions

---

## 🧩 Task

Create a shortcut command using `alias`.

---

<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Create a temporary alias (current session only)
alias ll='ls -la'

# Use the alias
ll

# Make it permanent (add to ~/.bashrc)
echo "alias ll='ls -la'" >> ~/.bashrc
source ~/.bashrc
```

### How it works

| Part | Description |
|------|-------------|
| `alias` | Built-in command to create shortcuts |
| `ll` | The **new shortcut name** you are creating |
| `=` | No spaces around the equals sign |
| `'ls -la'` | The **command** the alias expands to (quoted) |

---

## 🔢 Alias Lifecycle

```
Session alias (alias ll='ls -la')
├── Works in current terminal ✅
└── Lost when terminal closes ❌

Permanent alias (in ~/.bashrc)
├── Works in current terminal ✅
├── Works in all new terminals ✅
└── Persists across reboots ✅
```

---

## 📖 Extended Examples

### Example 1 — Basic Alias

```bash
# Create alias
alias ll='ls -la'

# Use it
ll
```

```bash
total 56
drwxr-xr-x  5 devuser devuser 4096 Mar 21 10:00 .
drwxr-xr-x 12 root    root    4096 Mar 20 08:00 ..
-rw-r--r--  1 devuser devuser  220 Mar 18 08:00 .bash_logout
-rw-r--r--  1 devuser devuser 3526 Mar 18 08:00 .bashrc
```

---

### Example 2 — List All Current Aliases

```bash
# Show all defined aliases
alias

# Show a specific alias
alias ll
```

```bash
# alias output:
alias ll='ls -la'
alias grep='grep --color=auto'
alias cp='cp -i'
alias rm='rm -i'
alias mv='mv -i'
```

---

### Example 3 — Common Useful Aliases

```bash
# Better ls commands
alias ll='ls -la'
alias la='ls -A'
alias l='ls -CF'
alias lh='ls -lah'

# Navigation shortcuts
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias ~='cd ~'

# Safety nets
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Colorful and human-readable output
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'
alias free='free -h'

# Quick system info
alias ports='ss -tlnp'
alias myip='curl -s ifconfig.me'
alias update='sudo apt update && sudo apt upgrade -y'
```

---

### Example 4 — Alias for Long Commands

```bash
# Shortcut for checking nginx status
alias nginx-status='sudo systemctl status nginx'

# Shortcut for restarting nginx
alias nginx-restart='sudo systemctl restart nginx'

# Shortcut for checking logs
alias syslog='tail -f /var/log/syslog'
alias authlog='tail -f /var/log/auth.log'

# Docker shortcuts
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'

# Git shortcuts
alias gs='git status'
alias ga='git add .'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph'
```

---

### Example 5 — Remove an Alias

```bash
# Remove a single alias
unalias ll

# Remove all aliases
unalias -a

# Verify it is removed
alias ll
# bash: alias: ll: not found
```

---

### Example 6 — Bypass an Alias

Sometimes you need the original command, not the alias:

```bash
# Use backslash to bypass the alias
\rm file.txt     # Runs real rm, not the aliased rm -i

# Use the full path
/bin/rm file.txt

# Use the command builtin
command rm file.txt
```

> 💡 `\rm` is useful when your alias adds safety flags (`rm -i`) but you want to skip the prompt for one command.

---

### Example 7 — Make Aliases Permanent

```bash
# Add aliases to ~/.bashrc (runs for every interactive shell)
nano ~/.bashrc

# Add these lines at the bottom:
alias ll='ls -la'
alias ..='cd ..'
alias grep='grep --color=auto'
alias df='df -h'
alias update='sudo apt update && sudo apt upgrade -y'

# Apply changes immediately (without reopening terminal)
source ~/.bashrc
# or
. ~/.bashrc
```

---

### Example 8 — Organize Aliases in a Separate File

For cleaner organization, put aliases in their own file:

```bash
# Create a dedicated aliases file
touch ~/.bash_aliases

# Add your aliases there
cat >> ~/.bash_aliases << 'EOF'
# Navigation
alias ll='ls -la'
alias ..='cd ..'
alias ...='cd ../..'

# System
alias df='df -h'
alias free='free -h'
alias ports='ss -tlnp'

# Git
alias gs='git status'
alias gp='git push'
alias gl='git log --oneline --graph'
EOF

# Make ~/.bashrc load your aliases file
grep -q '.bash_aliases' ~/.bashrc || echo '
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi' >> ~/.bashrc

# Apply
source ~/.bashrc
```

> 💡 Many Linux distros already include this block in the default `~/.bashrc` — check if it is there before adding it again.

---

### Example 9 — Alias with Multiple Commands

```bash
# Run multiple commands with one alias (use semicolons)
alias update='sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y'

# Alias to go to a directory and list its contents
alias cdd='cd ~/Downloads && ls -la'

# Alias to backup a file before editing
alias vimb='cp "$1" "$1.bak" && vim "$1"'
```

> ⚠️ Aliases do not support arguments well. For argument-based shortcuts, use a **function** instead:

```bash
# Function that accepts arguments (more powerful than alias)
mkcd() {
  mkdir -p "$1" && cd "$1"
}

# Usage:
mkcd /new/project/dir    # Creates and navigates to the directory
```

---

### Example 10 — Useful Security and DevOps Aliases

```bash
# System monitoring
alias top='htop'                           # Use htop instead of top
alias psg='ps aux | grep'                  # Quick process search
alias meminfo='free -h && cat /proc/meminfo | head -5'
alias cpuinfo='lscpu | grep -E "^CPU|^Model|^Thread"'

# Network
alias myip='curl -s ifconfig.me && echo'
alias localip="ip addr | grep 'inet ' | awk '{print \$2}'"
alias ports='sudo ss -tlnp'
alias ping='ping -c 5'                     # Limit to 5 packets

# Safety
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias chown='chown --preserve-root'
alias chmod='chmod --preserve-root'

# Timestamps
alias now='date +"%Y-%m-%d %H:%M:%S"'
alias today='date +"%Y-%m-%d"'
```

---

## 🗂️ Alias Commands — Quick Reference

| Command | Description |
|---------|-------------|
| `alias name='command'` | Create an alias |
| `alias` | List all aliases |
| `alias name` | Show a specific alias |
| `unalias name` | Remove an alias |
| `unalias -a` | Remove all aliases |
| `\command` | Bypass an alias for one command |
| `command cmd` | Bypass alias using `command` builtin |

---

## 🔄 Alias vs Function — When to Use Each

| Feature | Alias | Function |
|---------|-------|---------|
| Simple command shortcut | ✅ | ✅ |
| Accepts arguments | ❌ | ✅ |
| Multi-line logic | ❌ | ✅ |
| Conditionals (`if/else`) | ❌ | ✅ |
| Loops | ❌ | ✅ |
| Best for | Simple shortcuts | Complex shortcuts |

```bash
# Alias — simple, no arguments
alias gs='git status'

# Function — with arguments
gcommit() {
  git add . && git commit -m "$1"
}

# Usage:
gcommit "Fix login bug"
```

---

## 🚀 Full Workflow — Set Up a Personal Alias Configuration

```bash
# Step 1: Create a dedicated alias file
touch ~/.bash_aliases

# Step 2: Add your favourite aliases
cat > ~/.bash_aliases << 'EOF'
# ─── Navigation ──────────────────
alias ll='ls -la'
alias la='ls -A'
alias lh='ls -lah'
alias ..='cd ..'
alias ...='cd ../..'

# ─── Safety ──────────────────────
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# ─── System ──────────────────────
alias df='df -h'
alias free='free -h'
alias ports='sudo ss -tlnp'
alias myip='curl -s ifconfig.me'

# ─── Git ─────────────────────────
alias gs='git status'
alias ga='git add .'
alias gp='git push'
alias gl='git log --oneline --graph --all'
EOF

# Step 3: Ensure ~/.bashrc loads the aliases file
echo '[ -f ~/.bash_aliases ] && . ~/.bash_aliases' >> ~/.bashrc

# Step 4: Apply immediately
source ~/.bashrc

# Step 5: Verify aliases are loaded
alias | grep ll
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Spaces around `=` break the alias | `alias ll='ls -la'` ✅ — `alias ll = 'ls -la'` ❌ |
| Alias works now but gone after reboot | Add to `~/.bashrc` and run `source ~/.bashrc` |
| Alias does not work in scripts | Scripts do not load `~/.bashrc` by default — use functions or full commands |
| Alias name conflicts with a command | Use `which name` to check for conflicts first |
| Forgetting to quote the command | Always quote the expansion: `alias name='command with spaces'` |
| Alias not loading in new terminals | Check that `~/.bashrc` sources `~/.bash_aliases` |

---

## 🔑 Key Takeaways

- `alias name='command'` creates a shortcut — **no spaces around `=`**.
- Aliases are **session-only** by default — add to `~/.bashrc` or `~/.bash_aliases` for persistence.
- Use `source ~/.bashrc` (or `. ~/.bashrc`) to apply changes without reopening the terminal.
- `alias` with no arguments **lists all defined aliases**.
- `unalias name` removes an alias; `\command` bypasses it for a single use.
- Aliases **do not accept arguments** — use shell **functions** for that.
- Put aliases in `~/.bash_aliases` for clean organization — most distros already auto-load this file.

---

## 📚 Related Concepts

- [GNU Bash Aliases Manual](https://www.gnu.org/software/bash/manual/bash.html#Aliases)

</details>
