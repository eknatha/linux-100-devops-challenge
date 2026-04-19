# Challenge 44: DNS Resolution in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)


![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**DNS (Domain Name System)** translates human-readable hostnames like `google.com` into IP addresses that computers use to communicate. DNS resolution failures are one of the most common causes of "service unreachable" errors — even when the network is physically working fine.

Linux provides three primary tools for checking DNS resolution:
- **`nslookup`** — simple and interactive, works on all platforms
- **`dig`** — detailed DNS query tool with full response information
- **`host`** — simplest output, good for quick lookups

---

## 🧩 Task

Check DNS resolution to verify that a hostname correctly resolves to an IP address.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Quick DNS lookup with nslookup
nslookup google.com

# Detailed DNS query with dig
dig google.com

# Simple one-line answer
host google.com
```

### How it works

| Command | Description |
|---------|-------------|
| `nslookup hostname` | Query DNS and show the resolved IP |
| `dig hostname` | Full DNS query with detailed response |
| `host hostname` | Simple DNS lookup with clean output |

---

## 🔢 Understanding `nslookup` Output

```bash
nslookup google.com
```

```
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:    google.com
Address: 142.250.80.46
```

| Field | Description |
|-------|-------------|
| `Server` | The **DNS server** used to resolve the query |
| `Address` | DNS server IP + port 53 |
| `Non-authoritative answer` | Cached result — not from the authoritative DNS server |
| `Name` | The hostname queried |
| `Address` | The resolved **IP address** |

---

## 🔢 Understanding `dig` Output

```bash
dig google.com
```

```
; <<>> DiG 9.18.1 <<>> google.com
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.80.46

;; Query time: 8 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Thu Mar 21 10:35:22 UTC 2025
;; MSG SIZE  rcvd: 55
```

| Field | Description |
|-------|-------------|
| `QUESTION SECTION` | The query sent |
| `ANSWER SECTION` | The resolved record |
| `300` | **TTL** (Time To Live) — seconds before cache expires |
| `IN A` | **Record type** — A = IPv4 address |
| `Query time` | How long DNS resolution took |
| `SERVER` | Which DNS server answered |

---

## 📖 Extended Examples

### Example 1 — Basic DNS Lookup with `nslookup`

```bash
nslookup google.com
```

---

### Example 2 — Simple Lookup with `host`

```bash
host google.com
```

```bash
google.com has address 142.250.80.46
google.com has IPv6 address 2607:f8b0:4004:c1b::64
google.com mail is handled by 10 smtp.google.com.
```

> `host` is the cleanest output — shows A, AAAA, and MX records in one view.

---

### Example 3 — Short Answer Only with `dig`

```bash
# Just the IP address
dig +short google.com

# With record type specified
dig +short google.com A
```

```bash
142.250.80.46
```

---

### Example 4 — Query Specific DNS Record Types

```bash
# A record (IPv4)
dig google.com A

# AAAA record (IPv6)
dig google.com AAAA

# MX record (mail server)
dig google.com MX

# NS record (nameservers)
dig google.com NS

# TXT record (SPF, DKIM, verification)
dig google.com TXT

# PTR record (reverse DNS — IP to hostname)
dig -x 142.250.80.46

# CNAME record (alias)
dig www.github.com CNAME
```

---

### Example 5 — Query a Specific DNS Server

```bash
# Query Google's public DNS
dig @8.8.8.8 google.com

# Query Cloudflare's DNS
dig @1.1.1.1 google.com

# Query your local DNS server
dig @192.168.1.1 google.com

# nslookup with specific server
nslookup google.com 8.8.8.8
```

> 💡 If `dig @8.8.8.8 google.com` works but `dig google.com` fails, the problem is your **local DNS configuration**, not the domain.

---

### Example 6 — Trace the Full DNS Resolution Path

```bash
# Trace from root servers to authoritative
dig +trace google.com
```

```bash
# Output follows the delegation chain:
.               518400  IN  NS  a.root-servers.net.
com.            172800  IN  NS  a.gtld-servers.net.
google.com.     172800  IN  NS  ns1.google.com.
google.com.     300     IN  A   142.250.80.46
```

---

### Example 7 — Check Reverse DNS (PTR Record)

```bash
# Find hostname for an IP address
dig -x 8.8.8.8

# Using host
host 8.8.8.8

# Using nslookup
nslookup 8.8.8.8
```

```bash
# dig -x output:
8.8.8.8.in-addr.arpa. 21599 IN PTR dns.google.
```

---

### Example 8 — Check `/etc/resolv.conf`

```bash
# View configured DNS servers
cat /etc/resolv.conf
```

```bash
nameserver 192.168.1.1     # Primary DNS
nameserver 8.8.8.8         # Fallback DNS
search example.com         # Default search domain
```

```bash
# With systemd-resolved
resolvectl status
resolvectl query google.com
```

---

### Example 9 — Flush DNS Cache

```bash
# Flush systemd-resolved cache
sudo systemd-resolve --flush-caches

# Or with newer systemd
sudo resolvectl flush-caches

# Restart DNS resolver
sudo systemctl restart systemd-resolved

# On RHEL — restart NetworkManager
sudo systemctl restart NetworkManager
```

---

### Example 10 — DNS Troubleshooting Script

```bash
#!/bin/bash

TARGET="${1:-google.com}"
echo "=== DNS Resolution Check: $TARGET ==="

echo -e "\n[1] Local resolver:"
dig +short "$TARGET" A 2>/dev/null || echo "FAILED"

echo -e "\n[2] Google DNS (8.8.8.8):"
dig @8.8.8.8 +short "$TARGET" A 2>/dev/null || echo "FAILED"

echo -e "\n[3] Query time:"
dig "$TARGET" | grep "Query time"

echo -e "\n[4] DNS server used:"
dig "$TARGET" | grep "SERVER:"

echo -e "\n[5] Record TTL:"
dig +noall +answer "$TARGET" | awk '{print "TTL:", $2, "seconds"}'

echo -e "\n[6] All record types:"
for TYPE in A AAAA MX NS TXT; do
  RESULT=$(dig +short "$TARGET" $TYPE 2>/dev/null)
  [ -n "$RESULT" ] && echo "$TYPE: $RESULT"
done
```

---

## 🗂️ DNS Record Types — Quick Reference

| Record | Purpose | Example |
|--------|---------|---------|
| `A` | IPv4 address | `google.com → 142.250.80.46` |
| `AAAA` | IPv6 address | `google.com → 2607:f8b0::` |
| `CNAME` | Alias (canonical name) | `www → google.com` |
| `MX` | Mail server | `google.com → smtp.google.com` |
| `NS` | Nameservers | `google.com → ns1.google.com` |
| `TXT` | Text (SPF, DKIM, verify) | `v=spf1 include:...` |
| `PTR` | Reverse DNS (IP → hostname) | `8.8.8.8 → dns.google` |
| `SOA` | Start of authority | Zone admin info |

---

## 🗂️ DNS Tools — Quick Reference

| Command | Description |
|---------|-------------|
| `nslookup hostname` | Basic DNS query |
| `nslookup hostname server` | Query specific DNS server |
| `dig hostname` | Detailed DNS query |
| `dig +short hostname` | IP address only |
| `dig @server hostname` | Query specific server |
| `dig hostname TYPE` | Query specific record type |
| `dig +trace hostname` | Trace full resolution path |
| `dig -x IP` | Reverse DNS lookup |
| `host hostname` | Simple clean output |
| `host IP` | Reverse lookup |
| `resolvectl query hostname` | systemd-resolved query |

---

## 🚀 Full Workflow — Debug a DNS Failure

```bash
# Step 1: Try basic resolution
nslookup google.com

# Step 2: Try with dig for more detail
dig google.com

# Step 3: If it fails, try a public DNS server directly
dig @8.8.8.8 google.com

# Step 4: If @8.8.8.8 works → local DNS is broken
cat /etc/resolv.conf    # Check configured nameserver
resolvectl status        # Check systemd-resolved

# Step 5: Flush DNS cache
sudo resolvectl flush-caches

# Step 6: Trace the full resolution path
dig +trace google.com

# Step 7: Verify /etc/hosts is not overriding
grep google /etc/hosts

# Step 8: Restart the DNS resolver if needed
sudo systemctl restart systemd-resolved
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `nslookup` works but app can't resolve | Check `/etc/nsswitch.conf` — `hosts` line order matters |
| DNS works but returns wrong IP | Check `/etc/hosts` for override entries |
| `dig @8.8.8.8` works but `dig` fails | Local DNS is misconfigured — fix `/etc/resolv.conf` |
| `/etc/resolv.conf` gets overwritten | Edit via `NetworkManager` or `systemd-resolved` — not directly |
| Stale cached results | Run `resolvectl flush-caches` |
| `dig` not installed | `sudo apt install dnsutils` (Debian) or `sudo dnf install bind-utils` (RHEL) |

---

## 🔑 Key Takeaways

- `nslookup hostname` is the quickest DNS check — shows which server answered and what IP was returned.
- `dig +short hostname` returns **just the IP** — perfect for scripts.
- If **local DNS fails but `@8.8.8.8` works** — the problem is your local DNS configuration.
- `dig +trace` shows the **full delegation chain** from root servers to authoritative.
- `dig -x IP` performs a **reverse DNS lookup** — IP to hostname.
- Always check **`/etc/hosts`** — it takes priority over DNS and can cause misleading results.
- Use `resolvectl flush-caches` to clear stale DNS cache before retesting.

---

## 📚 Related Concepts

- [dig Manual](https://linux.die.net/man/1/dig)
- [nslookup Manual](https://linux.die.net/man/1/nslookup)

</details>
