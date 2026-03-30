# 🔧 Python for DevOps Engineers — Concept Decoder

## *Your Curriculum → DevOps Automation*

> **Goal:** Understand which Python concepts DevOps engineers use daily, map them to your curriculum, and identify the frameworks and patterns that power infrastructure automation.

---

## 🧰 What DevOps Engineers Build with Python

| What | Example | Python Used |
|------|---------|-------------|
| **CLI tools** | Deploy scripts, health checks | argparse, click, typer |
| **Infrastructure automation** | Provision AWS/Azure/GCP resources | boto3, azure-sdk, google-cloud |
| **Config management** | Generate/validate YAML/JSON configs | PyYAML, json, Jinja2 |
| **CI/CD helpers** | Pipeline scripts, artefact builders | subprocess, os, shutil |
| **Monitoring & alerting** | Log parsers, health dashboards | regex, requests, logging |
| **API automation** | Webhook handlers, REST API calls | requests, FastAPI, Flask |
| **Container orchestration** | Docker/K8s management | docker-py, kubernetes-client |
| **SSH automation** | Remote server management | paramiko, fabric |
| **Testing infrastructure** | Validate Terraform, test deployments | pytest, testinfra |
| **Glue scripts** | Connect systems, migrate data | json, csv, subprocess |

---

## 🏗️ Key DevOps Frameworks & Libraries

| Framework | What It Does | Install |
|-----------|-------------|---------|
| **boto3** | AWS SDK — EC2, S3, Lambda, IAM, everything AWS | `pip install boto3` |
| **requests** | HTTP calls — REST APIs, webhooks, health checks | `pip install requests` |
| **PyYAML** | Parse/generate YAML (K8s manifests, Ansible, configs) | `pip install pyyaml` |
| **Click / Typer** | Build professional CLI tools | `pip install click` / `typer` |
| **Jinja2** | Templating — generate configs from templates | `pip install jinja2` |
| **paramiko / fabric** | SSH into servers, run remote commands | `pip install fabric` |
| **docker** | Docker SDK — build, run, manage containers | `pip install docker` |
| **kubernetes** | K8s Python client — pods, deployments, services | `pip install kubernetes` |
| **FastAPI** | Build webhook receivers, internal APIs | `pip install fastapi` |
| **pytest / testinfra** | Test infrastructure, validate deployments | `pip install pytest testinfra` |

---

## 📋 Priority Learning Path

