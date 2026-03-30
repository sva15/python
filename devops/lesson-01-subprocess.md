# 🏛️ DevOps Module | Lesson 1: Run Shell Commands with subprocess

## *"The Corner Office Terminal"*

> **DevOps Objective:** *Run shell commands from Python, capture output, handle errors, and automate CLI tools like kubectl, docker, terraform, and git.*

> **Previously Learned:** Strings, f-strings, exception handling, file I/O, context managers

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

*The firm's DevOps team runs 30 commands every deployment — kubectl, docker, terraform, aws cli. They do it manually. It takes 45 minutes and someone always forgets a step.*

```bash
# ❌ Manual deployment — error-prone, slow
kubectl apply -f deployment.yaml
docker build -t billing:1.2.3 .
docker push registry.psl.com/billing:1.2.3
terraform plan
terraform apply -auto-approve
aws s3 sync ./dist s3://psl-static-assets
# ... 24 more commands
```

**Mike:** "Why are we typing these by hand? Python can run every one of these, check the output, handle errors, and retry if something fails."

```python
# ✅ Automated deployment script
import subprocess

result = subprocess.run(
    ["kubectl", "apply", "-f", "deployment.yaml"],
    capture_output=True, text=True, check=True,
)
print(result.stdout)    # Deployment applied!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "I don't do manual work. Neither should your deployments."

| Function | Purpose | Use When |
|----------|---------|----------|
| `subprocess.run()` | Run command, wait, return result | 95% of use cases ✅ |
| `subprocess.Popen()` | Run command with full control | Streaming output, piping |
| `os.system()` | Run command (basic) | ❌ Don't use — no output capture |

| Attribute | What It Contains |
|-----------|-----------------|
| `result.stdout` | Standard output (what the command prints) |
| `result.stderr` | Error output |
| `result.returncode` | Exit code (0 = success, non-zero = error) |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `subprocess.run()` — The Essential Function ⭐⭐⭐

```python
import subprocess

# === Basic command ===
result = subprocess.run(["echo", "Hello from Python"])
# Prints: Hello from Python
print(result.returncode)    # 0 (success)

# === Capture output ===
result = subprocess.run(
    ["python", "--version"],
    capture_output=True,    # Capture stdout AND stderr
    text=True,              # Return strings, not bytes
)
print(result.stdout)        # Python 3.12.0
print(result.stderr)        # (empty or version info)
print(result.returncode)    # 0

# === The command is a LIST of strings ===
# Each argument is a SEPARATE item in the list!

# ✅ Correct
subprocess.run(["ls", "-la", "/var/log"])
subprocess.run(["kubectl", "get", "pods", "-n", "production"])
subprocess.run(["docker", "build", "-t", "myapp:1.0", "."])

# ❌ Wrong — don't pass as a single string (without shell=True)
# subprocess.run(["ls -la /var/log"])    # Error!
```

---

### 3.2 Error Handling — `check=True` and return codes ⭐⭐⭐

```python
import subprocess

# === check=True — raise exception on failure ===
try:
    result = subprocess.run(
        ["kubectl", "apply", "-f", "deployment.yaml"],
        capture_output=True,
        text=True,
        check=True,    # ← Raises CalledProcessError if returncode != 0
    )
    print(f"✅ Applied: {result.stdout}")

except subprocess.CalledProcessError as e:
    print(f"❌ Command failed (exit code {e.returncode})")
    print(f"   Error: {e.stderr}")

# === Manual return code checking ===
result = subprocess.run(
    ["terraform", "plan", "-detailed-exitcode"],
    capture_output=True,
    text=True,
)

if result.returncode == 0:
    print("✅ No changes needed")
elif result.returncode == 2:
    print("📋 Changes detected:")
    print(result.stdout)
else:
    print(f"❌ Error: {result.stderr}")
```

---

### 3.3 Timeouts — Don't Hang Forever ⭐⭐

```python
import subprocess

