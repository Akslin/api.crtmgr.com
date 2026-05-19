# A collection of production-ready bash scripts and terminal one-liners to help DevOps engineers debug HTTP routing, network latency, and SSL issues using api.crtmgr.com

Base URL: `https://api.crtmgr.com/api/v1`

## Endpoint Overview

| Endpoint | Method | Expected Status | Description |
| --- | --- | --- | --- |
| `/api/v1/info` | `GET` | 200 | API version and author details. |
| `/api/v1/health` | `GET` | 200 | Service health status. |
| `/api/v1/timestamp` | `GET` | 200 | Server time in Unix and ISO-8601 formats. |
| `/api/v1/limits` | `GET` | 200 | Rate-limit usage for the calling IP. |
| `/api/v1/ip` | `GET` | 200 | Caller IPv4 address. |
| `/api/v1/ipv6` | `GET` | 200 / 404 | Caller IPv6 address (404 if unavailable). |
| `/api/v1/user-agent` | `GET` | 200 | Caller User-Agent string. |
| `/api/v1/headers` | `GET` | 200 | Echo all incoming request headers. |
| `/api/v1/status/{code}` | `GET` | *as requested* | Returns the exact HTTP status code requested. |
| `/api/v1/method` | `ANY` | 200 | Returns the HTTP method used by the request. |
| `/api/v1/echo` | `POST` | 200 / 400 / 413 | Echoes request body (max 1 KB). |
| `/api/v1/test-429` | `GET` | 429 | Always returns 429 for rate-limit testing. |
| `/api/v1/delay/{sec}` | `GET` | 200 / 400 | Delays response by `sec` seconds (1–5). |
| `/api/v1/timeout` | `GET` | — | Server never responds (hung connection test). |
| `/api/v1/drop` | `GET` | — | Drops TCP connection immediately. |

---

## Scripts

Run all commands from the repository root.

| Platform | Directory | File per endpoint |
| --- | --- | --- |
| Linux (Bash) | `api.crtmgr.com/scripts/linux/` | `<endpoint>.sh` |
| Windows (PowerShell) | `api.crtmgr.com/scripts/windows/` | `<endpoint>.ps1` |

```
scripts/
├── linux/
│   ├── info.sh
│   ├── health.sh
│   ├── timestamp.sh
│   ├── limits.sh
│   ├── ip.sh
│   ├── ipv6.sh
│   ├── user-agent.sh
│   ├── headers.sh
│   ├── status.sh
│   ├── method.sh
│   ├── echo.sh
│   ├── test-429.sh
│   ├── delay.sh
│   ├── timeout.sh
│   └── drop.sh
└── windows/
    ├── info.ps1
    ├── health.ps1
    ├── timestamp.ps1
    ├── limits.ps1
    ├── ip.ps1
    ├── ipv6.ps1
    ├── user-agent.ps1
    ├── headers.ps1
    ├── status.ps1
    ├── method.ps1
    ├── echo.ps1
    ├── test-429.ps1
    ├── delay.ps1
    ├── timeout.ps1
    └── drop.ps1
```

---

## Linux — All Examples (Bash + curl)

### `/api/v1/info`

```bash
# Raw response
curl -s https://api.crtmgr.com/api/v1/info

# Pretty JSON using python3 -m json.tool
curl -s https://api.crtmgr.com/api/v1/info | python3 -m json.tool

# HTTP status code only
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/info
```

### `/api/v1/health`

```bash
curl -s https://api.crtmgr.com/api/v1/health | python3 -m json.tool
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/health
```

### `/api/v1/timestamp`

```bash
curl -s https://api.crtmgr.com/api/v1/timestamp | python3 -m json.tool
```

### `/api/v1/limits`

```bash
curl -s https://api.crtmgr.com/api/v1/limits | python3 -m json.tool

# Include response headers
curl -si https://api.crtmgr.com/api/v1/limits
```

### `/api/v1/ip`

```bash
curl -s https://api.crtmgr.com/api/v1/ip | python3 -m json.tool
```

### `/api/v1/ipv6`

