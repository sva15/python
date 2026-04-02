# Phase 2 · Lesson 5 — HTTP & APIs with `requests`
### *"The Intelligence Network"*

> **DevOps Objective:** Make HTTP requests, call REST APIs, check service health, download files, and trigger webhooks using the requests library.
> **DevOps connection:** GitHub Actions, PagerDuty, Slack notifications, AWS APIs, Datadog, health checks — all REST APIs you call with requests.
> **10-min sprint:** Write a script that calls `https://httpbin.org/get` and prints the response status code, your public IP (it's in the JSON), and how long the request took.

---

## Why This Matters

Every modern service has a REST API. Your deploy script needs to: trigger a deployment via API, send a Slack alert when something breaks, check that services are healthy, query your monitoring system, update a ticket in Jira. All of that is `requests`.

---

## Setup

```bash
pip install requests
```

---

## The Core Concept

### GET Request — fetch data

```python
import requests

# Basic GET
response = requests.get("https://api.github.com/repos/sva15/python")

# Always check the status
print(response.status_code)    # 200
response.raise_for_status()    # raises HTTPError if 4xx or 5xx

# Parse JSON response
data = response.json()         # returns a Python dict
print(data["stargazers_count"])
print(data["description"])

# Access headers
print(response.headers["Content-Type"])
```

### GET with Parameters

```python
response = requests.get(
    "https://api.github.com/search/repositories",
    params={"q": "devops language:python", "sort": "stars", "per_page": 5},
    headers={"Authorization": f"token {os.environ['GITHUB_TOKEN']}"},
    timeout=10,    # always set a timeout!
)
response.raise_for_status()
results = response.json()

for repo in results["items"]:
    print(f"{repo['full_name']}: ⭐ {repo['stargazers_count']}")
```

### POST Request — send data

```python
import requests

# Trigger a deployment via API
response = requests.post(
    "https://api.example.com/deploy",
    json={"environment": "staging", "version": "1.2.3", "auto_approve": True},
    headers={
        "Authorization": f"Bearer {os.environ['DEPLOY_TOKEN']}",
        "Content-Type": "application/json",
    },
    timeout=30,
)
response.raise_for_status()
deploy_id = response.json()["deploy_id"]
print(f"Deploy started: {deploy_id}")
```

### `raise_for_status()` — always use it

```python
response = requests.get("https://api.example.com/clients")

# ❌ Don't check manually every time
if response.status_code == 200:
    data = response.json()
elif response.status_code == 404:
    print("Not found")
# ... etc.

# ✅ raise_for_status() handles it
try:
    response.raise_for_status()
    data = response.json()
except requests.HTTPError as e:
    print(f"HTTP error: {e.response.status_code} — {e.response.text}")
```

### Timeouts — always set one

```python
# Without timeout: your script can hang FOREVER if the server is slow
response = requests.get("http://api.example.com/health")   # ❌ no timeout!

# ✅ Always set both connect and read timeout
response = requests.get(
    "http://api.example.com/health",
    timeout=(5, 30)   # (connect_timeout, read_timeout) in seconds
)
# Or a single number for both:
response = requests.get(url, timeout=10)
```

### Authentication

```python
import os, requests

# Bearer token (most common for APIs)
headers = {"Authorization": f"Bearer {os.environ['API_TOKEN']}"}

# Basic auth (username/password)
response = requests.get(url, auth=("user", "password"))

# API key in header
headers = {"X-API-Key": os.environ["API_KEY"]}

# API key as query param
response = requests.get(url, params={"api_key": os.environ["API_KEY"]})
```

### Sessions — reuse connections for multiple requests

```python
import requests

# Session reuses the TCP connection and carries cookies/headers
session = requests.Session()
session.headers.update({
    "Authorization": f"Bearer {os.environ['API_TOKEN']}",
    "Accept": "application/json",
})

# All requests through this session use those headers
r1 = session.get("https://api.example.com/clients")
r2 = session.get("https://api.example.com/invoices")
r3 = session.post("https://api.example.com/deploy", json={"version": "1.2.3"})

session.close()   # or use as context manager:
with requests.Session() as session:
    session.headers.update({"Authorization": f"Bearer {token}"})
    response = session.get(url)
```

### Download a File

```python
import requests

def download_file(url: str, dest_path: str, chunk_size: int = 8192):
    """Stream a large file download without loading it all into memory."""
    with requests.get(url, stream=True, timeout=60) as response:
        response.raise_for_status()
        total = int(response.headers.get("Content-Length", 0))
        downloaded = 0

        with open(dest_path, "wb") as f:
            for chunk in response.iter_content(chunk_size=chunk_size):
                f.write(chunk)
                downloaded += len(chunk)
                if total:
                    pct = downloaded / total * 100
                    print(f"\r  {pct:.1f}% ({downloaded // 1024} KB)", end="")
        print()
    print(f"✅ Downloaded to {dest_path}")

download_file("https://releases.example.com/app-1.2.3.tar.gz", "app.tar.gz")
```

---

## DevOps Example — Health Check + Alerting

```python
import os
import time
import requests
import logging

logger = logging.getLogger(__name__)

def check_health(url: str, timeout: int = 5) -> tuple[bool, int, float]:
    """Check if a service is healthy. Returns (is_up, status_code, response_time)."""
    start = time.time()
    try:
        response = requests.get(f"{url}/health", timeout=timeout)
        elapsed = time.time() - start
        return response.status_code == 200, response.status_code, elapsed
    except requests.ConnectionError:
        return False, 0, time.time() - start
    except requests.Timeout:
        return False, 408, time.time() - start

def send_slack_alert(message: str, webhook_url: str = None):
    """Send a message to Slack via webhook."""
    webhook_url = webhook_url or os.environ.get("SLACK_WEBHOOK_URL")
    if not webhook_url:
        logger.warning("SLACK_WEBHOOK_URL not set — skipping alert")
        return

    try:
        response = requests.post(
            webhook_url,
            json={"text": message},
            timeout=10,
        )
        response.raise_for_status()
        logger.debug("Slack alert sent")
    except requests.RequestException as e:
        logger.error(f"Failed to send Slack alert: {e}")

def monitor_services(services: dict) -> dict:
    """Check all services and alert on failures."""
    results = {}
    down_services = []

    for name, url in services.items():
        is_up, code, elapsed = check_health(url)
        results[name] = {"up": is_up, "code": code, "ms": int(elapsed * 1000)}

        status = "✅" if is_up else "❌"
        logger.info(f"{status} {name}: HTTP {code} ({elapsed*1000:.0f}ms)")

        if not is_up:
            down_services.append(f"❌ {name} ({url}) — HTTP {code}")

    if down_services:
        alert = "🚨 *Services Down:*\n" + "\n".join(down_services)
        send_slack_alert(alert)

    return results

# Run the monitor
services = {
    "Billing API": "https://billing.psl.internal",
    "Client Portal": "https://portal.psl.internal",
    "Database Proxy": "http://db-proxy.internal:5432",
}
results = monitor_services(services)
```

---

## Retry Pattern for Flaky APIs

```python
import time, requests
from functools import wraps

def with_retry(max_attempts=3, delay=2, backoff=2, exceptions=(requests.RequestException,)):
    """Decorator: retry a function on specified exceptions."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            wait = delay
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts:
                        raise
                    logger.warning(f"Attempt {attempt}/{max_attempts} failed: {e}. Retrying in {wait}s...")
                    time.sleep(wait)
                    wait *= backoff
        return wrapper
    return decorator

@with_retry(max_attempts=3, delay=5)
def deploy_to_environment(version: str, env: str) -> dict:
    response = requests.post(
        f"https://deploy.api.example.com/v1/deploy",
        json={"version": version, "environment": env},
        headers={"Authorization": f"Bearer {os.environ['DEPLOY_TOKEN']}"},
        timeout=30,
    )
    response.raise_for_status()
    return response.json()
```

---

## Gotchas (Louis's warnings)

**1. Always set a timeout:**
```python
# ❌ Can hang forever
requests.get("http://slow-server.example.com")

# ✅ Will raise Timeout after 10 seconds
requests.get("http://slow-server.example.com", timeout=10)
```

**2. `raise_for_status()` must be called — status 200 is NOT automatic:**
```python
response = requests.get(url)
# Even if it returns 404 or 500, no exception is raised yet!
response.raise_for_status()   # NOW it raises for 4xx/5xx
```

**3. Secrets in headers, not URLs:**
```python
# ❌ Token visible in logs, proxy logs, browser history
requests.get("https://api.example.com?token=my_secret_token")

# ✅ Token in header (only visible in HTTPS body)
requests.get(url, headers={"Authorization": f"Bearer {token}"})
```

**4. Never hardcode tokens — use env vars:**
```python
# ❌ Secret in code = secret in git
TOKEN = "ghp_abc123..."

# ✅
TOKEN = os.environ["GITHUB_TOKEN"]
```

**5. Check for network errors separately from HTTP errors:**
```python
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()   # HTTP-level errors (404, 500)
    data = response.json()
except requests.ConnectionError:
    print("Cannot connect — is the service up?")
except requests.Timeout:
    print("Request timed out")
except requests.HTTPError as e:
    print(f"HTTP {e.response.status_code}: {e.response.text}")
except requests.JSONDecodeError:
    print("Response is not valid JSON")
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Call `https://httpbin.org/get` and print: status code, your public IP (it's in `response.json()["origin"]`), and response time in ms.

**Intermediate:** Write a `github_repo_info(owner, repo, token)` function. Fetch repo details from the GitHub API and return a dict with: name, description, stars, language, last push date, and whether it's public.

**DevOps-specific:** Write a Slack notifier. Function signature: `notify_slack(webhook_url, message, level="info")`. Color code the attachment: green for info, yellow for warning, red for error. Include a timestamp. Handle connection errors gracefully.

---

## Knowledge Check

1. What does `response.raise_for_status()` do?
2. Why should you always set a `timeout`?
3. What's the difference between `requests.get(url, params={})` and `requests.post(url, json={})`?
4. When would you use a `requests.Session()`?
5. How do you stream a large file download without loading it all into memory?

**Answers:** (1) Raises `HTTPError` if the response status is 4xx or 5xx. (2) Without it, the script can hang indefinitely waiting for a slow/dead server. (3) `params` adds query string to URL; `json` sends data in POST body as JSON. (4) When making multiple requests to the same host — reuses connections and shares headers/cookies. (5) `requests.get(url, stream=True)` then iterate `response.iter_content(chunk_size=...)`.

---

> **Next:** Lesson 6 — PyYAML → Parse K8s manifests, Ansible playbooks, and GitHub Actions configs.
