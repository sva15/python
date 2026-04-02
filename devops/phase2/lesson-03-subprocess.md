# Phase 2 · Lesson 3 — subprocess
### *"The Corner Office Terminal"*

> **DevOps Objective:** Run shell commands from Python, capture output, handle errors, and automate CLI tools like kubectl, docker, terraform, and git.
> **DevOps connection:** This IS DevOps with Python. Every manual shell command you type today can be automated, retried, logged, and chained into a pipeline.
> **10-min sprint:** Write a Python script that runs `git status` and prints the output. Then make it check if the working tree is clean, and print either "✅ clean" or "⚠️ uncommitted changes".

---

## Why This Matters (Harvey's take)

Your team runs 30 shell commands for every deployment — manually, in the right order, hoping nobody forgets a step. Python's `subprocess` module lets you run every one of those commands from a script: capture the output, check for errors, retry on failure, and chain them into a pipeline. That 45-minute manual deploy becomes a 5-minute automated one.

---

## The Core Concept (Mike's breakdown)

### `subprocess.run()` — your main tool (95% of use cases)

```python
import subprocess

# Run a command and capture everything
result = subprocess.run(
    ["kubectl", "get", "pods", "-n", "production"],
    capture_output=True,    # capture stdout and stderr
    text=True,              # return strings, not bytes
    check=True,             # raise exception if exit code != 0
)

print(result.stdout)        # command's output
print(result.stderr)        # error output
print(result.returncode)    # 0 = success, anything else = failure
```

**The command is a LIST — each argument is a separate item:**
```python
# ✅ Correct
subprocess.run(["ls", "-la", "/var/log"])
subprocess.run(["kubectl", "get", "pods", "-n", "production"])
subprocess.run(["docker", "build", "-t", "myapp:1.0", "."])

# ❌ Wrong — passing as single string without shell=True
subprocess.run(["ls -la /var/log"])   # treats entire string as one arg
```

### Error Handling

```python
import subprocess

# Method 1: check=True — raise exception on failure (recommended)
try:
    result = subprocess.run(
        ["kubectl", "apply", "-f", "deploy.yaml"],
        capture_output=True, text=True, check=True,
    )
    print(f"✅ {result.stdout.strip()}")

except subprocess.CalledProcessError as e:
    print(f"❌ Failed (exit {e.returncode}): {e.stderr.strip()}")

# Method 2: manual return code check
result = subprocess.run(["terraform", "plan"], capture_output=True, text=True)
if result.returncode == 0:
    print("✅ No changes")
else:
    print(f"❌ Error:\n{result.stderr}")
```

### Timeouts — never hang forever

```python
try:
    result = subprocess.run(
        ["curl", "-s", "http://api.example.com/health"],
        capture_output=True, text=True,
        timeout=10,    # kill if takes more than 10 seconds
    )
    print(result.stdout)

except subprocess.TimeoutExpired:
    print("⏰ Command timed out!")

except FileNotFoundError:
    print("❌ Command not found — is curl installed?")
```

### `shell=True` — use carefully

```python
# shell=True lets you use pipes, wildcards, environment variables
result = subprocess.run(
    "docker ps | grep nginx | wc -l",
    shell=True, capture_output=True, text=True,
)
print(f"Nginx containers running: {result.stdout.strip()}")

# ⚠️ SECURITY: NEVER use shell=True with user input
user_input = "file.txt; rm -rf /"    # malicious!
subprocess.run(f"cat {user_input}", shell=True)    # DANGEROUS!

# ✅ SAFE — always use list form with user data
subprocess.run(["cat", user_input])    # treats entire string as filename
```

**Rule:** `shell=True` only with your own hardcoded strings. Never with variables from users, APIs, or config files.

### Environment Variables and Working Directory

```python
import subprocess, os

# Pass environment variables
env = os.environ.copy()
env["DEPLOY_ENV"] = "staging"
env["VERSION"] = "1.2.3"

result = subprocess.run(["./deploy.sh"], env=env, capture_output=True, text=True)

# Set working directory
result = subprocess.run(
    ["terraform", "apply", "-auto-approve"],
    cwd="/infra/terraform/production",   # run from this directory
    capture_output=True, text=True, check=True,
)
```

### Streaming Output in Real Time

```python
import subprocess

def run_with_live_output(cmd, label="CMD"):
    """Print command output line-by-line as it arrives."""
    print(f"▶ Running: {' '.join(cmd)}")

    process = subprocess.Popen(
        cmd,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,   # merge stderr into stdout
        text=True,
    )

    for line in process.stdout:
        print(f"  [{label}] {line.rstrip()}")

    process.wait()
    return process.returncode

# Use when you want to see progress (e.g., docker build)
run_with_live_output(["docker", "build", "-t", "myapp:1.0", "."], "BUILD")
```

---

## DevOps Example — Full Deployment Pipeline

