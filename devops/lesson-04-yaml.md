# 🏛️ DevOps Module | Lesson 4: Parse and Generate YAML with PyYAML

## *"The Contract Language"*

> **DevOps Objective:** *Read, write, and manipulate YAML files — the configuration language of Kubernetes, Docker Compose, Ansible, GitHub Actions, and virtually every DevOps tool.*

> **Previously Learned:** Dictionaries, lists, JSON, file I/O, pathlib

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

*The firm's Kubernetes manifests are managed by hand. 50 YAML files across 10 environments, and someone just deployed a staging config to production.*

```yaml
# deployment.yaml — looks simple, but...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: billing-api
spec:
  replicas: 3    # Was this supposed to be 1 for staging?
  template:
    spec:
      containers:
        - name: app
          image: billing:1.2.3    # Is this the right version?
```

**Mike:** "YAML is just data — dictionaries and lists. Python can read it, validate it, transform it, and generate it. No more copy-paste errors."

```python
import yaml

with open("deployment.yaml") as f:
    manifest = yaml.safe_load(f)

# It's just a dict!
print(manifest["spec"]["replicas"])           # 3
print(manifest["spec"]["template"]["spec"]
      ["containers"][0]["image"])              # billing:1.2.3
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "YAML is the language of infrastructure. If you can't read and write it from Python, you're doing DevOps by hand."

| Where YAML Lives | Example |
|-------------------|---------|
| Kubernetes | Deployments, Services, ConfigMaps, Ingress |
| Docker Compose | `docker-compose.yml` |
| GitHub Actions | `.github/workflows/*.yml` |
| Ansible | Playbooks, inventories, variables |
| Helm | `values.yaml`, chart templates |
| CI/CD | GitLab CI, CircleCI, Azure Pipelines |
| App config | `config.yaml`, feature flags |

```bash
pip install pyyaml
```

| YAML | Python |
|------|--------|
| `key: value` | `{"key": "value"}` |
| `- item` | `["item"]` |
| `nested:\n  key: value` | `{"nested": {"key": "value"}}` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |
| `42` | `42` (int) |
| `3.14` | `3.14` (float) |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Reading YAML — `yaml.safe_load()` ⭐⭐⭐

```python
import yaml

# === From a file ===
with open("config.yaml") as f:
    config = yaml.safe_load(f)    # Returns a dict (or list)!

# config.yaml:
# database:
#   host: postgres.prod.svc
#   port: 5432
#   name: billing
# features:
#   dark_mode: true
#   beta_users:
#     - harvey
#     - mike
#     - donna

print(type(config))                       # <class 'dict'>
print(config["database"]["host"])         # postgres.prod.svc
print(config["database"]["port"])         # 5432 (int, not string!)
print(config["features"]["dark_mode"])    # True (Python bool!)
print(config["features"]["beta_users"])   # ['harvey', 'mike', 'donna']

# === From a string ===
yaml_string = """
app:
  name: billing-api
  version: 1.2.3
  replicas: 3
  env:
    - name: DB_HOST
      value: localhost
    - name: LOG_LEVEL
      value: INFO
"""

data = yaml.safe_load(yaml_string)
print(data["app"]["name"])                # billing-api
print(data["app"]["env"][0]["value"])      # localhost

# ⚠️ ALWAYS use safe_load(), NEVER yaml.load()!
# yaml.load() can execute arbitrary Python code — security risk!
```

---

### 3.2 Writing YAML — `yaml.dump()` ⭐⭐

```python
import yaml

# === Python dict → YAML ===
config = {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {
        "name": "billing-api",
        "namespace": "production",
        "labels": {
            "app": "billing",
            "version": "1.2.3",
        },
    },
    "spec": {
        "replicas": 3,
        "selector": {
            "matchLabels": {"app": "billing"},
        },
        "template": {
            "spec": {
                "containers": [{
                    "name": "app",
                    "image": "billing:1.2.3",
                    "ports": [{"containerPort": 8080}],
                    "env": [
                        {"name": "DB_HOST", "value": "postgres.prod.svc"},
                        {"name": "LOG_LEVEL", "value": "INFO"},
                    ],
                }],
            },
        },
    },
}

# Write to file
with open("generated-deployment.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False, sort_keys=False)

# Dump to string
yaml_string = yaml.dump(config, default_flow_style=False, sort_keys=False)
print(yaml_string)

# === dump() options ===
yaml.dump(data, f,
    default_flow_style=False,   # Block style (readable), not inline
    sort_keys=False,            # Keep original key order
    indent=2,                   # Indentation
    allow_unicode=True,         # Allow non-ASCII characters
    width=120,                  # Line width before wrapping
)
```

---

### 3.3 Multi-Document YAML ⭐⭐

```python
import yaml

# Many K8s files have multiple documents separated by ---
# deployment.yaml:
# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: billing-svc
# ---
# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: billing-api

# === Read multi-document YAML ===
with open("k8s-manifests.yaml") as f:
    docs = list(yaml.safe_load_all(f))

print(f"Documents: {len(docs)}")
for doc in docs:
    print(f"  {doc['kind']}: {doc['metadata']['name']}")

# === Write multi-document YAML ===
documents = [
    {"apiVersion": "v1", "kind": "Service", "metadata": {"name": "billing-svc"}},
    {"apiVersion": "apps/v1", "kind": "Deployment", "metadata": {"name": "billing-api"}},
]

with open("output.yaml", "w") as f:
    yaml.dump_all(documents, f, default_flow_style=False, sort_keys=False)

# Output:
# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: billing-svc
# ---
# apiVersion: apps/v1
# kind: Deployment
# ...
```

---

### 3.4 Modifying YAML Files ⭐⭐⭐

```python
import yaml
from pathlib import Path

def update_image_tag(filepath, new_tag):
    """Update the container image tag in a K8s deployment."""
    path = Path(filepath)
    manifest = yaml.safe_load(path.read_text())
    
    # Navigate the dict structure
    containers = manifest["spec"]["template"]["spec"]["containers"]
    for container in containers:
        image = container["image"]
        repo = image.rsplit(":", 1)[0]       # "billing"
        container["image"] = f"{repo}:{new_tag}"
        print(f"  Updated: {image} → {container['image']}")
    
    # Write back
    path.write_text(yaml.dump(manifest, default_flow_style=False, sort_keys=False))

# update_image_tag("deployment.yaml", "1.3.0")

def scale_deployment(filepath, replicas):
    """Change replica count."""
    path = Path(filepath)
    manifest = yaml.safe_load(path.read_text())
    
    old = manifest["spec"]["replicas"]
    manifest["spec"]["replicas"] = replicas
    
    path.write_text(yaml.dump(manifest, default_flow_style=False, sort_keys=False))
    print(f"  Scaled: {old} → {replicas} replicas")

# scale_deployment("deployment.yaml", 5)
```

---

### 3.5 Validating YAML Configs ⭐⭐

```python
import yaml
from pathlib import Path

def validate_deployment(filepath):
    """Validate a K8s deployment manifest."""
    errors = []
    
    try:
        manifest = yaml.safe_load(Path(filepath).read_text())
    except yaml.YAMLError as e:
        return [f"Invalid YAML: {e}"]
    
    # Required fields
    if manifest.get("apiVersion") is None:
        errors.append("Missing 'apiVersion'")
    if manifest.get("kind") is None:
        errors.append("Missing 'kind'")
    if manifest.get("metadata", {}).get("name") is None:
        errors.append("Missing 'metadata.name'")
    
    # Deployment-specific
    if manifest.get("kind") == "Deployment":
        replicas = manifest.get("spec", {}).get("replicas")
        if replicas is not None and replicas < 1:
            errors.append(f"Replicas must be >= 1, got {replicas}")
        
        containers = (manifest.get("spec", {})
                      .get("template", {})
                      .get("spec", {})
                      .get("containers", []))
        
        if not containers:
            errors.append("No containers defined")
        
        for c in containers:
            if ":" not in c.get("image", ""):
                errors.append(f"Container '{c.get('name')}': image should have a tag")
    
    return errors

# Validate all YAML files in a directory
configs_dir = Path("k8s/")
if configs_dir.exists():
    for yaml_file in configs_dir.glob("**/*.yaml"):
        errors = validate_deployment(yaml_file)
        status = "✅" if not errors else "❌"
        print(f"  {status} {yaml_file.name}")
        for err in errors:
            print(f"      → {err}")
