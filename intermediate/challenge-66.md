# Challenge 66: Configure Firewall in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

A **firewall** controls which network traffic is allowed to reach your server. **`firewalld`** is the default dynamic firewall manager on RHEL, CentOS, Fedora, and Rocky Linux. It uses the concept of **zones** and **services** to define rules, and supports changes at runtime without interrupting existing connections.

Unlike raw `iptables`, `firewalld` provides a higher-level interface — managing services by name (e.g., `http`, `ssh`) rather than raw port numbers — while still supporting custom port rules when needed.

---

## 🧩 Task

Allow incoming HTTP traffic (port 80) through the firewall permanently.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Allow HTTP service permanently
sudo firewall-cmd --add-service=http --permanent

# Reload firewall to apply the permanent rule
sudo firewall-cmd --reload
```

### How it works

| Part | Description |
|------|-------------|
| `firewall-cmd` | Command-line interface for `firewalld` |
| `--add-service=http` | Allow the predefined `http` service (port 80/tcp) |
| `--permanent` | Make the rule **persist across reboots** (without this, rule is lost on restart) |
| `--reload` | Apply permanent rules to the **runtime** configuration without restarting the service |

### Runtime vs Permanent Rules

```
  Without --permanent          With --permanent + --reload
  ─────────────────────        ───────────────────────────
  Rule active immediately      Rule saved to disk
  Lost after reboot            Survives reboots
  Good for testing             Required for production
```

---

## 📖 Extended Examples

### Example 1 — Allow HTTP (Port 80)

```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

---

### Example 2 — Allow HTTPS (Port 443)

```bash
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

---

### Example 3 — Allow Both HTTP and HTTPS in One Command

```bash
sudo firewall-cmd --add-service=http --add-service=https --permanent
sudo firewall-cmd --reload
```

---

### Example 4 — Allow a Custom Port

```bash
# Allow a specific TCP port
sudo firewall-cmd --add-port=8080/tcp --permanent

# Allow a UDP port
sudo firewall-cmd --add-port=53/udp --permanent

# Allow a range of ports
sudo firewall-cmd --add-port=8000-8100/tcp --permanent

sudo firewall-cmd --reload
```

---

### Example 5 — Remove (Block) a Service or Port

```bash
# Remove HTTP service
sudo firewall-cmd --remove-service=http --permanent

# Remove a custom port
sudo firewall-cmd --remove-port=8080/tcp --permanent

sudo firewall-cmd --reload
```

---

### Example 6 — Check Firewall Status and Active Rules

```bash
# Check if firewalld is running
sudo firewall-cmd --state

# List all active rules in the default zone
sudo firewall-cmd --list-all
```

```bash
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client http https ssh
  ports: 8080/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

---

### Example 7 — List All Available Predefined Services

```bash
# List all services firewalld knows about
sudo firewall-cmd --get-services
```

```bash
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula
bacula-client bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph
ceph-mon cfengine cockpit condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns
docker-registry ...ftp http https imap imaps ipp ipp-client ipsec ...ssh ...
```

> 💡 Using service names is cleaner than memorizing port numbers — `http` automatically maps to `80/tcp`.

---

### Example 8 — Work with Zones

`firewalld` uses **zones** to define trust levels for network interfaces. Each zone has its own set of rules.

```bash
# List all available zones
sudo firewall-cmd --get-zones

# See the default zone
sudo firewall-cmd --get-default-zone

# List rules for a specific zone
sudo firewall-cmd --zone=public --list-all

# Add a service to a specific zone
sudo firewall-cmd --zone=public --add-service=http --permanent

# Change the default zone
sudo firewall-cmd --set-default-zone=home
```

| Zone | Trust Level | Typical Use |
|------|------------|-------------|
| `drop` | None | Silently drops all incoming traffic |
| `block` | None | Rejects all incoming traffic |
| `public` | Low | Public-facing servers (default) |
| `external` | Low | External facing with masquerade |
| `internal` | Medium | Internal network |
| `work` | Medium | Work network (trusted users) |
| `home` | High | Home network |
| `trusted` | Full | All connections accepted |
| `dmz` | Low | DMZ — public-facing, limited internal access |

---

### Example 9 — Allow Access from a Specific IP Address (Rich Rules)

```bash
# Allow HTTP from a specific IP
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.50" service name="http" accept' --permanent

# Allow SSH from a subnet only
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept' --permanent

# Block a specific IP completely
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.0.0.5" reject' --permanent

sudo firewall-cmd --reload
```

---

### Example 10 — Allow SSH on a Custom Port

```bash
# Allow SSH on port 2222 instead of 22
sudo firewall-cmd --add-port=2222/tcp --permanent

# Remove the default SSH service if you changed the port
sudo firewall-cmd --remove-service=ssh --permanent

sudo firewall-cmd --reload
```

> ⚠️ Open the new port **before** removing the old one to avoid locking yourself out.

---

### Example 11 — Enable Port Forwarding / Masquerading