# === Timeout = kill the command if it takes too long ===
try:
    result = subprocess.run(
        ["ping", "-c", "5", "10.0.0.1"],
        capture_output=True,
        text=True,
        timeout=10,    # Kill after 10 seconds
    )
    print(result.stdout)

except subprocess.TimeoutExpired:
    print("⏰ Command timed out! Server may be unreachable.")

# === Health check with timeout ===
def check_service(host, port, timeout=5):
    """Check if a service is reachable."""
    try:
        result = subprocess.run(
            ["curl", "-s", "-o", "/dev/null", "-w", "%{http_code}",
             f"http://{host}:{port}/health"],
            capture_output=True, text=True, timeout=timeout,
        )
        return result.stdout.strip() == "200"
    except (subprocess.TimeoutExpired, FileNotFoundError):
        return False
```

---

### 3.4 `shell=True` — When and When NOT To Use ⭐⭐

```python
import subprocess

# === shell=True lets you use shell features ===
# Pipes, wildcards, environment variable expansion

# Piping (command1 | command2)
result = subprocess.run(
    "docker ps | grep nginx | wc -l",
    shell=True,
    capture_output=True,
    text=True,
)
print(f"Nginx containers: {result.stdout.strip()}")

# Wildcards
result = subprocess.run(
    "ls *.yaml",
    shell=True,
    capture_output=True,
    text=True,
)

# ⚠️ SECURITY WARNING ⚠️
# NEVER use shell=True with USER INPUT!

# ❌ DANGEROUS — command injection!
user_input = "file.txt; rm -rf /"    # Malicious input!
subprocess.run(f"cat {user_input}", shell=True)    # Deletes everything!

# ✅ SAFE — use list form (no shell)
subprocess.run(["cat", user_input])    # Treats entire input as filename

# RULE: shell=True only with YOUR OWN hardcoded strings
# NEVER with variables from users, APIs, or config files
```

---

### 3.5 Environment Variables ⭐⭐

```python
import subprocess
import os

# === Pass environment variables to commands ===
env = os.environ.copy()    # Start with current environment
env["DEPLOY_ENV"] = "staging"
env["AWS_REGION"] = "us-east-1"
env["VERSION"] = "1.2.3"

result = subprocess.run(
    ["./deploy.sh"],
    env=env,
    capture_output=True,
    text=True,
)

# === Use cwd to set working directory ===
result = subprocess.run(
    ["terraform", "init"],
    cwd="/infra/terraform/production",    # Run FROM this directory
    capture_output=True,
    text=True,
    check=True,
)
```

---

### 3.6 Streaming Output in Real-Time ⭐⭐

```python
import subprocess
import sys

# === subprocess.run captures ALL output, returns it at the end ===
# === Popen lets you read line-by-line in real time ===

def run_streaming(cmd, label=""):
    """Run a command and print output in real time."""
    print(f"🚀 Running: {' '.join(cmd)}")
    
    process = subprocess.Popen(
        cmd,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,    # Merge stderr into stdout
        text=True,
        bufsize=1,                   # Line buffered
    )
    
    output_lines = []
    for line in process.stdout:
        clean = line.rstrip()
        print(f"  [{label}] {clean}")
        output_lines.append(clean)
    
    process.wait()
    
    if process.returncode != 0:
        raise subprocess.CalledProcessError(process.returncode, cmd)
    
    return output_lines

# Usage: see docker build output in real time
# run_streaming(["docker", "build", "-t", "myapp:1.0", "."], label="docker")
```

---

### 3.7 Chaining Commands — Build a Pipeline ⭐⭐

```python
import subprocess
import sys

def run_step(name, cmd, **kwargs):
    """Run a deployment step with logging."""
    print(f"\n{'='*50}")
    print(f"📋 Step: {name}")
    print(f"   Command: {' '.join(cmd)}")
    print(f"{'='*50}")
    
    defaults = {"capture_output": True, "text": True, "check": True}
    defaults.update(kwargs)
    
    try:
        result = subprocess.run(cmd, **defaults)
        print(f"✅ {name} — SUCCESS")
        if result.stdout.strip():
            for line in result.stdout.strip().split('\n')[:5]:
                print(f"   {line}")
        return result
    except subprocess.CalledProcessError as e:
        print(f"❌ {name} — FAILED (exit code {e.returncode})")
        print(f"   Error: {e.stderr[:200]}")
        raise

