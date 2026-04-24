# Challenge 45: Fetch Data with curl in Linux

![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)

![Eknatha](https://img.shields.io/badge/Eknatha-4EAA25?style=flat&logo=gnu-bash&logoColor=white)

## 📌 Overview

**`curl`** (**C**lient **URL**) is the most versatile command-line tool for transferring data over the network. It supports dozens of protocols — HTTP, HTTPS, FTP, SFTP, and more — making it the go-to tool for:

- Fetching web pages and API responses
- Testing REST APIs
- Downloading files
- Checking HTTP response headers
- Measuring web server response times
- Automating web requests in scripts

---

## 🧩 Task

Fetch the content of a webpage or API endpoint from the command line.

---


<details>
<summary>💡 Click to view solution</summary>

## ✅ Solution

```bash
# Fetch a webpage (print to screen)
curl https://example.com

# Fetch just the HTTP headers
curl -I https://example.com

# Download a file
curl -O https://example.com/file.tar.gz
```

### How it works

| Command | Description |
|---------|-------------|
| `curl URL` | Fetch the URL and print to stdout |
| `-I` | **HEAD** request — show only HTTP headers |
| `-O` | **Output** — save with the original filename |
| `-o filename` | Save to a specific filename |
| `-s` | **Silent** — suppress progress output |
| `-v` | **Verbose** — show full request/response |

---

## 🔢 Understanding `curl -I` Output

```bash
curl -I https://example.com
```

```
HTTP/2 200
content-type: text/html; charset=UTF-8
content-length: 1256
date: Thu, 21 Mar 2025 10:35:22 GMT
server: nginx/1.24.0
cache-control: max-age=604800
```

| Field | Description |
|-------|-------------|
| `HTTP/2 200` | Protocol and **status code** (200 = OK) |
| `content-type` | Type of content returned |
| `content-length` | Size of response in bytes |
| `date` | Server timestamp |
| `server` | Web server software |
| `cache-control` | Cache duration |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| `200` | OK — success |
| `301` / `302` | Redirect |
| `400` | Bad Request |
| `401` | Unauthorized |
| `403` | Forbidden |
| `404` | Not Found |
| `500` | Internal Server Error |
| `502` / `503` | Bad Gateway / Service Unavailable |

---

## 📖 Extended Examples

### Example 1 — Fetch a Webpage

```bash
# Print webpage content to terminal
curl https://example.com

# Fetch and save to a file
curl https://example.com -o page.html
```

---

### Example 2 — View HTTP Headers Only

```bash
# Show only response headers (HEAD request)
curl -I https://example.com

# Show headers AND body
curl -i https://example.com
```

---

### Example 3 — Follow Redirects

```bash
# Follow HTTP redirects automatically
curl -L https://example.com

# See each redirect step
curl -Lv https://example.com 2>&1 | grep -E "< HTTP|Location"
```

---

### Example 4 — Download a File

```bash
# Save with original filename
curl -O https://example.com/archive.tar.gz

# Save with a custom filename
curl -o myarchive.tar.gz https://example.com/archive.tar.gz

# Show download progress
curl -# -O https://example.com/largefile.tar.gz

# Resume an interrupted download
curl -C - -O https://example.com/largefile.tar.gz
```

---

### Example 5 — Pass HTTP Headers

```bash
# Add a custom header
curl -H "Authorization: Bearer mytoken123" https://api.example.com/data

# Add multiple headers
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer mytoken" \
  https://api.example.com/data

# Set User-Agent
curl -A "MyApp/1.0" https://example.com
```

---

### Example 6 — Make POST Requests

```bash
# POST with form data
curl -X POST -d "username=alice&password=secret" https://example.com/login

# POST with JSON body
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "email": "alice@example.com"}' \
  https://api.example.com/users

# POST with JSON from a file
curl -X POST \
  -H "Content-Type: application/json" \
  -d @payload.json \
  https://api.example.com/users
```

---

### Example 7 — Test a REST API

```bash
# GET request
curl https://api.example.com/users

# GET with authentication
curl -H "Authorization: Bearer TOKEN" https://api.example.com/users

# POST — create a resource
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"name": "Alice"}' \
  https://api.example.com/users

# PUT — update a resource
curl -X PUT \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice Updated"}' \
  https://api.example.com/users/1

# DELETE — remove a resource
curl -X DELETE https://api.example.com/users/1
```

---

### Example 8 — Measure Response Time

```bash
# Detailed timing breakdown
curl -o /dev/null -s -w "
DNS Lookup:      %{time_namelookup}s
TCP Connect:     %{time_connect}s
TLS Handshake:   %{time_appconnect}s
Time to First Byte: %{time_starttransfer}s
Total Time:      %{time_total}s
HTTP Status:     %{http_code}
Response Size:   %{size_download} bytes
" https://example.com
```

```bash
# Output:
DNS Lookup:         0.008s
TCP Connect:        0.025s
TLS Handshake:      0.089s
Time to First Byte: 0.124s
Total Time:         0.125s
HTTP Status:        200
Response Size:      1256 bytes
```

---

### Example 9 — Silent Mode for Scripts

```bash
# Silent — no progress bar, just output
curl -s https://api.example.com/status

# Silent + fail on HTTP errors
curl -sf https://api.example.com/status

# Check if a URL is reachable
if curl -sf --head https://example.com > /dev/null; then
  echo "✅ Site is up"
else
  echo "❌ Site is down"
fi
```

---

### Example 10 — Real-World: Get Your Public IP

```bash
# Get your public IP address
curl -s ifconfig.me
curl -s icanhazip.com
curl -s api.ipify.org

# Get detailed IP info (JSON)
curl -s ipinfo.io
```

```bash
# Output:
203.0.113.42
```

---

### Example 11 — Verbose Mode for Debugging

```bash
# Show full request and response details
curl -v https://example.com

# Show only the exchange (useful for debugging)
curl -v https://example.com 2>&1 | grep -E "^[<>*]"
```

```bash
# -v output:
*   Trying 142.250.80.46:443...
* Connected to example.com (142.250.80.46) port 443 (#0)
* TLS handshake complete
> GET / HTTP/2
> Host: example.com
>
< HTTP/2 200
< content-type: text/html
<
```

---

### Example 12 — Useful curl One-Liners

```bash
# Check if a service is responding
curl -sf http://localhost:8080/health && echo "OK" || echo "DOWN"

# Download and pipe directly to shell (use with caution!)
curl -s https://example.com/install.sh | bash

# Get weather
curl -s wttr.in/London

# Get a random quote
curl -s https://api.quotable.io/random | python3 -m json.tool

# Download with retry on failure
curl --retry 3 --retry-delay 2 -O https://example.com/file.tar.gz
```

---

## 🗂️ `curl` Flags — Quick Reference

| Flag | Description |
|------|-------------|
| `-I` / `--head` | Show **headers only** (HEAD request) |
| `-i` | Show **headers + body** |
| `-v` | **Verbose** — show full request/response |
| `-s` | **Silent** — no progress output |
| `-S` | Show error even with `-s` |
| `-f` | **Fail** silently on HTTP errors (exit code 22) |
| `-o FILE` | Save output to FILE |
| `-O` | Save with **original filename** |
| `-L` | **Follow** redirects |
| `-X METHOD` | Set HTTP **method** (GET, POST, PUT, DELETE) |
| `-H "Header"` | Add a custom **header** |
| `-d DATA` | **Data** to POST |
| `-u user:pass` | **Basic authentication** |
| `-A "agent"` | Set **User-Agent** |
| `-k` | **Insecure** — skip TLS verification |
| `-C -` | **Resume** interrupted download |
| `--retry N` | **Retry** N times on failure |
| `-w FORMAT` | **Write-out** — custom output format |
| `--limit-rate 1M` | **Limit** download speed |

---

## 🔄 `curl` vs `wget`

| Feature | `curl` | `wget` |
|---------|--------|--------|
| API testing | ✅ Full HTTP methods | Limited |
| Download files | ✅ | ✅ |
| Recursive download | ❌ | ✅ |
| Resume downloads | ✅ `-C -` | ✅ `-c` |
| Custom headers | ✅ | Limited |
| Pipe to stdout | ✅ | ✅ |
| Best for | API testing, scripting | File downloads |

---

## 🚀 Full Workflow — Test a REST API

```bash
# Step 1: Check the API is reachable
curl -sf https://api.example.com/health && echo "API is up"

# Step 2: Authenticate and get a token
TOKEN=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"secret"}' \
  https://api.example.com/auth | python3 -c "import sys,json; print(json.load(sys.stdin)['token'])")

# Step 3: Use the token to fetch data
curl -s -H "Authorization: Bearer $TOKEN" https://api.example.com/users

# Step 4: Create a resource
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name": "Alice", "role": "admin"}' \
  https://api.example.com/users

# Step 5: Check timing and response code
curl -o /dev/null -s -w "Status: %{http_code} | Time: %{time_total}s\n" \
  https://api.example.com/users
```

---

## ⚠️ Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| curl follows redirect but shows wrong content | Use `-L` to follow redirects |
| SSL certificate error | Use `-k` to skip (testing only!) or fix certs |
| POST data not sent correctly | Check `-H "Content-Type: application/json"` is set |
| Progress bar messing up script output | Always use `-s` in scripts |
| `curl -O` saves wrong filename | Use `-o filename` to specify explicitly |
| curl pipe to bash — security risk | Verify the URL and source before piping |

---

## 🔑 Key Takeaways

- `curl URL` fetches content and prints to stdout — combine with `-s` to suppress progress in scripts.
- `-I` checks HTTP headers and status codes — the fastest way to verify a web server is responding.
- `-L` follows redirects — essential for modern websites that redirect HTTP → HTTPS.
- `-o filename` saves to a file; `-O` saves with the original filename from the URL.
- Use `-w "%{http_code}"` to capture the HTTP status code in scripts for conditional logic.
- `-sf` (silent + fail) is the standard for health checks in scripts — exits non-zero on HTTP errors.
- `-v` shows the full request and response — the most powerful debugging mode.

---

## 📚 Related Concepts


- [curl Manual](https://curl.se/docs/manpage.html)
- [curl Cookbook](https://catonmat.net/cookbooks/curl)

</details>