```bash
# Returns 200 + IPv6 address, or 404 when no IPv6 connectivity
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/ipv6
curl -s https://api.crtmgr.com/api/v1/ipv6 | python3 -m json.tool
```

### `/api/v1/user-agent`

```bash
# Default User-Agent
curl -s https://api.crtmgr.com/api/v1/user-agent | python3 -m json.tool

# Custom User-Agent
curl -s -A "MyApp/2.0" https://api.crtmgr.com/api/v1/user-agent | python3 -m json.tool

# Empty User-Agent
curl -s -A "" https://api.crtmgr.com/api/v1/user-agent | python3 -m json.tool
```

### `/api/v1/headers`

```bash
# Default headers
curl -s https://api.crtmgr.com/api/v1/headers

# Custom headers
curl -s https://api.crtmgr.com/api/v1/headers -H "X-Custom-Header: hello" -H "Accept-Language: en-US" -H "X-Request-Id: abc-123"

# With Authorization
curl -s https://api.crtmgr.com/api/v1/headers -H "Authorization: Bearer test-token-xyz"

# Default headers with python3 -m json.tool
curl -s https://api.crtmgr.com/api/v1/headers | python3 -m json.tool

# Custom headers with python3 -m json.tool
curl -s https://api.crtmgr.com/api/v1/headers \
  -H "X-Custom-Header: hello" \
  -H "Accept-Language: en-US" \
  -H "X-Request-Id: abc-123" | python3 -m json.tool

# With Authorization with python3 -m json.tool
curl -s https://api.crtmgr.com/api/v1/headers \
  -H "Authorization: Bearer test-token-xyz" | python3 -m json.tool
```

### `/api/v1/status/{code}` — All HTTP Status Codes

```bash
# 1xx Informational
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/100   # Continue
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/101   # Switching Protocols

# 2xx Success
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/200   # OK
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/201   # Created
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/202   # Accepted
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/204   # No Content

# 3xx Redirection
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/301   # Moved Permanently
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/302   # Found
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/304   # Not Modified
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/307   # Temporary Redirect
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/308   # Permanent

# 4xx Client Errors
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/400   # Bad Request
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/401   # Unauthorized
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/403   # Forbidden
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/404   # Not Found
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/405   # Method Not Allowed
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/408   # Request Timeout
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/409   # Conflict
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/410   # Gone
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/413   # Payload Too Large
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/422   # Unprocessable Entity
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/429   # Too Many Requests
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/451   # Unavailable For Legal Reasons

# 5xx Server Errors
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/500   # Internal Server Error
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/501   # Not Implemented
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/502   # Bad Gateway
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/503   # Service Unavailable
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/504   # Gateway Timeout
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/507   # Insufficient Storage
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/status/511   # Network Authentication
```

### `/api/v1/method` — All HTTP Methods

```bash
curl -s -X GET     https://api.crtmgr.com/api/v1/method | python3 -m json.tool
curl -s -X POST    https://api.crtmgr.com/api/v1/method | python3 -m json.tool
curl -s -X PUT     https://api.crtmgr.com/api/v1/method | python3 -m json.tool
curl -s -X PATCH   https://api.crtmgr.com/api/v1/method | python3 -m json.tool
curl -s -X DELETE  https://api.crtmgr.com/api/v1/method | python3 -m json.tool
curl -s -X OPTIONS https://api.crtmgr.com/api/v1/method | python3 -m json.tool
```

### `/api/v1/echo`

```bash
# JSON body → 200
curl -s -X POST https://api.crtmgr.com/api/v1/echo \
  -H "Content-Type: application/json" \
  -d '{"debug": "on", "id": 501}' | python3 -m json.tool

# Plain text body → 200
curl -s -X POST https://api.crtmgr.com/api/v1/echo \
  -H "Content-Type: text/plain" \
  -d "hello world"

# Nested JSON → 200
curl -s -X POST https://api.crtmgr.com/api/v1/echo \
  -H "Content-Type: application/json" \
  -d '{"user":{"name":"Alice","roles":["admin","viewer"]},"action":"login"}' | python3 -m json.tool


```

### `/api/v1/test-429`