# === Deployment pipeline ===
def deploy(version, environment):
    """Full deployment pipeline."""
    steps = [
        ("Run tests", ["pytest", "tests/", "-v"]),
        ("Build image", ["docker", "build", "-t", f"billing:{version}", "."]),
        ("Push image", ["docker", "push", f"registry.psl.com/billing:{version}"]),
        ("Update K8s", ["kubectl", "set", "image",
                        f"deployment/billing", f"billing=billing:{version}",
                        "-n", environment]),
        ("Check rollout", ["kubectl", "rollout", "status",
                          "deployment/billing", "-n", environment,
                          "--timeout=120s"]),
    ]
    
    completed = []
    try:
        for name, cmd in steps:
            run_step(name, cmd)
            completed.append(name)
    except subprocess.CalledProcessError:
        print(f"\n🔥 Pipeline failed at: {name}")
        print(f"   Completed steps: {completed}")
        print(f"   Rolling back...")
        # Rollback logic here
        sys.exit(1)
    
    print(f"\n🎉 Deployment complete! v{version} → {environment}")

# deploy("1.2.3", "staging")
```

---

### 3.8 Piping Between Commands (Without shell=True) ⭐

```python
import subprocess

# === Pipe safely without shell=True ===
# Equivalent to: cat access.log | grep "500" | wc -l

p1 = subprocess.Popen(
    ["cat", "/var/log/access.log"],
    stdout=subprocess.PIPE,
)

p2 = subprocess.Popen(
    ["grep", "500"],
    stdin=p1.stdout,        # Pipe p1's output into p2
    stdout=subprocess.PIPE,
)
p1.stdout.close()           # Allow p1 to receive SIGPIPE

p3 = subprocess.Popen(
    ["wc", "-l"],
    stdin=p2.stdout,
    stdout=subprocess.PIPE,
    text=True,
)
p2.stdout.close()

output, _ = p3.communicate()
print(f"500 errors: {output.strip()}")
```

---

### 3.9 Common DevOps Commands Automated ⭐

```python
import subprocess
import json

# === Git operations ===
def git_current_branch():
    result = subprocess.run(
        ["git", "branch", "--show-current"],
        capture_output=True, text=True, check=True,
    )
    return result.stdout.strip()

def git_last_commit():
    result = subprocess.run(
        ["git", "log", "-1", "--format=%H %s"],
        capture_output=True, text=True, check=True,
    )
    return result.stdout.strip()

# === Docker operations ===
def docker_running_containers():
    result = subprocess.run(
        ["docker", "ps", "--format", "{{json .}}"],
        capture_output=True, text=True, check=True,
    )
    containers = []
    for line in result.stdout.strip().split('\n'):
        if line:
            containers.append(json.loads(line))
    return containers

# === kubectl operations ===
def kubectl_get_pods(namespace="default"):
    result = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace, "-o", "json"],
        capture_output=True, text=True, check=True,
    )
    data = json.loads(result.stdout)
    return [
        {"name": p["metadata"]["name"], "status": p["status"]["phase"]}
        for p in data["items"]
    ]

# === AWS CLI ===
def aws_s3_list(bucket):
    result = subprocess.run(
        ["aws", "s3", "ls", f"s3://{bucket}", "--recursive", "--summarize"],
        capture_output=True, text=True, check=True,
    )
    return result.stdout
```

---

### 3.10 Solving the Firm's Problem — Deployment Automation

```python
# ============================================
# Pearson Specter Litt — Deployment Automator
# Replace 30 manual steps with one script
# ============================================

import subprocess
import sys
import time
from datetime import datetime