```bash
# Enable IP masquerading (NAT) — needed for port forwarding
sudo firewall-cmd --add-masquerade --permanent

# Forward external port 80 to internal host
sudo firewall-cmd \
  --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.10:toport=8080 \
  --permanent

sudo firewall-cmd --reload
```

---

### Example 12 — Allow a Service Temporarily (Runtime Only — For Testing)

```bash
# Add rule at runtime only (lost after reboot)
sudo firewall-cmd --add-service=http

# Test the service
curl http://localhost

# If OK, make it permanent
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

# If NOT OK, remove it without touching permanent rules
sudo firewall-cmd --remove-service=http
```

> 💡 This is the **safest workflow** — test at runtime first, then commit to permanent config only when confirmed working.

---

### Example 13 — Panic Mode (Emergency Block All Traffic)

```bash
# Enable panic mode — immediately blocks ALL traffic (including your SSH session!)
sudo firewall-cmd --panic-on

# Disable panic mode
sudo firewall-cmd --panic-off

# Check if panic mode is active
sudo firewall-cmd --query-panic
```

> ⚠️ `--panic-on` will **drop your SSH connection immediately**. Only use on physical/console access or in an emergency.

---

## 🗂️ `firewall-cmd` Quick Reference

| Command | Description |
|---------|-------------|
| `--state` | Check if firewalld is running |
| `--list-all` | List all rules in the default zone |
| `--list-all --zone=NAME` | List rules for a specific zone |
| `--get-zones` | List all available zones |
| `--get-default-zone` | Show the default zone |
| `--get-services` | List all predefined services |
| `--add-service=NAME` | Allow a service |
| `--remove-service=NAME` | Remove a service |
| `--add-port=PORT/PROTO` | Allow a custom port |
| `--remove-port=PORT/PROTO` | Remove a custom port |
| `--add-rich-rule='RULE'` | Add an advanced rule |
| `--add-masquerade` | Enable NAT/masquerading |
| `--permanent` | Persist rule across reboots |
| `--reload` | Apply permanent rules to runtime |
| `--runtime-to-permanent` | Save all runtime rules as permanent |
| `--panic-on / --panic-off` | Emergency block all traffic |

---

## 🔄 `firewalld` vs `ufw` vs `iptables`

| Feature | `firewalld` | `ufw` | `iptables` |
|---------|-------------|-------|------------|
| Default on | RHEL/CentOS/Fedora | Ubuntu/Debian | All (backend) |
| Interface | High-level (zones/services) | High-level (simple rules) | Low-level (raw rules) |
| Dynamic reload | ✅ No restart needed | ❌ | ❌ |
| Zone support | ✅ | ❌ | Manual |
| Rich rules | ✅ | Limited | ✅ |
| Complexity | Medium | Low | High |
| Best for | RHEL-based servers | Ubuntu servers | Advanced/custom setups |

---

## 🚀 Full Workflow — Open HTTP and HTTPS on a Web Server

```bash
# Step 1: Check firewalld is running
sudo firewall-cmd --state

# Step 2: See what's currently allowed
sudo firewall-cmd --list-all

# Step 3: Test at runtime first (no --permanent)
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https

# Step 4: Verify the service is reachable
curl -I http://localhost
curl -I https://localhost

# Step 5: Make rules permanent
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent

# Step 6: Reload to sync runtime and permanent config
sudo firewall-cmd --reload

# Step 7: Verify final state
sudo firewall-cmd --list-all
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Added rule but forgot `--reload` | Run `sudo firewall-cmd --reload` to push permanent rules to runtime |
| Rule added without `--permanent` | Rule will disappear after reboot — always add `--permanent` for production |
| Blocked own SSH session | Keep port `22` or your SSH port open at all times before reloading |
| `firewalld` not installed | Install with `sudo yum install firewalld` or `sudo dnf install firewalld` |
| Wrong zone being configured | Verify your interface zone with `sudo firewall-cmd --get-active-zones` |
| Rich rule syntax error | Test with `sudo firewall-cmd --check-config` before reloading |

---

## 🔑 Key Takeaways

- `--permanent` saves the rule to disk — without it, rules are **lost after reboot**.
- Always run `--reload` after adding `--permanent` rules to activate them at runtime.
- Use **service names** (`http`, `https`, `ssh`) instead of raw ports whenever possible — it's cleaner and self-documenting.
- **Test rules at runtime first** (no `--permanent`), then commit with `--permanent` + `--reload`.
- Use **zones** to apply different trust levels to different network interfaces.
- Use **rich rules** for IP-specific allow/deny rules when service-level control isn't enough.
- Always verify with `--list-all` after making changes to confirm rules are applied correctly.

---

## 📚 Related Concepts

- [firewalld Documentation](https://firewalld.org/documentation/)
- [firewall-cmd Man Page](https://linux.die.net/man/1/firewall-cmd)
- [UFW Guide](https://help.ubuntu.com/community/UFW) — Ubuntu's simpler firewall alternative
- [nftables](https://wiki.nftables.org/) — the modern backend replacing iptables

</details>