```bash
# Always returns 429
curl -si https://api.crtmgr.com/api/v1/test-429

# Status code only
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/test-429

# Check Retry-After header
curl -sI https://api.crtmgr.com/api/v1/test-429 | grep -i "Retry-After"
```

### `/api/v1/delay/{sec}`

```bash
# Valid delays: 1 seconds
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/delay/1

# Valid delays: 5 seconds
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/delay/5

# Out-of-range → 400
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/delay/0
curl -s -o /dev/null -w "%{http_code}\n" https://api.crtmgr.com/api/v1/delay/6
```

### `/api/v1/timeout`

```bash

curl https://api.crtmgr.com/api/v1/timeout 

# Server never responds — use --max-time to avoid hanging indefinitely
curl -s --max-time 3 -o /dev/null -w "%{http_code}\n" \
  https://api.crtmgr.com/api/v1/timeout || echo "Connection timed out (expected)"

curl -s --max-time 5 -o /dev/null \
  -w "HTTP: %{http_code}  Time: %{time_total}s\n" \
  https://api.crtmgr.com/api/v1/timeout || echo "Connection timed out (expected)"
```

### `/api/v1/drop`

```bash
# TCP connection dropped
curl https://api.crtmgr.com/api/v1/drop

# Verbose output showing the connection drop
curl -sv https://api.crtmgr.com/api/v1/drop 2>&1 | grep -E "(Connected|recv|Empty reply|curl:)"
```

---

## Windows — All Examples (PowerShell)

### `/api/v1/info`

```powershell
# Raw object
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/info"

# Pretty JSON
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/info" | ConvertTo-Json -Depth 10

# HTTP status code only
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/info" -SkipHttpErrorCheck).StatusCode
```

### `/api/v1/health`

```powershell
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/health" | ConvertTo-Json -Depth 10
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/health" -SkipHttpErrorCheck).StatusCode
```

### `/api/v1/timestamp`

```powershell
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/timestamp" | ConvertTo-Json -Depth 10
```

### `/api/v1/limits`

```powershell
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/limits" | ConvertTo-Json -Depth 10

# Include response headers
$r = Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/limits" -SkipHttpErrorCheck
$r.Headers | Format-Table -AutoSize
```

### `/api/v1/ip`

```powershell
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/ip" | ConvertTo-Json -Depth 10
```

### `/api/v1/ipv6`

```powershell
# Returns 200 + IPv6, or 404 when no IPv6 connectivity
$r = Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/ipv6" -SkipHttpErrorCheck
Write-Host "Status: $($r.StatusCode)"
Write-Host $r.Content
```

### `/api/v1/user-agent`

```powershell
# Default User-Agent
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/user-agent" | ConvertTo-Json -Depth 10

# Custom User-Agent
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/user-agent" `
  -Headers @{"User-Agent" = "MyApp/2.0"} | ConvertTo-Json -Depth 10
```

### `/api/v1/headers`

```powershell
# Default headers
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/headers" | ConvertTo-Json -Depth 10

# Custom headers
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/headers" -Headers @{
  "X-Custom-Header" = "hello"
  "Accept-Language" = "en-US"
  "X-Request-Id"    = "abc-123"
} | ConvertTo-Json -Depth 10

# With Authorization
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/headers" `
  -Headers @{"Authorization" = "Bearer test-token-xyz"} | ConvertTo-Json -Depth 10
```

### `/api/v1/status/{code}` — All HTTP Status Codes

```powershell
# 1xx Informational
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/100" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/101" -SkipHttpErrorCheck).StatusCode

# 2xx Success
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/200" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/201" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/202" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/204" -SkipHttpErrorCheck).StatusCode

# 3xx Redirection
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/301" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/302" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/304" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/307" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/308" -SkipHttpErrorCheck).StatusCode