class Deployer:
    """Automated deployment pipeline."""
    
    def __init__(self, service, version, environment):
        self.service = service
        self.version = version
        self.environment = environment
        self.log = []
    
    def _run(self, cmd, label, timeout=120):
        """Run a command with logging."""
        start = time.perf_counter()
        self.log.append(f"[{datetime.now():%H:%M:%S}] START: {label}")
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                check=True,
                timeout=timeout,
            )
            elapsed = time.perf_counter() - start
            self.log.append(f"[{datetime.now():%H:%M:%S}] ✅ OK: {label} ({elapsed:.1f}s)")
            return result
        except subprocess.CalledProcessError as e:
            self.log.append(f"[{datetime.now():%H:%M:%S}] ❌ FAIL: {label}")
            self.log.append(f"    stderr: {e.stderr[:200]}")
            raise
        except subprocess.TimeoutExpired:
            self.log.append(f"[{datetime.now():%H:%M:%S}] ⏰ TIMEOUT: {label}")
            raise
    
    def deploy(self):
        """Execute full deployment."""
        print(f"\n{'='*55}")
        print(f"  🚀 Deploying {self.service} v{self.version} → {self.environment}")
        print(f"{'='*55}\n")
        
        try:
            self._run(
                ["python", "-m", "pytest", "tests/", "-q"],
                "Run tests"
            )
            self._run(
                ["docker", "build", "-t", f"{self.service}:{self.version}", "."],
                "Build image", timeout=300
            )
            self._run(
                ["docker", "push", f"registry.psl.com/{self.service}:{self.version}"],
                "Push image", timeout=120
            )
            print(f"\n🎉 SUCCESS — {self.service} v{self.version} deployed!\n")
            
        except (subprocess.CalledProcessError, subprocess.TimeoutExpired):
            print(f"\n🔥 FAILED — see log below\n")
            sys.exit(1)
        finally:
            print("📋 Deployment Log:")
            for entry in self.log:
                print(f"  {entry}")

# Usage:
# Deployer("billing-api", "1.2.3", "staging").deploy()
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: Using `os.system()` Instead of `subprocess` 🔥
```python
# ❌ os.system — no output capture, no error handling
import os
os.system("kubectl get pods")    # Returns exit code only!

# ✅ subprocess.run — full control
result = subprocess.run(["kubectl", "get", "pods"], capture_output=True, text=True)
```

### Pitfall 2: Command Injection with `shell=True` 🔥
```python
# ❌ User input in shell command
filename = input("File to read: ")
subprocess.run(f"cat {filename}", shell=True)    # "file.txt; rm -rf /" !

# ✅ Use list form
subprocess.run(["cat", filename])    # Safe!
```

### Pitfall 3: Forgetting `text=True` 🔥
```python
# ❌ Returns bytes — confusing!
result = subprocess.run(["echo", "hello"], capture_output=True)
print(result.stdout)    # b'hello\n'  ← bytes!

# ✅ text=True returns strings
result = subprocess.run(["echo", "hello"], capture_output=True, text=True)
print(result.stdout)    # hello\n  ← string!
```

### Pitfall 4: Not Checking Return Codes 🔥
```python
# ❌ Command failed but script continues!
subprocess.run(["deploy", "--prod"])    # Failed silently!
subprocess.run(["notify", "success"])   # Notifies success despite failure!

# ✅ check=True stops on error
subprocess.run(["deploy", "--prod"], check=True)    # Raises exception!
```

