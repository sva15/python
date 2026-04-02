# Phase 2 · Lesson 6 — PyYAML
### *"The Language of DevOps"*

> **DevOps Objective:** Read, write, and generate YAML files — the format of Kubernetes, Ansible, Docker Compose, GitHub Actions, and most DevOps configs.
> **DevOps connection:** YAML is literally the language of modern infrastructure. Every K8s manifest, every CI/CD pipeline, every Ansible playbook — all YAML. Being able to parse and generate it from Python is essential.
> **10-min sprint:** Read a Kubernetes deployment YAML file. Print the container image, replica count, and all environment variable names. Change the replica count and write the modified YAML back.

---

## Setup

```bash
pip install pyyaml
```

---

## Why This Matters

If you've written Kubernetes manifests, Ansible playbooks, or GitHub Actions workflows, you've written YAML. PyYAML lets you read those files into Python dicts, manipulate them programmatically, and write them back out — enabling you to generate configs dynamically, validate manifests, and automate config management.

**Key insight: YAML → Python dict, Python dict → YAML. That's it.**

---

## The Core Concept

### Reading YAML

```python
import yaml

# Read a YAML file
with open("deployment.yaml") as f:
    config = yaml.safe_load(f)    # returns a Python dict (or list)

# Access values — it's just a dict!
print(config["apiVersion"])           # apps/v1
print(config["spec"]["replicas"])     # 3
print(config["metadata"]["name"])     # billing-api
```

> **Always use `yaml.safe_load()`**, never `yaml.load()` — the unsafe version can execute arbitrary Python code.

### Sample YAML → Python Dict

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: billing-api
  namespace: production
  labels:
    app: billing
    version: "1.2.3"
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: billing
          image: registry.psl.com/billing:1.2.3
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: postgres.prod.svc
            - name: DB_PORT
              value: "5432"
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "1000m"
              memory: "512Mi"
```

```python
# After yaml.safe_load():
config["spec"]["replicas"]                           # 3
config["spec"]["template"]["spec"]["containers"][0]["image"]   # registry.psl.com/billing:1.2.3
config["metadata"]["labels"]["version"]             # "1.2.3"

# Loop through env vars
containers = config["spec"]["template"]["spec"]["containers"]
for container in containers:
    print(f"Container: {container['name']}")
    for env_var in container.get("env", []):
        print(f"  {env_var['name']} = {env_var.get('value', '(from secret)')}")
```

### Writing YAML

```python
import yaml

# Python dict → YAML
deployment = {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {"name": "billing-api", "namespace": "production"},
    "spec": {
        "replicas": 3,
        "template": {
            "spec": {
                "containers": [{
                    "name": "billing",
                    "image": "billing:1.2.3",
                    "ports": [{"containerPort": 8080}],
                }]
            }
        }
    }
}

with open("generated-deployment.yaml", "w") as f:
    yaml.dump(deployment, f, default_flow_style=False, indent=2)
```

### YAML from Strings

```python
yaml_string = """
host: localhost
port: 5432
database: billing_prod
ssl: true
max_connections: 20
"""

config = yaml.safe_load(yaml_string)
print(config["port"])       # 5432 (int, not string!)
print(config["ssl"])        # True (bool, not string!)
print(type(config["ssl"]))  # <class 'bool'>
```

> **Important:** PyYAML auto-converts types. `"5432"` (quoted) becomes a string; `5432` (unquoted) becomes an int. `true`/`false` become Python `True`/`False`.

### Multi-Document YAML (K8s manifests often use `---`)

```python
import yaml

# K8s manifest with multiple documents separated by ---
with open("k8s-manifests.yaml") as f:
    docs = list(yaml.safe_load_all(f))   # returns a list of dicts

for doc in docs:
    if doc:
        print(f"Kind: {doc.get('kind')}, Name: {doc.get('metadata', {}).get('name')}")
```

### Dumping to String (not file)

```python
import yaml

config = {"host": "localhost", "port": 5432, "ssl": True}

# To string
yaml_text = yaml.dump(config, default_flow_style=False)
print(yaml_text)
# host: localhost
# port: 5432
# ssl: true
```

---

## DevOps Example — K8s Manifest Generator

```python
import yaml
import os
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

def generate_deployment(
    service_name: str,
    image: str,
    replicas: int,
    namespace: str,
    env_vars: dict,
    resources: dict = None,
) -> dict:
    """Generate a Kubernetes Deployment manifest as a Python dict."""

    default_resources = {
        "requests": {"cpu": "100m", "memory": "128Mi"},
        "limits": {"cpu": "500m", "memory": "512Mi"},
    }

    return {
        "apiVersion": "apps/v1",
        "kind": "Deployment",
        "metadata": {
            "name": service_name,
            "namespace": namespace,
            "labels": {"app": service_name},
        },
        "spec": {
            "replicas": replicas,
            "selector": {"matchLabels": {"app": service_name}},
            "template": {
                "metadata": {"labels": {"app": service_name}},
                "spec": {
                    "containers": [{
                        "name": service_name,
                        "image": image,
                        "env": [
                            {"name": k, "value": str(v)}
                            for k, v in env_vars.items()
                        ],
                        "resources": resources or default_resources,
                        "readinessProbe": {
                            "httpGet": {"path": "/health", "port": 8080},
                            "initialDelaySeconds": 10,
                            "periodSeconds": 5,
                        },
                    }]
                }
            }
        }
    }