```python
import subprocess
import logging
import sys

logger = logging.getLogger("deploy")
logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")

def run_step(name, cmd, timeout=120, **kwargs):
    """Run one step of a deployment pipeline."""
    logger.info(f"{'='*50}")
    logger.info(f"Step: {name}")
    logger.info(f"Command: {' '.join(cmd)}")

    defaults = dict(capture_output=True, text=True, check=True, timeout=timeout)
    defaults.update(kwargs)

    try:
        result = subprocess.run(cmd, **defaults)
        logger.info(f"✅ {name} succeeded")
        if result.stdout.strip():
            logger.debug(result.stdout.strip())
        return result

    except subprocess.CalledProcessError as e:
        logger.error(f"❌ {name} failed (exit {e.returncode})")
        logger.error(f"   {e.stderr.strip()}")
        raise

    except subprocess.TimeoutExpired:
        logger.error(f"⏰ {name} timed out after {timeout}s")
        raise

def deploy(version, environment):
    """Run the full deployment pipeline."""
    steps = [
        ("Run Tests",       ["pytest", "tests/", "-q"]),
        ("Build Image",     ["docker", "build", "-t", f"billing:{version}", "."]),
        ("Push Image",      ["docker", "push", f"registry.psl.com/billing:{version}"]),
        ("Apply K8s",       ["kubectl", "set", "image", "deployment/billing",
                              f"billing=billing:{version}", "-n", environment]),
        ("Wait for Rollout",["kubectl", "rollout", "status",
                              "deployment/billing", "-n", environment, "--timeout=120s"]),
    ]

    completed = []
    try:
        for name, cmd in steps:
            run_step(name, cmd)
            completed.append(name)
        logger.info(f"🚀 Deployment of v{version} to {environment} complete!")

    except Exception as e:
        logger.error(f"Pipeline failed after: {completed}")
        logger.error(f"Triggering rollback...")
        subprocess.run(
            ["kubectl", "rollout", "undo", "deployment/billing", "-n", environment],
            capture_output=True, text=True,
        )
        sys.exit(1)

# Run it:
# deploy("1.2.3", "staging")
```

---

## Helper: `run()` wrapper for daily use

```python
import subprocess, logging

logger = logging.getLogger(__name__)

def run(cmd, timeout=60, check=True, **kwargs):
    """
    Simplified subprocess.run() wrapper.
    Returns (stdout, stderr, returncode) or raises on failure.
    """
    logger.debug(f"Running: {' '.join(cmd)}")
    try:
        result = subprocess.run(
            cmd, capture_output=True, text=True,
            timeout=timeout, check=check, **kwargs
        )
        return result.stdout.strip(), result.stderr.strip(), result.returncode

    except subprocess.CalledProcessError as e:
        logger.error(f"Command failed: {e.stderr.strip()}")
        raise
    except subprocess.TimeoutExpired:
        logger.error(f"Command timed out after {timeout}s")
        raise
    except FileNotFoundError:
        logger.error(f"Command not found: {cmd[0]}")
        raise

# Usage:
stdout, stderr, code = run(["kubectl", "get", "nodes"])
print(stdout)
```

---

## Gotchas (Louis's warnings)

**1. Always use list form — never shell string form without `shell=True`:**
```python
subprocess.run(["kubectl", "get", "pods"])    # ✅ correct
subprocess.run("kubectl get pods")            # ❌ treated as one argument
subprocess.run("kubectl get pods", shell=True)  # ✅ works but see warning above
```

**2. Capture output or it prints directly to your terminal:**
```python
subprocess.run(["kubectl", "get", "pods"])    # prints to terminal, can't use in code
result = subprocess.run(["kubectl", "get", "pods"], capture_output=True, text=True)
print(result.stdout)    # now you have it as a string
```

**3. `check=True` raises — don't forget to catch it:**
```python
result = subprocess.run(cmd, check=True)   # raises CalledProcessError on failure
# Always wrap in try-except if the command might fail
```

**4. `text=True` — otherwise you get bytes:**
```python
result = subprocess.run(["echo", "hello"], capture_output=True)
print(result.stdout)          # b'hello\n' — bytes!

result = subprocess.run(["echo", "hello"], capture_output=True, text=True)
print(result.stdout)          # 'hello\n' — string ✅
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Write a script that runs `git log --oneline -10` and prints the last 10 commits. If git isn't available or the directory isn't a repo, print a helpful error message instead of crashing.

**Intermediate:** Write a health checker that pings 5 services using `curl`. For each one, capture the HTTP status code. Print a summary: which are up (200), which are down, which timed out.

**DevOps-specific:** Write a `kubectl_status()` function that gets pod statuses in a namespace, parses the output, and returns a dict of `{pod_name: status}`. Handle the case where kubectl is not installed or the namespace doesn't exist.

---

## Knowledge Check

1. Why should you pass commands as a list, not a string?
2. What does `check=True` do?
3. What does `capture_output=True, text=True` give you?
4. When is `shell=True` safe to use? When is it dangerous?
5. What exception does `check=True` raise on failure?

**Answers:** (1) Each element is a separate argument — avoids shell injection and parsing issues. (2) Raises `CalledProcessError` if the command exits with non-zero code. (3) `stdout` and `stderr` as Python strings instead of bytes. (4) Safe with your own hardcoded strings; dangerous with any user-provided or externally-sourced data. (5) `subprocess.CalledProcessError`.

---

> **Next:** Lesson 4 — os/pathlib/shutil → Env vars, file paths, directory operations, and copying files — the filesystem toolkit.