### Pitfall 5: Blocking on Long Commands 🔥
```python
# ❌ Script hangs waiting for a command that takes forever
subprocess.run(["ping", "10.0.0.1"])    # Runs forever!

# ✅ Always set timeout for network/remote commands
subprocess.run(["ping", "-c", "3", "10.0.0.1"], timeout=10)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
SUBPROCESS = DONNA ON THE PHONE:
  She calls external people (shell commands),
  listens to their response (stdout),
  notes any problems (stderr),
  and reports the outcome (returncode).

subprocess.run()  = "Call this person, wait for an answer."
capture_output    = "Write down what they say."
check=True        = "If they say NO, escalate immediately."
timeout           = "If they don't pick up in 10 seconds, hang up."
shell=True        = "Let them transfer you around (pipes)."
                    ⚠️ Only for trusted contacts!

DONNA'S RULE:
  "Always capture the output. Always check the result.
   Never trust strangers with shell=True.
   And always set a timeout — nobody waits forever."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

### 🟢 Beginner
1. **System Info**: Run `python --version`, `pip --version`, and `git --version`. Print all three.
2. **Directory Listing**: Run `ls` (or `dir`) and capture the output. Count the files.
3. **Ping Test**: Ping a website with timeout. Print "UP" or "DOWN".

### 🟡 Intermediate
4. **Git Dashboard**: Show current branch, last 5 commits, and uncommitted changes.
5. **Multi-Step Pipeline**: Run 3 commands in sequence. Stop if any fails. Print summary.
6. **Docker Cleanup**: List stopped containers, remove them, report freed space.

### 🔴 Advanced
7. **Deployment Script**: Build a full deploy pipeline with tests → build → push → apply.
8. **Parallel Health Check**: Check 10 URLs concurrently using threads + subprocess.
9. **Log Streamer**: Stream docker logs in real time, filtering for ERROR lines.

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛
```python
subprocess.run("kubectl get pods -n production")              # Bug 1: string not list
result = subprocess.run(["kubectl", "apply", "-f", "x.yaml"]) # Bug 2: no error check
print(result.stdout)                                           # Bug 3: stdout not captured
subprocess.run(f"grep {user_input} log.txt", shell=True)       # Bug 4: injection risk
```

### Fixed Code ✅
```python
subprocess.run(["kubectl", "get", "pods", "-n", "production"])
result = subprocess.run(["kubectl", "apply", "-f", "x.yaml"],
    capture_output=True, text=True, check=True)
print(result.stdout)
subprocess.run(["grep", user_input, "log.txt"])
```

---

## 8. 🌍 Real-World Application

| Tool | subprocess Command |
|------|-------------------|
| **kubectl** | `["kubectl", "get", "pods", "-o", "json"]` |
| **docker** | `["docker", "build", "-t", "app:v1", "."]` |
| **terraform** | `["terraform", "plan", "-out=plan.tfplan"]` |
| **ansible** | `["ansible-playbook", "deploy.yml", "-i", "hosts"]` |
| **aws cli** | `["aws", "s3", "sync", "./dist", "s3://bucket"]` |
| **git** | `["git", "log", "--oneline", "-5"]` |
| **helm** | `["helm", "upgrade", "app", "./chart", "--install"]` |

---

## 9. ✅ Knowledge Check

**Q1.** Why use `subprocess.run()` instead of `os.system()`?
> It captures output, returns structured results, supports timeouts, and enables proper error handling.

**Q2.** What does `check=True` do?
> Raises `CalledProcessError` if the command exits with a non-zero return code.

**Q3.** Why pass commands as a list, not a string?
> Security (prevents injection) and correctness (proper argument splitting).

**Q4.** When is `shell=True` acceptable?
> Only with hardcoded strings you control — for pipes, wildcards, or shell built-ins. Never with user input.

**Q5.** What is `capture_output=True`?
> Captures both stdout and stderr so you can read them from `result.stdout` and `result.stderr`.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `subprocess.run()` basics | ✅ |
| 2 | Capturing output (`capture_output`, `text`) | ✅ |
| 3 | Error handling (`check`, return codes) | ✅ |
| 4 | Timeouts | ✅ |
| 5 | `shell=True` security | ✅ |
| 6 | Environment variables and `cwd` | ✅ |
| 7 | Streaming with `Popen` | ✅ |
| 8 | Command pipelines | ✅ |
| 9 | Common DevOps commands | ✅ |
| 10 | Deployment automation | ✅ |

---

> **Next Lesson →** DevOps Lesson 2: *Navigate the File System with os, pathlib, and shutil*