def update_image(manifest_path: str, new_image: str, dry_run: bool = False) -> bool:
    """Update the container image in an existing K8s manifest."""
    path = Path(manifest_path)
    if not path.exists():
        raise FileNotFoundError(f"Manifest not found: {manifest_path}")

    with open(path) as f:
        manifest = yaml.safe_load(f)

    containers = (manifest
                  .get("spec", {})
                  .get("template", {})
                  .get("spec", {})
                  .get("containers", []))

    if not containers:
        logger.error("No containers found in manifest")
        return False

    old_image = containers[0]["image"]
    containers[0]["image"] = new_image
    logger.info(f"Image update: {old_image} → {new_image}")

    if dry_run:
        logger.info("[DRY RUN] Would write:")
        print(yaml.dump(manifest, default_flow_style=False))
        return True

    with open(path, "w") as f:
        yaml.dump(manifest, f, default_flow_style=False)

    logger.info(f"Updated: {manifest_path}")
    return True


# --- Usage ---
# Generate a new deployment
deployment = generate_deployment(
    service_name="billing-api",
    image="registry.psl.com/billing:1.2.3",
    replicas=3,
    namespace="production",
    env_vars={"DB_HOST": "postgres.prod.svc", "DB_PORT": "5432", "LOG_LEVEL": "INFO"},
)

with open("billing-deployment.yaml", "w") as f:
    yaml.dump(deployment, f, default_flow_style=False, indent=2)

print(f"Generated deployment for {deployment['metadata']['name']}")

# Update image in existing manifest
update_image("billing-deployment.yaml", "registry.psl.com/billing:1.3.0", dry_run=True)
```

---

## Gotchas (Louis's warnings)

**1. Always use `safe_load`, never `load`:**
```python
# ❌ DANGEROUS — can execute arbitrary Python code!
config = yaml.load(f)

# ✅ SAFE
config = yaml.safe_load(f)
```

**2. YAML auto-converts types — be careful:**
```python
# In YAML:  port: 5432  → Python int
# In YAML:  port: "5432" → Python string
# In YAML:  enabled: true → Python True
# In YAML:  version: 1.0 → Python float (not string "1.0"!)
# In YAML:  version: "1.0" → Python string

# This can cause issues:
config["port"]     # int 5432 — if you try to use it as a string, TypeError!
str(config["port"])  # "5432" ✅
```

**3. Indentation errors in generated YAML crash kubectl:**
```python
# Always validate generated YAML before applying to Kubernetes
# Use: kubectl apply --dry-run=client -f generated.yaml
```

**4. Dumping Unicode — set `allow_unicode=True`:**
```python
yaml.dump(data, f, allow_unicode=True)   # Don't encode non-ASCII as \uXXXX
```

**5. Ordered output — YAML dicts may sort keys:**
```python
# yaml.dump() sorts keys alphabetically by default
yaml.dump(data, f, sort_keys=False)   # preserve insertion order
```

---

## YAML Type Cheat Sheet

| YAML | Python type | Notes |
|------|------------|-------|
| `name: billing` | `str` | |
| `port: 5432` | `int` | |
| `rate: 3.14` | `float` | |
| `enabled: true` | `bool` | also: `yes`, `on` |
| `enabled: false` | `bool` | also: `no`, `off` |
| `value: null` | `None` | also: `~` |
| `version: "1.0"` | `str` | quotes force string |
| `items: [a, b, c]` | `list` | |
| `key: {a: 1}` | `dict` | |

---

## Practice (pick one, 10 minutes)

**Beginner:** Read a `docker-compose.yaml` file. Print all service names and their images. Count total services defined.

**Intermediate:** Write a script that reads a K8s deployment YAML, changes the replica count and image tag based on command-line args, and writes the updated manifest back. Add a `--dry-run` flag.

**DevOps-specific:** Write a multi-environment config generator. Given a base config and an environment name (dev/staging/prod), merge environment-specific overrides and generate the final YAML config file.

---

## Knowledge Check

1. What function should you always use to read YAML? Why not the alternative?
2. What Python type does `replicas: 3` produce? What about `replicas: "3"`?
3. How do you read a YAML file with multiple `---` separated documents?
4. What does `default_flow_style=False` do in `yaml.dump()`?
5. How do you preserve dict key order when dumping YAML?

**Answers:** (1) `yaml.safe_load()` — `yaml.load()` can execute arbitrary Python code in the YAML. (2) `int 3` vs `str "3"` — unquoted numbers are converted. (3) `list(yaml.safe_load_all(f))`. (4) Outputs block style (multi-line, readable) instead of inline style. (5) `yaml.dump(data, sort_keys=False)`.

---

> **Next:** Lesson 7 — Jinja2 → Generate nginx configs, K8s manifests, and Ansible playbooks from templates.