# 4xx Client Errors
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/400" -SkipHttpErrorCheck).StatusCode  # Bad Request
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/401" -SkipHttpErrorCheck).StatusCode  # Unauthorized
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/403" -SkipHttpErrorCheck).StatusCode  # Forbidden
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/404" -SkipHttpErrorCheck).StatusCode  # Not Found
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/405" -SkipHttpErrorCheck).StatusCode  # Method Not Allowed
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/408" -SkipHttpErrorCheck).StatusCode  # Request Timeout
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/409" -SkipHttpErrorCheck).StatusCode  # Conflict
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/410" -SkipHttpErrorCheck).StatusCode  # Gone
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/413" -SkipHttpErrorCheck).StatusCode  # Payload Too Large
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/422" -SkipHttpErrorCheck).StatusCode  # Unprocessable Entity
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/429" -SkipHttpErrorCheck).StatusCode  # Too Many Requests
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/451" -SkipHttpErrorCheck).StatusCode  # Unavailable For Legal Reasons

# 5xx Server Errors
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/500" -SkipHttpErrorCheck).StatusCode  # Internal Server Error
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/501" -SkipHttpErrorCheck).StatusCode  # Not Implemented
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/502" -SkipHttpErrorCheck).StatusCode  # Bad Gateway
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/503" -SkipHttpErrorCheck).StatusCode  # Service Unavailable
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/504" -SkipHttpErrorCheck).StatusCode  # Gateway Timeout
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/507" -SkipHttpErrorCheck).StatusCode  # Insufficient Storage
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/status/511" -SkipHttpErrorCheck).StatusCode  # Network Authentication Required
```

### `/api/v1/method` — All HTTP Methods

```powershell
foreach ($method in @("GET","POST","PUT","PATCH","DELETE","OPTIONS")) {
    Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/method" -Method $method | ConvertTo-Json -Depth 5
}

# HEAD (no body)
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/method" -Method HEAD -SkipHttpErrorCheck).StatusCode
```

### `/api/v1/echo`

```powershell
# JSON body → 200
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/echo" -Method Post `
  -ContentType "application/json" `
  -Body '{"debug": "on", "id": 501}' | ConvertTo-Json -Depth 10

# Plain text → 200
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/echo" -Method Post `
  -ContentType "text/plain" -Body "hello world"

# Nested JSON → 200
Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/echo" -Method Post `
  -ContentType "application/json" `
  -Body '{"user":{"name":"Alice","roles":["admin","viewer"]},"action":"login"}' | ConvertTo-Json -Depth 10

# Empty body → 400
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/echo" -Method Post -SkipHttpErrorCheck).StatusCode

# Body > 1 KB → 413
$largeBody = "x" * 1025
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/echo" -Method Post `
  -ContentType "text/plain" -Body $largeBody -SkipHttpErrorCheck).StatusCode
```

### `/api/v1/test-429`

```powershell
# Always returns 429
$r = Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/test-429" -SkipHttpErrorCheck
Write-Host "Status: $($r.StatusCode)"
Write-Host "Body:   $($r.Content)"

# Retry-After header
$r.Headers["Retry-After"]
```

### `/api/v1/delay/{sec}`

```powershell
# Valid delays: 1–5 seconds
foreach ($sec in @(1, 2, 3, 4, 5)) {
    $start = Get-Date
    $r = Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/delay/$sec" -SkipHttpErrorCheck
    $elapsed = [math]::Round(((Get-Date) - $start).TotalSeconds, 2)
    Write-Host "Delay ${sec}s → HTTP $($r.StatusCode)  Elapsed: ${elapsed}s"
}

# Out-of-range → 400
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/delay/0" -SkipHttpErrorCheck).StatusCode
(Invoke-WebRequest -Uri "https://api.crtmgr.com/api/v1/delay/6" -SkipHttpErrorCheck).StatusCode

# Client timeout shorter than delay → exception
try {
    Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/delay/3" -TimeoutSec 2
} catch {
    Write-Host "Client timed out (expected): $_"
}
```

### `/api/v1/timeout`

```powershell
# Server never responds — TimeoutSec prevents hanging indefinitely
try {
    Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/timeout" -TimeoutSec 3
} catch {
    Write-Host "Connection timed out (expected): $_"
}
```

### `/api/v1/drop`

```powershell
# TCP connection dropped — expect an exception (no HTTP response)
try {
    Invoke-RestMethod -Uri "https://api.crtmgr.com/api/v1/drop"
} catch {
    Write-Host "Connection dropped (expected): $_"
}
```