```

---

### 3.6 YAML Anchors and Aliases ⭐

```python
import yaml

# YAML has anchors (&) and aliases (*) for reuse
# Like variables in YAML!

yaml_with_anchors = """
defaults: &defaults
  replicas: 3
  resources:
    limits:
      cpu: "500m"
      memory: "256Mi"

services:
  billing:
    <<: *defaults        # Merge defaults
    image: billing:1.0
  
  users:
    <<: *defaults        # Same defaults
    image: users:2.1
    replicas: 5          # Override just this one
"""

data = yaml.safe_load(yaml_with_anchors)
print(data["services"]["billing"]["replicas"])   # 3 (from defaults)
print(data["services"]["users"]["replicas"])     # 5 (overridden)
print(data["services"]["users"]["resources"])    # From defaults
```

---

### 3.7 Environment-Specific Configs ⭐⭐

```python
import yaml
import os
from pathlib import Path

def load_config(env=None):
    """Load base config, then overlay environment-specific values."""
    env = env or os.environ.get("DEPLOY_ENV", "development")
    config_dir = Path(__file__).parent / "configs"
    
    # Load base config
    base_path = config_dir / "base.yaml"
    config = yaml.safe_load(base_path.read_text())
    
    # Overlay environment-specific config
    env_path = config_dir / f"{env}.yaml"
    if env_path.exists():
        env_config = yaml.safe_load(env_path.read_text())
        deep_merge(config, env_config)
    
    return config