### 🔴 Must-Know (Daily DevOps Work)
| Priority | Lesson | Why It Matters for DevOps |
|:--------:|--------|--------------------------|
| 1 | [L1-07: Dictionaries](https://github.com/sva15/python/blob/main/level-1/lesson-07-dictionaries.md) | API responses, configs, cloud resource tags — all dicts |
| 2 | [L2-08: File I/O](https://github.com/sva15/python/blob/main/level-2/lesson-08-file-io.md) | Reading configs, writing logs, processing files |
| 3 | [L1-09: Functions](https://github.com/sva15/python/blob/main/level-1/lesson-09-functions.md) | Every script is built from functions |
| 4 | [L2-02: String Formatting](https://github.com/sva15/python/blob/main/level-2/lesson-02-string-formatting.md) | Building commands, log messages, config values |
| 5 | [L3-05: Exception Handling](https://github.com/sva15/python/blob/main/level-3/lesson-05-exception-handling.md) | Robust scripts that don't crash at 3 AM |
| 6 | [L5-03: CSV/JSON](https://github.com/sva15/python/blob/main/level-5/lesson-03-csv-json.md) | Parsing API responses, config files, data exports |
| 7 | [L4-08: Regex](https://github.com/sva15/python/blob/main/level-4/lesson-08-regex.md) | Log parsing, pattern matching, text extraction |
| 8 | [L3-10: Logging](https://github.com/sva15/python/blob/main/level-3/lesson-10-logging-module.md) | Proper logging for scripts running in production |

### 🟡 Important (Weekly Use)
| Priority | Lesson | Why |
|:--------:|--------|-----|
| 9 | [L1-06: Loops](https://github.com/sva15/python/blob/main/level-1/lesson-06-loops.md) | Iterating over servers, resources, log lines |
| 10 | [L1-04: Lists](https://github.com/sva15/python/blob/main/level-1/lesson-04-lists.md) | Lists of instances, pods, files to process |
| 11 | [L3-09: Context Managers](https://github.com/sva15/python/blob/main/level-3/lesson-09-context-managers.md) | Safe file/connection handling (`with` statement) |
| 12 | [L2-12: Virtual Envs & pip](https://github.com/sva15/python/blob/main/level-2/lesson-12-virtual-environments-pip.md) | Managing Python dependencies on servers |
| 13 | [L3-11: Unit Testing](https://github.com/sva15/python/blob/main/level-3/lesson-11-unit-testing.md) | Testing infrastructure code, deployment validators |
| 14 | [L2-14: Type Hints](https://github.com/sva15/python/blob/main/level-2/lesson-14-type-hints.md) | Maintainable scripts others can understand |
| 15 | [L1-11: Classes](https://github.com/sva15/python/blob/main/level-1/lesson-11-define-classes.md) | AWS clients, K8s resources, config objects |
| 16 | [L5-09: Packaging](https://github.com/sva15/python/blob/main/level-5/lesson-09-packaging-publishing.md) | Distributing internal CLI tools |

### 🟢 Helpful (As Needed)
| Priority | Lesson | Why |
|:--------:|--------|-----|
| 17 | [L4-01: Decorators](https://github.com/sva15/python/blob/main/level-4/lesson-01-decorators.md) | Retry logic, rate limiting, caching |
| 18 | [L2-05: *args/**kwargs](https://github.com/sva15/python/blob/main/level-2/lesson-05-args-kwargs.md) | Wrapper functions, flexible APIs |
| 19 | [L5-05: Threading/Async](https://github.com/sva15/python/blob/main/level-5/lesson-05-threading-async.md) | Parallel operations (multi-server, concurrent APIs) |
| 20 | [L2-03: Comprehensions](https://github.com/sva15/python/blob/main/level-2/lesson-03-comprehensions.md) | Filtering resources, transforming data |
| 21 | [L3-01: Generators](https://github.com/sva15/python/blob/main/level-3/lesson-01-generator-functions.md) | Processing large log files without memory issues |
| 22 | [L1-03: Conditionals](https://github.com/sva15/python/blob/main/level-1/lesson-03-booleans-conditionals.md) | Decision logic in deployment scripts |

---

## 🔧 Python Concepts → DevOps Code Patterns

### Pattern 1: `subprocess` — Running Shell Commands ⭐⭐⭐

> ⚠️ **Not covered in curriculum** — this is a critical DevOps topic!

```python
import subprocess

# === Run a command and get output ===
result = subprocess.run(
    ["kubectl", "get", "pods", "-n", "production"],
    capture_output=True,     # Capture stdout/stderr
    text=True,               # Return strings, not bytes
    check=True,              # Raise exception on error
)
print(result.stdout)

# === Run with shell=True (when you need pipes) ===
result = subprocess.run(
    "docker ps | grep nginx",
    shell=True,              # ⚠️ Only with trusted input!
    capture_output=True,
    text=True,
)

# === Check return code ===
result = subprocess.run(["terraform", "plan"], capture_output=True, text=True)
if result.returncode != 0:
    print(f"ERROR: {result.stderr}")
else:
    print(f"Plan output:\n{result.stdout}")

# === Run a command with timeout ===
try:
    result = subprocess.run(
        ["ping", "-c", "3", "10.0.0.1"],
        timeout=10,           # Kill after 10 seconds
        capture_output=True,
        text=True,
    )
except subprocess.TimeoutExpired:
    print("Server unreachable!")

# === Common DevOps uses ===
# subprocess.run(["terraform", "apply", "-auto-approve"])
# subprocess.run(["ansible-playbook", "deploy.yml"])
# subprocess.run(["docker", "build", "-t", "myapp:latest", "."])
# subprocess.run(["aws", "s3", "sync", "./dist", "s3://my-bucket"])
```

---

### Pattern 2: `os` and `pathlib` — File System Operations ⭐⭐⭐

> ⚠️ **Only partially covered** — File I/O lesson covers `open()`, not filesystem navigation.

```python
import os
from pathlib import Path
import shutil

# === os module ===
# Environment variables (CRITICAL for DevOps — secrets, config)
aws_region = os.environ.get("AWS_REGION", "us-east-1")
db_password = os.environ["DB_PASSWORD"]    # Raises KeyError if missing

os.environ["DEPLOY_ENV"] = "staging"       # Set for child processes

# Check if file/dir exists
os.path.exists("/etc/nginx/nginx.conf")
os.path.isfile("config.yaml")
os.path.isdir("/var/log/app")

# List directory contents
for f in os.listdir("/var/log"):
    print(f)

# Walk directory tree
for root, dirs, files in os.walk("/app/configs"):
    for file in files:
        if file.endswith(".yaml"):
            full_path = os.path.join(root, file)
            print(full_path)

# === pathlib (modern, preferred) ===
config_dir = Path("/app/configs")
log_dir = Path("/var/log/app")

# Create directories
log_dir.mkdir(parents=True, exist_ok=True)

# Find files
yaml_files = list(config_dir.glob("**/*.yaml"))    # Recursive!

# Read/write (cleaner than open())
content = Path("config.yaml").read_text()
Path("output.json").write_text('{"status": "ok"}')

# Path manipulation
p = Path("/var/log/app/error.log")
print(p.name)       # error.log
print(p.stem)       # error
print(p.suffix)     # .log
print(p.parent)     # /var/log/app

# === shutil — copy, move, archive ===
shutil.copy("app.conf", "/etc/nginx/app.conf")
shutil.copytree("configs/", "/deploy/configs/")
shutil.rmtree("/tmp/build")    # ⚠️ Deletes recursively!
shutil.make_archive("backup", "tar", "/var/data")
```

---

### Pattern 3: `requests` — HTTP & API Calls ⭐⭐⭐

> ⚠️ **Not covered in curriculum** — DevOps engineers call APIs constantly.

```python
import requests
import json

# === GET request ===
response = requests.get(
    "https://api.github.com/repos/sva15/python",
    headers={"Authorization": f"token {os.environ['GITHUB_TOKEN']}"},
    timeout=10,
)
response.raise_for_status()    # Raise exception on 4xx/5xx
data = response.json()         # Parse JSON response (returns dict!)
print(f"Stars: {data['stargazers_count']}")

# === POST request (trigger a deployment) ===
response = requests.post(
    "https://api.example.com/deploy",
    json={"environment": "staging", "version": "1.2.3"},
    headers={"Authorization": "Bearer my-token"},
    timeout=30,
)

# === Health check ===
def check_health(url, timeout=5):
    try:
        r = requests.get(f"{url}/health", timeout=timeout)
        return r.status_code == 200
    except requests.ConnectionError:
        return False
    except requests.Timeout:
        return False

services = ["http://app:8080", "http://api:3000", "http://db:5432"]
for svc in services:
    status = "✅ UP" if check_health(svc) else "❌ DOWN"
    print(f"  {svc}: {status}")

# === Download a file ===
response = requests.get("https://releases.example.com/app-1.2.3.tar.gz", stream=True)
with open("app-1.2.3.tar.gz", "wb") as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)
```

---

### Pattern 4: PyYAML — Config File Parsing ⭐⭐⭐

> ⚠️ **Not covered in curriculum** — YAML is the language of DevOps (K8s, Ansible, CI/CD).

```python
import yaml

# === Read YAML ===
# config.yaml:
# app:
#   name: billing-service
#   replicas: 3
#   env:
#     - name: DB_HOST
#       value: postgres.prod.svc
#     - name: DB_PORT
#       value: "5432"

with open("config.yaml") as f:
    config = yaml.safe_load(f)    # Returns a dict!

print(config["app"]["name"])           # billing-service
print(config["app"]["replicas"])       # 3
print(config["app"]["env"][0]["value"])  # postgres.prod.svc

# === Write YAML ===
deployment = {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {"name": "billing-service", "namespace": "production"},
    "spec": {
        "replicas": 3,
        "template": {
            "spec": {
                "containers": [{
                    "name": "app",
                    "image": "billing:1.2.3",
                    "ports": [{"containerPort": 8080}],
                }]
            }
        }
    }
}

with open("deployment.yaml", "w") as f:
    yaml.dump(deployment, f, default_flow_style=False)

# === Multi-document YAML ===
with open("k8s-manifests.yaml") as f:
    docs = list(yaml.safe_load_all(f))    # Multiple --- separated docs

# KEY INSIGHT:
# YAML → Python dict (just like JSON!)
# yaml.safe_load() = json.loads() but for YAML
# DevOps config files (K8s, Docker Compose, Ansible, GitHub Actions) are ALL YAML
```

---

### Pattern 5: Jinja2 — Config Templating ⭐⭐

> ⚠️ **Not covered in curriculum** — Ansible, Helm, and many DevOps tools use Jinja2.

```python
from jinja2 import Template, Environment, FileSystemLoader

# === Basic templating ===
template_str = """
server {
    listen {{ port }};
    server_name {{ domain }};
    
    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
    
    {% for redirect in redirects %}
    location {{ redirect.from }} {
        return 301 {{ redirect.to }};
    }
    {% endfor %}
}
"""

template = Template(template_str)
nginx_conf = template.render(
    port=443,
    domain="app.example.com",
    backend_host="10.0.1.50",
    backend_port=8080,
    redirects=[
        {"from": "/old", "to": "/new"},
        {"from": "/legacy", "to": "/v2"},
    ]
)
print(nginx_conf)

# === From files (typical DevOps pattern) ===
# templates/deployment.yaml.j2:
#   replicas: {{ replicas }}
#   image: {{ image }}:{{ tag }}

env = Environment(loader=FileSystemLoader("templates"))
template = env.get_template("deployment.yaml.j2")
output = template.render(replicas=3, image="billing", tag="1.2.3")

# KEY INSIGHT:
# {{ variable }} → insert value (like f-strings)
# {% for %} / {% if %} → logic
# Ansible playbooks, Helm charts, and many CI configs use Jinja2
# If you know f-strings (L2-02), Jinja2 is the same idea for files
```

---

### Pattern 6: `boto3` — AWS Automation ⭐⭐⭐

```python
import boto3

# === S3: File operations ===
s3 = boto3.client("s3")                          # Instantiate client (L1-13)

# Upload
s3.upload_file("build.zip", "my-bucket", "deploys/build.zip")

# Download
s3.download_file("my-bucket", "configs/app.yaml", "local-app.yaml")

# List objects
response = s3.list_objects_v2(Bucket="my-bucket", Prefix="logs/")
for obj in response.get("Contents", []):          # Dict access (L1-07)!
    print(f"  {obj['Key']}: {obj['Size']} bytes")

# === EC2: Server management ===
ec2 = boto3.resource("ec2")

# List running instances
for instance in ec2.instances.filter(                    # Generator (L3-01)!
    Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
):
    print(f"  {instance.id}: {instance.instance_type} - {instance.public_ip_address}")

# Start/stop instances
ec2_client = boto3.client("ec2")
ec2_client.stop_instances(InstanceIds=["i-1234567890"])

# === Lambda: Invoke function ===
lambda_client = boto3.client("lambda")
response = lambda_client.invoke(
    FunctionName="process-billing",
    Payload=json.dumps({"client": "Wayne", "hours": 10}),   # JSON (L5-03)!
)
result = json.loads(response["Payload"].read())

# KEY INSIGHT:
# boto3 returns DICTS everywhere — your dict skills (L1-07) are critical!
# Every AWS service = a boto3 client with methods
# Pattern: client = boto3.client("service") → client.operation(**params)
```

---

### Pattern 7: Click/argparse — Building CLI Tools ⭐⭐

```python
# === argparse (standard library — covered briefly in L5-09) ===
import argparse

parser = argparse.ArgumentParser(description="Deploy application")
parser.add_argument("environment", choices=["dev", "staging", "prod"])
parser.add_argument("--version", required=True, help="Version to deploy")
parser.add_argument("--dry-run", action="store_true", help="Simulate only")
parser.add_argument("--replicas", type=int, default=3)

args = parser.parse_args()
print(f"Deploying v{args.version} to {args.environment} ({args.replicas} replicas)")

# === Click (popular DevOps choice — cleaner than argparse) ===
# pip install click

import click

@click.command()                                # Decorator (L4-01)!
@click.argument("environment", type=click.Choice(["dev", "staging", "prod"]))
@click.option("--version", "-v", required=True, help="Version to deploy")
@click.option("--dry-run", is_flag=True, help="Simulate only")
@click.option("--replicas", "-r", default=3, type=int)
def deploy(environment, version, dry_run, replicas):
    """Deploy application to target environment."""
    click.echo(f"🚀 Deploying v{version} to {environment}")
    
    if dry_run:
        click.echo("  [DRY RUN] No changes made")
        return
    
    # Actual deployment logic here
    click.echo(f"  ✅ Deployed with {replicas} replicas")

if __name__ == "__main__":
    deploy()

# Usage: python deploy.py staging --version 1.2.3 --dry-run
```

---

### Pattern 8: Exception Handling — Robust Automation ⭐⭐⭐

**Lesson reference:** [Exception Handling](https://github.com/sva15/python/blob/main/level-3/lesson-05-exception-handling.md)

```python
import time
import requests
import functools

# === Retry decorator (L4-01: Decorators + L3-05: Exceptions) ===
def retry(max_attempts=3, delay=2, exceptions=(Exception,)):
    """Retry a function on failure — critical for flaky infrastructure!"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts:
                        raise
                    print(f"  ⚠️ Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=5, exceptions=(requests.ConnectionError, requests.Timeout))
def deploy_service(version):
    """Deploy with automatic retry on network errors."""
    response = requests.post(
        "https://deploy.internal/api/deploy",
        json={"version": version},
        timeout=30,
    )
    response.raise_for_status()
    return response.json()

# === Cleanup on failure (L3-09: Context Managers) ===
class DeploymentContext:
    """Rollback on failure."""
    def __init__(self, service, version):
        self.service = service
        self.version = version
        self.previous_version = None
    
    def __enter__(self):
        self.previous_version = get_current_version(self.service)
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            print(f"  ❌ Deployment failed! Rolling back to {self.previous_version}")
            rollback(self.service, self.previous_version)
        return False    # Don't suppress the exception

# with DeploymentContext("billing-api", "1.2.3"):
#     deploy("billing-api", "1.2.3")    # If this fails → auto rollback!
```

---

### Pattern 9: Log Parsing with Regex ⭐⭐

**Lesson reference:** [Regex](https://github.com/sva15/python/blob/main/level-4/lesson-08-regex.md)

```python
import re
from collections import Counter

# Parse Nginx access logs
# Format: 127.0.0.1 - - [29/Mar/2026:10:00:00 +0000] "GET /api/health HTTP/1.1" 200 13

LOG_PATTERN = re.compile(
    r'(?P<ip>\S+) .+ \[(?P<date>[^\]]+)\] "(?P<method>\S+) (?P<path>\S+) \S+" (?P<status>\d+) (?P<size>\d+)'
)

def parse_logs(filepath):
    """Parse access logs and return statistics."""
    status_counts = Counter()
    path_counts = Counter()
    error_ips = set()
    
    with open(filepath) as f:
        for line in f:                          # Generator-style (L3-01)!
            match = LOG_PATTERN.match(line)
            if match:
                d = match.groupdict()           # Returns a dict (L1-07)!
                status = int(d["status"])
                status_counts[status] += 1
                path_counts[d["path"]] += 1
                
                if status >= 500:
                    error_ips.add(d["ip"])
    
    return {
        "status_codes": dict(status_counts.most_common(10)),
        "top_paths": dict(path_counts.most_common(10)),
        "error_sources": error_ips,
    }
```

---

### Pattern 10: Docker & Kubernetes SDKs ⭐⭐

```python
# === Docker SDK ===
import docker

client = docker.from_env()                    # Class instantiation (L1-13)

# List running containers
for container in client.containers.list():    # Loop (L1-06) over objects
    print(f"  {container.name}: {container.status}")

# Run a container
container = client.containers.run(
    "nginx:latest",
    detach=True,
    ports={"80/tcp": 8080},
    name="my-nginx",
)

# Build an image
image, logs = client.images.build(
    path=".",
    tag="myapp:latest",
)

# === Kubernetes client ===
from kubernetes import client, config

config.load_kube_config()                     # Or load_incluster_config() in-cluster
v1 = client.CoreV1Api()

# List pods
pods = v1.list_namespaced_pod("production")
for pod in pods.items:                        # Loop over objects
    print(f"  {pod.metadata.name}: {pod.status.phase}")

# Scale deployment
apps_v1 = client.AppsV1Api()
apps_v1.patch_namespaced_deployment_scale(
    name="billing-api",
    namespace="production",
    body={"spec": {"replicas": 5}},           # Dict (L1-07)!
)
```

---

## 📊 Concept Frequency in DevOps Scripts

| Concept | Frequency | Lessons |
|---------|:---------:|---------|
| **Dictionaries** | ████████████████████ | L1-07 |
| **String formatting** | ████████████████████ | L2-02 |
| **File I/O** | ███████████████████ | L2-08 |
| **Exception handling** | ███████████████████ | L3-05 |
| **Functions** | ██████████████████ | L1-09 |
| **Loops** | ██████████████████ | L1-06 |
| **Lists** | █████████████████ | L1-04 |
| **Conditionals** | ████████████████ | L1-03 |
| **subprocess** | ████████████████ | [DevOps L1](https://github.com/sva15/python/blob/main/devops/lesson-01-subprocess.md) |
| **os / pathlib** | ███████████████ | [DevOps L2](https://github.com/sva15/python/blob/main/devops/lesson-02-os-pathlib-shutil.md) |
| **requests (HTTP)** | ██████████████ | [DevOps L3](https://github.com/sva15/python/blob/main/devops/lesson-03-requests.md) |
| **YAML** | █████████████ | [DevOps L4](https://github.com/sva15/python/blob/main/devops/lesson-04-yaml.md) |
| **Regex** | ████████████ | L4-08 |
| **Logging** | ███████████ | L3-10 |
| **JSON** | ██████████ | L5-03 |
| **Decorators** | █████████ | L4-01 |
| **Classes** | ████████ | L1-11 |
| **Jinja2** | ███████ | [DevOps L5](https://github.com/sva15/python/blob/main/devops/lesson-05-jinja2.md) |

---

## ✅ DevOps Supplemental Lessons

All gaps are now covered with full Suits-themed lessons in the [`devops/`](https://github.com/sva15/python/blob/main/devops-module) module:

| # | Lesson | Topics Covered |
|---|--------|----------------|
| 1 | [subprocess](https://github.com/sva15/python/blob/main/devops/lesson-01-subprocess.md) | `subprocess.run`, Popen, shell security, pipelines |
| 2 | [os / pathlib / shutil](https://github.com/sva15/python/blob/main/devops/lesson-02-os-pathlib-shutil.md) | Env vars, paths, directories, copy/move/archive, hashlib |
| 3 | [requests](https://github.com/sva15/python/blob/main/devops/lesson-03-requests.md) | GET/POST, auth, health checks, retries, downloads |
| 4 | [PyYAML](https://github.com/sva15/python/blob/main/devops/lesson-04-yaml.md) | safe_load/dump, multi-doc, K8s/Compose/Actions generation |
| 5 | [Jinja2](https://github.com/sva15/python/blob/main/devops/lesson-05-jinja2.md) | Variables, loops, filters, inheritance, Nginx/K8s templates |
| 6 | [Click / argparse](https://github.com/sva15/python/blob/main/devops/lesson-06-click-argparse.md) | CLI tools, subcommands, colors, validation, testing |

---

## 🎯 Recommended Reading Order

```
WEEK 1: Script Foundations
  L1-07 Dictionaries       → API responses, configs, everything
  L2-08 File I/O           → Reading/writing config files
  L2-02 String Formatting  → Building commands, messages
  L1-09 Functions          → Organizing scripts

WEEK 2: Robust Automation
  L3-05 Exception Handling → Scripts that don't crash
  L5-03 CSV/JSON           → API responses, data files
  L3-10 Logging            → Production-ready scripts
  L4-08 Regex              → Log parsing

WEEK 3: DevOps Patterns
  L4-01 Decorators         → Retry logic, caching
  L3-09 Context Managers   → Cleanup, rollback
  L1-11 Classes            → SDK clients, resource objects
  L2-12 Virtual Envs       → Dependency management

WEEK 4: Professional Tools
  L3-11 Unit Testing       → Test infrastructure code
  L5-09 Packaging          → Distribute internal tools
  L5-05 Async              → Parallel operations

WEEK 5: DevOps Module (Supplemental)
  DevOps L1 subprocess     → Run shell commands
  DevOps L2 os/pathlib     → File system & env vars
  DevOps L3 requests       → HTTP & APIs
  DevOps L4 PyYAML         → YAML configs
  DevOps L5 Jinja2         → Config templating
  DevOps L6 Click          → Build CLI tools
```
