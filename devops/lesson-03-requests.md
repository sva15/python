# 🏛️ DevOps Module | Lesson 3: Make HTTP Requests with the requests Library

## *"The Outside Counsel"*

> **DevOps Objective:** *Make HTTP requests (GET, POST, PUT, DELETE) to REST APIs, handle authentication, parse JSON responses, and build health checks using the `requests` library.*

> **Previously Learned:** Dictionaries, JSON, exception handling, f-strings, context managers

---

## 📖 Table of Contents

1. [Opening Scene](#1--opening-scene)
2. [Concept Introduction – Harvey Specter](#2--concept-introduction--harvey-specter)
3. [Technical Breakdown – Mike Ross](#3--technical-breakdown--mike-ross)
4. [Edge Cases & Pitfalls – Louis Litt](#4--edge-cases--pitfalls--louis-litt)
5. [Mental Model – Donna Paulsen](#5--mental-model--donna-paulsen)
6. [Practice Exercises – Rachel Zane](#6--practice-exercises--rachel-zane)
7. [Debugging Scenario](#7--debugging-scenario)
8. [Real-World Application](#8--real-world-application)
9. [Knowledge Check](#9--knowledge-check)

---

## 1. 🎬 Opening Scene

*The firm needs to check deployment status from GitHub, trigger builds on Jenkins, pull metrics from Prometheus, and send alerts to Slack. All via REST APIs.*

```python
# ❌ Calling curl from subprocess
import subprocess
result = subprocess.run(
    ["curl", "-s", "https://api.github.com/repos/psl/billing"],
    capture_output=True, text=True
)
data = json.loads(result.stdout)    # Clunky!

# ✅ The requests library — Python's HTTP standard
import requests
response = requests.get("https://api.github.com/repos/psl/billing")
data = response.json()    # Clean!
print(f"Stars: {data['stargazers_count']}")
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "APIs are how systems talk. `requests` is how Python joins the conversation."

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Read data | List pods, get metrics |
| `POST` | Create data | Trigger build, send alert |
| `PUT` | Replace data | Update config |
| `PATCH` | Partial update | Scale replicas |
| `DELETE` | Remove data | Delete resource |

```bash
# Install
pip install requests
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 GET Requests — Read Data ⭐⭐⭐

```python
import requests

# === Basic GET ===
response = requests.get("https://api.github.com/repos/python/cpython")
print(response.status_code)    # 200
print(response.headers["content-type"])    # application/json

# === Parse JSON response (returns a dict!) ===
data = response.json()
print(f"Repo: {data['full_name']}")
print(f"Stars: {data['stargazers_count']}")
print(f"Language: {data['language']}")

# === Response properties ===
response.status_code     # 200, 404, 500, etc.
response.ok              # True if status < 400
response.text            # Raw text body
response.json()          # Parse JSON → dict
response.content         # Raw bytes
response.headers         # Response headers (dict-like)
response.url             # Final URL (after redirects)
response.elapsed         # Time taken

# === Query parameters ===
# Instead of building URL strings manually:
# ❌ f"https://api.github.com/search/repos?q=python&sort=stars&per_page=5"
# ✅ Use params dict:
response = requests.get(
    "https://api.github.com/search/repos",
    params={
        "q": "python devops",
        "sort": "stars",
        "per_page": 5,
    },
)
repos = response.json()["items"]
for repo in repos:
    print(f"  ⭐ {repo['stargazers_count']:>6} | {repo['full_name']}")
```

---

### 3.2 POST Requests — Send Data ⭐⭐⭐

```python
import requests
import os

# === POST JSON data ===
response = requests.post(
    "https://hooks.slack.com/services/T.../B.../xxx",
    json={                              # json= auto-serializes AND sets Content-Type!
        "text": "🚀 Deployment v1.2.3 to production complete!",
        "channel": "#deployments",
    },
)
print(f"Slack: {response.status_code}")    # 200

# === POST form data ===
response = requests.post(
    "https://api.example.com/login",
    data={                              # data= sends as form-encoded
        "username": "deploy-bot",
        "password": os.environ["BOT_PASSWORD"],
    },
)

# === POST with file upload ===
with open("artifact.tar.gz", "rb") as f:
    response = requests.post(
        "https://artifacts.example.com/upload",
        files={"file": ("artifact.tar.gz", f, "application/gzip")},
        data={"version": "1.2.3"},
    )

# json= vs data=:
# json=   → sends JSON, sets Content-Type: application/json
# data=   → sends form-encoded, sets Content-Type: application/x-www-form-urlencoded
```

---

### 3.3 PUT, PATCH, DELETE ⭐⭐

```python
import requests

API = "https://api.example.com"
headers = {"Authorization": f"Bearer {os.environ['API_TOKEN']}"}

# === PUT — replace entire resource ===
response = requests.put(
    f"{API}/deployments/billing-api",
    json={"version": "1.2.3", "replicas": 3, "env": "production"},
    headers=headers,
)

# === PATCH — partial update ===
response = requests.patch(
    f"{API}/deployments/billing-api",
    json={"replicas": 5},    # Only update replicas
    headers=headers,
)

# === DELETE — remove resource ===
response = requests.delete(
    f"{API}/deployments/billing-api-canary",
    headers=headers,
)
if response.status_code == 204:
    print("✅ Deleted")
```

---

### 3.4 Authentication ⭐⭐

```python
import requests
import os

# === Bearer token (most common for APIs) ===
response = requests.get(
    "https://api.github.com/user",
    headers={"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
)

# === Basic auth ===
response = requests.get(
    "https://jenkins.example.com/api/json",
    auth=("admin", os.environ["JENKINS_PASSWORD"]),    # Tuple!
)

# === API key in header ===
response = requests.get(
    "https://api.datadog.com/api/v1/metrics",
    headers={
        "DD-API-KEY": os.environ["DD_API_KEY"],
        "DD-APPLICATION-KEY": os.environ["DD_APP_KEY"],
    },
)

# === Session — reuse auth across requests ===
session = requests.Session()
session.headers.update({
    "Authorization": f"Bearer {os.environ['API_TOKEN']}",
    "Accept": "application/json",
})

# All requests through this session share the headers!
response1 = session.get(f"{API}/pods")
response2 = session.get(f"{API}/services")
response3 = session.post(f"{API}/deployments", json=deploy_data)
```

---

### 3.5 Error Handling ⭐⭐⭐

```python
import requests
import time

# === Check status codes ===
response = requests.get("https://api.example.com/health")

if response.ok:                         # True for 2xx
    print("✅ Service is healthy")
elif response.status_code == 404:
    print("⚠️ Endpoint not found")
elif response.status_code == 401:
    print("🔒 Authentication failed")
elif response.status_code >= 500:
    print(f"🔥 Server error: {response.status_code}")

# === raise_for_status() — raise on 4xx/5xx ===
try:
    response = requests.get("https://api.example.com/data")
    response.raise_for_status()    # Raises HTTPError for 4xx/5xx
    data = response.json()
except requests.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.ConnectionError:
    print("Cannot connect — is the service running?")
except requests.Timeout:
    print("Request timed out!")
except requests.RequestException as e:
    print(f"Request failed: {e}")

# === Retry logic ===
def api_call_with_retry(url, max_retries=3, backoff=2, **kwargs):
    """GET with exponential backoff retry."""
    for attempt in range(1, max_retries + 1):
        try:
            response = requests.get(url, timeout=10, **kwargs)
            response.raise_for_status()
            return response
        except (requests.ConnectionError, requests.Timeout, requests.HTTPError) as e:
            if attempt == max_retries:
                raise
            wait = backoff ** attempt
            print(f"  ⚠️ Attempt {attempt} failed: {e}. Retrying in {wait}s...")
            time.sleep(wait)
```

---

### 3.6 Timeouts — Never Hang ⭐⭐

```python
import requests

# === Always set timeouts! ===
# timeout = (connect_timeout, read_timeout)
response = requests.get(
    "https://api.example.com/data",
    timeout=(5, 30),    # 5s to connect, 30s to read
)

# Single value = both
response = requests.get("https://api.example.com/data", timeout=10)

# ❌ NEVER omit timeout in production!
# requests.get(url)    # Waits FOREVER if server is unresponsive!
```

---

### 3.7 Health Checks — Monitoring Services ⭐⭐

```python
import requests
import time
from datetime import datetime

def health_check(services):
    """Check health of multiple services."""
    print(f"\n🏥 Health Check — {datetime.now():%Y-%m-%d %H:%M:%S}")
    print(f"{'Service':<30} {'Status':>8} {'Time':>8}")
    print(f"{'─'*30} {'─'*8} {'─'*8}")
    
    results = []
    for name, url in services.items():
        try:
            start = time.perf_counter()
            r = requests.get(url, timeout=5)
            elapsed = (time.perf_counter() - start) * 1000
            
            status = "✅ UP" if r.ok else f"⚠️ {r.status_code}"
            results.append({"name": name, "up": r.ok, "ms": elapsed})
        except requests.ConnectionError:
            status = "❌ DOWN"
            elapsed = 0
            results.append({"name": name, "up": False, "ms": 0})
        except requests.Timeout:
            status = "⏰ TIMEOUT"
            elapsed = 5000
            results.append({"name": name, "up": False, "ms": 5000})
        
        print(f"  {name:<28} {status:>8} {elapsed:>6.0f}ms")
    
    up_count = sum(1 for r in results if r["up"])
    print(f"\n  {up_count}/{len(results)} services healthy")
    return results

# Usage
services = {
    "Billing API": "http://billing-api:8080/health",
    "User Service": "http://user-svc:3000/healthz",
    "Database": "http://postgres:5432",
    "Redis Cache": "http://redis:6379",
}
# health_check(services)
```

---

### 3.8 Downloading Files ⭐

```python
import requests
from pathlib import Path

def download_file(url, destination, chunk_size=8192):
    """Download a file with progress."""
    response = requests.get(url, stream=True, timeout=60)
    response.raise_for_status()
    
    total = int(response.headers.get("content-length", 0))
    downloaded = 0
    
    dest = Path(destination)
    dest.parent.mkdir(parents=True, exist_ok=True)
    
    with open(dest, "wb") as f:
        for chunk in response.iter_content(chunk_size=chunk_size):
            f.write(chunk)
            downloaded += len(chunk)
            if total:
                pct = downloaded / total * 100
                print(f"\r  Downloading: {pct:.1f}%", end="", flush=True)
    
    print(f"\n  ✅ Downloaded: {dest} ({dest.stat().st_size:,} bytes)")

# download_file(
#     "https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip",
#     "/tmp/terraform.zip"
# )
```

---

### 3.9 Webhook Receiver (Bonus — FastAPI) ⭐

```python
# pip install fastapi uvicorn

from fastapi import FastAPI, Request
import json

app = FastAPI()

@app.post("/webhook/github")
async def github_webhook(request: Request):
    """Receive GitHub webhooks for auto-deployment."""
    payload = await request.json()
    event = request.headers.get("X-GitHub-Event", "unknown")
    
    if event == "push":
        branch = payload["ref"].split("/")[-1]
        commit = payload["head_commit"]["message"]
        
        if branch == "main":
            print(f"🚀 Deploy triggered! Commit: {commit}")
            # trigger_deployment()
            return {"status": "deploying"}
    
    return {"status": "ignored", "event": event}

# Run: uvicorn webhook:app --port 8000
```

---

### 3.10 Solving the Firm's Problem — API Integration Hub

```python
# ============================================
# Pearson Specter Litt — API Integration Hub
# Monitor, deploy, and notify — all via APIs
# ============================================

import requests
import os
import time
from datetime import datetime

class APIHub:
    """Central hub for all API integrations."""
    
    def __init__(self):
        self.session = requests.Session()
        self.session.headers.update({"User-Agent": "PSL-DeployBot/1.0"})
    
    def github_latest_release(self, repo):
        """Get latest release from GitHub."""
        r = self.session.get(
            f"https://api.github.com/repos/{repo}/releases/latest",
            headers={"Authorization": f"Bearer {os.environ.get('GITHUB_TOKEN', '')}"},
            timeout=10,
        )
        r.raise_for_status()
        data = r.json()
        return {"tag": data["tag_name"], "name": data["name"], "date": data["published_at"]}
    
    def slack_notify(self, message, channel="#deployments"):
        """Send a Slack notification."""
        webhook_url = os.environ.get("SLACK_WEBHOOK_URL")
        if not webhook_url:
            print(f"  [SLACK] {message}")
            return
        
        r = self.session.post(
            webhook_url,
            json={"text": message, "channel": channel},
            timeout=5,
        )
        return r.ok
    
    def health_check(self, endpoints):
        """Check multiple endpoints."""
        results = {}
        for name, url in endpoints.items():
            try:
                r = self.session.get(url, timeout=5)
                results[name] = {"status": r.status_code, "healthy": r.ok}
            except requests.RequestException as e:
                results[name] = {"status": 0, "healthy": False, "error": str(e)}
        return results

# Demo
print("=" * 55)
print(f"{'🏛️  API INTEGRATION HUB':^55}")
print("=" * 55)

hub = APIHub()

try:
    release = hub.github_latest_release("python/cpython")
    print(f"\n📦 Latest CPython: {release['tag']} ({release['name']})")
except requests.RequestException as e:
    print(f"\n⚠️ GitHub API: {e}")

print(f"{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: No Timeout 🔥
```python
# ❌ Hangs forever if server is down
requests.get("https://api.example.com")

# ✅ Always set timeout
requests.get("https://api.example.com", timeout=10)
```

### Pitfall 2: Not Checking Status 🔥
```python
# ❌ Assumes success
data = requests.get(url).json()    # Crashes on 500 error!

# ✅ Check first
r = requests.get(url, timeout=10)
r.raise_for_status()
data = r.json()
```

### Pitfall 3: `json=` vs `data=` Confusion 🔥
```python
# ❌ Sending dict as form data when API expects JSON
requests.post(url, data={"key": "value"})    # application/x-www-form-urlencoded

# ✅ Use json= for JSON APIs
requests.post(url, json={"key": "value"})    # application/json
```

### Pitfall 4: Hardcoded Tokens 🔥
```python
# ❌ Token in source code
headers = {"Authorization": "Bearer ghp_abc123xyz"}

# ✅ From environment
headers = {"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"}
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
REQUESTS = DONNA MAKING PHONE CALLS:
  GET    = "Can you send me that file?"
  POST   = "Here's the information you need."
  PUT    = "Replace what you have with this."
  PATCH  = "Just update this one thing."
  DELETE = "Cancel that."

  response.ok       = "Did they say yes?"
  response.json()   = "What exactly did they say?"
  timeout=10        = "If they don't answer in 10 seconds, hang up."
  raise_for_status()= "If they said something bad, I need to know NOW."

DONNA'S RULE:
  "Always set a timeout. Always check the response.
   Never put secrets in your Rolodex (source code)."
```

---

## 9. ✅ Knowledge Check

**Q1.** What is the `requests` library?
> A Python HTTP library for making web requests (GET, POST, etc.) to APIs.

**Q2.** How do you parse a JSON response?
> `response.json()` — returns a Python dict.

**Q3.** What does `raise_for_status()` do?
> Raises an `HTTPError` exception if the response status code is 4xx or 5xx.

**Q4.** Why always set `timeout`?
> Without it, requests can hang forever if the server is unresponsive.

**Q5.** What is the difference between `json=` and `data=`?
> `json=` serializes to JSON and sets Content-Type to application/json. `data=` sends as form-encoded.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | GET requests and query params | ✅ |
| 2 | POST with JSON and form data | ✅ |
| 3 | PUT, PATCH, DELETE | ✅ |
| 4 | Authentication (Bearer, Basic, API key) | ✅ |
| 5 | Error handling and status codes | ✅ |
| 6 | Timeouts and retries | ✅ |
| 7 | Health checks | ✅ |
| 8 | File downloads | ✅ |
| 9 | Sessions for connection reuse | ✅ |
| 10 | API integration hub | ✅ |

---

> **Next Lesson →** DevOps Lesson 4: *Parse and Generate YAML with PyYAML*