def deep_merge(base, override):
    """Recursively merge override into base."""
    for key, value in override.items():
        if key in base and isinstance(base[key], dict) and isinstance(value, dict):
            deep_merge(base[key], value)
        else:
            base[key] = value

# configs/base.yaml:
#   app:
#     name: billing
#     log_level: INFO
#   database:
#     port: 5432

# configs/production.yaml:
#   app:
#     log_level: WARNING     # Override
#   database:
#     host: prod-db.internal # Add

# Result for production:
#   app: {name: billing, log_level: WARNING}
#   database: {port: 5432, host: prod-db.internal}
```

---

### 3.8 Docker Compose Generation ⭐⭐

```python
import yaml

def generate_compose(services_config):
    """Generate docker-compose.yml from service definitions."""
    compose = {
        "version": "3.8",
        "services": {},
        "networks": {
            "app-network": {"driver": "bridge"},
        },
    }
    
    for svc in services_config:
        compose["services"][svc["name"]] = {
            "image": svc["image"],
            "ports": [f"{svc.get('port', 8080)}:{svc.get('container_port', 8080)}"],
            "environment": svc.get("env", {}),
            "networks": ["app-network"],
            "restart": "unless-stopped",
        }
        if svc.get("depends_on"):
            compose["services"][svc["name"]]["depends_on"] = svc["depends_on"]
    
    return yaml.dump(compose, default_flow_style=False, sort_keys=False)

services = [
    {
        "name": "billing-api",
        "image": "billing:1.2.3",
        "port": 8080,
        "container_port": 8080,
        "env": {"DB_HOST": "postgres", "LOG_LEVEL": "INFO"},
        "depends_on": ["postgres"],
    },
    {
        "name": "postgres",
        "image": "postgres:15",
        "port": 5432,
        "container_port": 5432,
        "env": {"POSTGRES_DB": "billing", "POSTGRES_PASSWORD": "secret"},
    },
]

print(generate_compose(services))
```

---

### 3.9 GitHub Actions Workflow Generation ⭐

```python
import yaml

def generate_ci_workflow(python_versions=None, test_command="pytest"):
    """Generate a GitHub Actions CI workflow."""
    if python_versions is None:
        python_versions = ["3.11", "3.12", "3.13"]
    
    workflow = {
        "name": "CI",
        "on": ["push", "pull_request"],
        "jobs": {
            "test": {
                "runs-on": "ubuntu-latest",
                "strategy": {
                    "matrix": {
                        "python-version": python_versions,
                    },
                },
                "steps": [
                    {"uses": "actions/checkout@v4"},
                    {
                        "name": "Set up Python",
                        "uses": "actions/setup-python@v5",
                        "with": {"python-version": "${{ matrix.python-version }}"},
                    },
                    {
                        "name": "Install dependencies",
                        "run": "pip install -e '.[dev]'",
                    },
                    {
                        "name": "Run tests",
                        "run": test_command,
                    },
                ],
            },
        },
    }
    
    return yaml.dump(workflow, default_flow_style=False, sort_keys=False)

print(generate_ci_workflow())
```

---

### 3.10 Solving the Firm's Problem — YAML Config Pipeline

```python
# ============================================
# Pearson Specter Litt — YAML Config Pipeline
# Read, validate, transform, and deploy configs
# ============================================

import yaml
import os
from pathlib import Path
from datetime import datetime

class ConfigPipeline:
    """Process YAML configurations for deployment."""
    
    REQUIRED_FIELDS = {
        "Deployment": ["apiVersion", "kind", "metadata", "spec"],
        "Service": ["apiVersion", "kind", "metadata", "spec"],
    }
    
    def __init__(self, config_dir):
        self.config_dir = Path(config_dir)
        self.errors = []
        self.processed = []
    
    def discover(self, pattern="**/*.yaml"):
        """Find all YAML files."""
        files = list(self.config_dir.glob(pattern))
        print(f"📁 Found {len(files)} YAML files in {self.config_dir}")
        return files
    
    def validate(self, filepath):
        """Validate a YAML file."""
        try:
            content = Path(filepath).read_text()
            docs = list(yaml.safe_load_all(content))
            
            for doc in docs:
                if doc is None:
                    continue
                kind = doc.get("kind", "Unknown")
                required = self.REQUIRED_FIELDS.get(kind, [])
                
                for field in required:
                    if field not in doc:
                        self.errors.append(f"{filepath}: Missing '{field}' in {kind}")
            
            return len(self.errors) == 0
        except yaml.YAMLError as e:
            self.errors.append(f"{filepath}: YAML parse error: {e}")
            return False
    
    def transform(self, filepath, env, version):
        """Apply environment-specific transformations."""
        content = Path(filepath).read_text()
        docs = list(yaml.safe_load_all(content))
        
        transformed = []
        for doc in docs:
            if doc is None:
                continue
            
            # Update namespace
            if "metadata" in doc:
                doc["metadata"]["namespace"] = env
                doc["metadata"].setdefault("labels", {})["version"] = version
                doc["metadata"]["labels"]["managed-by"] = "config-pipeline"
            
            # Update image tags
            containers = (doc.get("spec", {})
                         .get("template", {})
                         .get("spec", {})
                         .get("containers", []))
            for c in containers:
                if "image" in c:
                    repo = c["image"].rsplit(":", 1)[0]
                    c["image"] = f"{repo}:{version}"
            
            transformed.append(doc)
        
        return transformed
    
    def run(self, env, version):
        """Execute the full pipeline."""
        print(f"\n{'='*55}")
        print(f"  🏛️ Config Pipeline: {env} v{version}")
        print(f"{'='*55}")
        
        files = self.discover()
        
        # Validate
        print(f"\n🔍 Validating...")
        valid_count = sum(1 for f in files if self.validate(f))
        print(f"  ✅ {valid_count}/{len(files)} valid")
        if self.errors:
            for err in self.errors:
                print(f"  ❌ {err}")
            return False
        
        # Transform
        print(f"\n🔄 Transforming for '{env}'...")
        output_dir = self.config_dir / "generated" / env
        output_dir.mkdir(parents=True, exist_ok=True)
        
        for filepath in files:
            docs = self.transform(filepath, env, version)
            output_path = output_dir / filepath.name
            
            with open(output_path, "w") as f:
                yaml.dump_all(docs, f, default_flow_style=False, sort_keys=False)
            
            print(f"  📄 {filepath.name} → {output_path}")
            self.processed.append(output_path)
        
        print(f"\n✅ Pipeline complete: {len(self.processed)} files → {output_dir}")
        print(f"{'='*55}")
        return True

# pipeline = ConfigPipeline("k8s/")
# pipeline.run("production", "1.2.3")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: `yaml.load()` Is Unsafe 🔥
```python
# ❌ yaml.load() can execute arbitrary code!
data = yaml.load(user_input)    # DANGEROUS!

# ✅ ALWAYS use safe_load
data = yaml.safe_load(user_input)
```

### Pitfall 2: YAML Boolean Gotcha 🔥
```python
# YAML auto-converts these to booleans!
# "yes", "no", "on", "off", "true", "false"

yaml_string = "country: no"    # Norway? NOPE → False!
data = yaml.safe_load(yaml_string)
print(data["country"])    # False  ← not "no"!

# ✅ Quote strings that look like booleans
# country: "no"
# enabled: "yes"
```

### Pitfall 3: Indentation Matters 🔥
```yaml
# ❌ Wrong indentation = wrong structure
items:
- name: billing    # This is WRONG
  port: 8080

# ✅ Correct indentation
items:
  - name: billing
    port: 8080
```

### Pitfall 4: sort_keys Reorders Your YAML 🔥
```python
# ❌ Default sorts keys alphabetically!
yaml.dump(manifest)    # apiVersion before kind before metadata

# ✅ Preserve original order
yaml.dump(manifest, sort_keys=False)
```

### Pitfall 5: Empty YAML Document 🔥
```python
# ❌ Empty file or "---" returns None, not {}
data = yaml.safe_load("")      # None
data = yaml.safe_load("---")   # None

# ✅ Handle None
data = yaml.safe_load(content) or {}
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
YAML = THE FIRM'S FILING SYSTEM:
  Indentation = folder hierarchy
  Keys = folder labels
  Values = documents inside

  database:              ← Folder "database"
    host: localhost       ← Document: host=localhost
    port: 5432           ← Document: port=5432

YAML → PYTHON:
  YAML dictionaries  →  Python dicts
  YAML lists         →  Python lists
  YAML scalars       →  Python str/int/float/bool/None
  
  That's it. YAML is just dicts and lists written sideways.

DONNA'S RULE:
  "YAML is the language of infrastructure.
   Learn to read it like a contract.
   Always use safe_load. Always preserve key order.
   And never, EVER, trust YAML booleans."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

### 🟢 Beginner
1. **YAML Reader**: Load a K8s deployment YAML, print image name and replica count.
2. **Config Writer**: Generate a `config.yaml` from a Python dict with nested structure.
3. **Multi-Doc**: Read a multi-document YAML file, print the `kind` and `name` of each.

### 🟡 Intermediate
4. **Image Updater**: Read a deployment YAML, update the image tag, write it back.
5. **Config Merger**: Merge `base.yaml` + `production.yaml` with deep merge.
6. **YAML Validator**: Validate all YAML files in a directory, report errors.

### 🔴 Advanced
7. **Compose Generator**: Generate Docker Compose from a services definition dict.
8. **Config Pipeline**: Read → validate → transform → write for multiple environments.
9. **YAML Diff**: Compare two YAML files and report differences.

---

## 9. ✅ Knowledge Check

**Q1.** What does `yaml.safe_load()` return?
> A Python dict (or list, or scalar) — the Python equivalent of the YAML structure.

**Q2.** Why use `safe_load` instead of `load`?
> `yaml.load()` can execute arbitrary Python code embedded in YAML — a security vulnerability. `safe_load` only parses data.

**Q3.** How do you handle multi-document YAML files?
> `yaml.safe_load_all(f)` returns a generator of documents. Use `list()` to get all of them.

**Q4.** What Python type does a YAML mapping become?
> A Python `dict`.

**Q5.** What's the `sort_keys=False` option for?
> Prevents `yaml.dump()` from alphabetically sorting keys, preserving the original order.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Reading YAML (`safe_load`) | ✅ |
| 2 | Writing YAML (`dump`) | ✅ |
| 3 | Multi-document YAML | ✅ |
| 4 | Modifying YAML files | ✅ |
| 5 | Validating YAML configs | ✅ |
| 6 | Anchors and aliases | ✅ |
| 7 | Environment-specific configs | ✅ |
| 8 | Docker Compose generation | ✅ |
| 9 | GitHub Actions generation | ✅ |
| 10 | Config pipeline | ✅ |

---

> **Next Lesson →** DevOps Lesson 5: *Generate Configs with Jinja2 Templating*
