# 🏛️ DevOps Module | Lesson 5: Generate Configs with Jinja2 Templating

## *"The Boilerplate Machine"*

> **DevOps Objective:** *Use Jinja2 templates to generate configuration files, Kubernetes manifests, Nginx configs, and CI/CD pipelines from reusable templates with variables, loops, and conditionals.*

> **Previously Learned:** f-strings, dictionaries, loops, conditionals, YAML, file I/O

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

*The firm runs 12 microservices. Each needs a Kubernetes Deployment, Service, ConfigMap, and Ingress — 48 YAML files. They're 90% identical. Somebody has to maintain all 48.*

```yaml
# ❌ Copied 48 times, manually updated
# billing-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: billing-api        # Different per service
spec:
  replicas: 3              # Different per environment
  template:
    spec:
      containers:
        - name: billing
          image: billing:1.2.3    # Different per deploy

# users-deployment.yaml — 95% identical, just different names!
```

**Mike:** "One template. Twelve configs. Zero copy-paste."

```python
# ✅ One template, many outputs
from jinja2 import Template

template = Template("""
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ service_name }}
spec:
  replicas: {{ replicas }}
  template:
    spec:
      containers:
        - name: {{ service_name }}
          image: {{ image }}:{{ version }}
""")

# Generate for every service
for svc in ["billing", "users", "search", "auth"]:
    print(template.render(service_name=svc, replicas=3, image=svc, version="1.2.3"))
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "If you're writing the same thing twice, you're doing it wrong. Template it."

| Jinja2 Syntax | Purpose | Python Equivalent |
|---------------|---------|-------------------|
| `{{ variable }}` | Insert a value | `f"{variable}"` |
| `{% if condition %}` | Conditional block | `if condition:` |
| `{% for item in list %}` | Loop | `for item in list:` |
| `{# comment #}` | Comment (not rendered) | `# comment` |
| `{{ var \| filter }}` | Transform a value | `filter(var)` |

```bash
pip install jinja2
```

**Where Jinja2 is already used:**
- **Ansible** — all playbooks and templates
- **Helm** — Kubernetes chart templates
- **Flask** — HTML templates
- **Cookiecutter** — project scaffolding
- **Salt** — configuration management
- **dbt** — SQL templates

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Variables — `{{ }}` ⭐⭐⭐

```python
from jinja2 import Template

# === Simple variable substitution ===
template = Template("Hello, {{ name }}! You have {{ count }} cases.")
result = template.render(name="Harvey", count=15)
print(result)    # Hello, Harvey! You have 15 cases.

# === Dict/object access ===
template = Template("""
Server: {{ server.host }}:{{ server.port }}
Environment: {{ config['env'] }}
""")

result = template.render(
    server={"host": "10.0.1.50", "port": 8080},
    config={"env": "production"},
)
print(result)
# Server: 10.0.1.50:8080
# Environment: production

# === Default values ===
template = Template("Region: {{ region | default('us-east-1') }}")
print(template.render())                        # Region: us-east-1
print(template.render(region="eu-west-1"))      # Region: eu-west-1

# === Undefined variables ===
# By default, undefined variables render as empty string
# To catch them: use jinja2.StrictUndefined (see 3.7)
```

---

### 3.2 Conditionals — `{% if %}` ⭐⭐⭐

```python
from jinja2 import Template

# === if / elif / else ===
template = Template("""
{% if environment == 'production' %}
replicas: 5
resources:
  limits:
    cpu: "2000m"
    memory: "4Gi"
{% elif environment == 'staging' %}
replicas: 2
resources:
  limits:
    cpu: "500m"
    memory: "1Gi"
{% else %}
replicas: 1
resources:
  limits:
    cpu: "200m"
    memory: "256Mi"
{% endif %}
""")

print(template.render(environment="production"))
# replicas: 5
# resources: ...cpu: "2000m"

print(template.render(environment="development"))
# replicas: 1
# resources: ...cpu: "200m"

# === Inline conditional ===
template = Template(
    "Protocol: {{ 'https' if ssl_enabled else 'http' }}"
)
print(template.render(ssl_enabled=True))     # Protocol: https
print(template.render(ssl_enabled=False))    # Protocol: http

# === Truthiness ===
# Empty lists, empty strings, None, 0, False are all falsy
template = Template("""
{% if extra_labels %}
labels:
{% for key, value in extra_labels.items() %}
  {{ key }}: {{ value }}
{% endfor %}
{% endif %}
""")
```

---

### 3.3 Loops — `{% for %}` ⭐⭐⭐

```python
from jinja2 import Template

# === Loop over a list ===
template = Template("""
env:
{% for var in env_vars %}
  - name: {{ var.name }}
    value: "{{ var.value }}"
{% endfor %}
""")

result = template.render(env_vars=[
    {"name": "DB_HOST", "value": "postgres.prod.svc"},
    {"name": "DB_PORT", "value": "5432"},
    {"name": "LOG_LEVEL", "value": "INFO"},
])
print(result)
# env:
#   - name: DB_HOST
#     value: "postgres.prod.svc"
#   - name: DB_PORT
#     value: "5432"
#   - name: LOG_LEVEL
#     value: "INFO"

# === Loop over a dict ===
template = Template("""
{% for key, value in labels.items() %}
  {{ key }}: {{ value }}
{% endfor %}
""")

result = template.render(labels={"app": "billing", "tier": "backend", "version": "1.2.3"})

# === Loop variables ===
template = Template("""
{% for host in hosts %}
server {{ loop.index }} {
    hostname {{ host }};
}
{% endfor %}
Total: {{ hosts | length }} servers
""")

# loop.index   = 1-based counter (1, 2, 3...)
# loop.index0  = 0-based counter (0, 1, 2...)
# loop.first   = True on first iteration
# loop.last    = True on last iteration
# loop.length  = Total items

# === Conditional in loop ===
template = Template("""
{% for svc in services %}
{% if svc.enabled %}
  - {{ svc.name }}: {{ svc.port }}
{% endif %}
{% endfor %}
""")
```

---

### 3.4 Filters — Transform Values ⭐⭐

```python
from jinja2 import Template

# Filters use the pipe syntax: {{ value | filter }}

template = Template("""
Name: {{ name | upper }}
Slug: {{ name | lower | replace(' ', '-') }}
Default: {{ missing | default('N/A') }}
Length: {{ items | length }}
Joined: {{ tags | join(', ') }}
First: {{ items | first }}
Last: {{ items | last }}
JSON: {{ data | tojson }}
Trimmed: {{ messy | trim }}
Indented: {{ block | indent(4) }}
""")

result = template.render(
    name="Billing Service",
    items=["a", "b", "c"],
    tags=["backend", "python", "api"],
    data={"key": "value"},
    messy="  extra spaces  ",
    block="line1\nline2\nline3",
)
print(result)

# Common DevOps filters:
# {{ var | default('fallback') }}     — Default value
# {{ list | join(',') }}              — Join list items
# {{ text | indent(4) }}             — Add indentation
# {{ dict | tojson }}                — Serialize to JSON
# {{ text | trim }}                  — Strip whitespace
# {{ text | upper / lower }}         — Case conversion
# {{ list | length }}                — Count items
# {{ list | sort }}                  — Sort
# {{ list | unique }}                — Remove duplicates
# {{ text | regex_replace(p, r) }}   — Regex replacement
```

---

### 3.5 Loading Templates from Files ⭐⭐

```python
from jinja2 import Environment, FileSystemLoader
from pathlib import Path

# === File-based templates (the standard approach) ===
# Directory structure:
# templates/
#   deployment.yaml.j2
#   service.yaml.j2
#   configmap.yaml.j2
#   nginx.conf.j2

env = Environment(
    loader=FileSystemLoader("templates"),
    trim_blocks=True,           # Remove newline after block tags
    lstrip_blocks=True,         # Strip leading whitespace from block tags
    keep_trailing_newline=True, # Keep final newline
)

# Load and render a template
template = env.get_template("deployment.yaml.j2")
output = template.render(
    service_name="billing-api",
    namespace="production",
    replicas=3,
    image="billing",
    version="1.2.3",
    env_vars=[
        {"name": "DB_HOST", "value": "postgres.prod.svc"},
    ],
)

# Write the rendered output
Path("output/deployment.yaml").write_text(output)

# === Template file: templates/deployment.yaml.j2 ===
# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: {{ service_name }}
#   namespace: {{ namespace }}
# spec:
#   replicas: {{ replicas }}
#   selector:
#     matchLabels:
#       app: {{ service_name }}
#   template:
#     metadata:
#       labels:
#         app: {{ service_name }}
#         version: {{ version }}
#     spec:
#       containers:
#         - name: {{ service_name }}
#           image: {{ image }}:{{ version }}
#           ports:
#             - containerPort: 8080
#           {% if env_vars %}
#           env:
#             {% for var in env_vars %}
#             - name: {{ var.name }}
#               value: "{{ var.value }}"
#             {% endfor %}
#           {% endif %}
```

---

### 3.6 Template Inheritance — DRY Configs ⭐⭐

```python
from jinja2 import Environment, FileSystemLoader

# === Base template: templates/base-deployment.yaml.j2 ===
# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: {{ service_name }}
#   namespace: {{ namespace }}
#   labels:
#     {% block labels %}
#     app: {{ service_name }}
#     {% endblock %}
# spec:
#   replicas: {% block replicas %}1{% endblock %}
#   template:
#     spec:
#       containers:
#         - name: {{ service_name }}
#           image: {{ image }}:{{ version }}
#           {% block resources %}{% endblock %}
#           {% block env %}{% endblock %}

# === Child template: templates/prod-deployment.yaml.j2 ===
# {% extends "base-deployment.yaml.j2" %}
#
# {% block replicas %}3{% endblock %}
#
# {% block resources %}
#           resources:
#             limits:
#               cpu: "2000m"
#               memory: "4Gi"
# {% endblock %}
#
# {% block env %}
#           env:
#             - name: LOG_LEVEL
#               value: "WARNING"
# {% endblock %}

env = Environment(loader=FileSystemLoader("templates"))

# Base template → minimal deployment
base = env.get_template("base-deployment.yaml.j2")
print(base.render(service_name="billing", namespace="dev", image="billing", version="1.0"))

# Child template → production deployment (inherits + overrides)
prod = env.get_template("prod-deployment.yaml.j2")
print(prod.render(service_name="billing", namespace="prod", image="billing", version="1.0"))
```

---

### 3.7 Strict Mode and Custom Filters ⭐

```python
from jinja2 import Environment, FileSystemLoader, StrictUndefined
import base64

# === Strict mode — catch undefined variables ===
env = Environment(
    loader=FileSystemLoader("templates"),
    undefined=StrictUndefined,    # Raise error on undefined vars!
)

# template.render(name="Harvey")
# If template uses {{ email }} but email isn't passed → UndefinedError!

# === Custom filters ===
def b64encode(value):
    """Base64 encode (for K8s Secrets)."""
    return base64.b64encode(value.encode()).decode()

def quote(value):
    """Wrap in quotes."""
    return f'"{value}"'

env.filters["b64encode"] = b64encode
env.filters["quote"] = quote

# Now usable in templates:
# password: {{ db_password | b64encode }}
# name: {{ service_name | quote }}

# === Custom tests ===
def is_production(value):
    return value in ("production", "prod")

env.tests["production"] = is_production

# In template:
# {% if environment is production %}
#   replicas: 5
# {% endif %}
```

---

### 3.8 Nginx Config Generation ⭐⭐

```python
from jinja2 import Environment, FileSystemLoader

# templates/nginx.conf.j2:
nginx_template = """
upstream {{ service_name }} {
    {% for backend in backends %}
    server {{ backend.host }}:{{ backend.port }} weight={{ backend.weight | default(1) }};
    {% endfor %}
}

server {
    listen {{ listen_port | default(80) }};
    server_name {{ domain }};

    {% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/{{ domain }}.crt;
    ssl_certificate_key /etc/ssl/{{ domain }}.key;
    
    if ($scheme = http) {
        return 301 https://$host$request_uri;
    }
    {% endif %}

    {% for location in locations %}
    location {{ location.path }} {
        {% if location.type == 'proxy' %}
        proxy_pass http://{{ service_name }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        {% elif location.type == 'static' %}
        root {{ location.root }};
        expires {{ location.cache | default('1h') }};
        {% elif location.type == 'redirect' %}
        return 301 {{ location.target }};
        {% endif %}
    }
    {% endfor %}

    {% if health_check %}
    location /health {
        access_log off;
        return 200 'OK';
    }
    {% endif %}
}
"""

from jinja2 import Template
template = Template(nginx_template)

config = template.render(
    service_name="billing-api",
    domain="billing.pearsonspecter.com",
    listen_port=80,
    ssl_enabled=True,
    health_check=True,
    backends=[
        {"host": "10.0.1.10", "port": 8080, "weight": 3},
        {"host": "10.0.1.11", "port": 8080, "weight": 2},
        {"host": "10.0.1.12", "port": 8080, "weight": 1},
    ],
    locations=[
        {"path": "/", "type": "proxy"},
        {"path": "/static/", "type": "static", "root": "/var/www/static", "cache": "7d"},
        {"path": "/old-api/", "type": "redirect", "target": "/api/v2/"},
    ],
)

print(config)
```

---

### 3.9 Macros — Reusable Template Functions ⭐

```python
from jinja2 import Template

# Macros = functions inside templates

template = Template("""
{% macro container(name, image, port, cpu="200m", memory="256Mi") %}
        - name: {{ name }}
          image: {{ image }}
          ports:
            - containerPort: {{ port }}
          resources:
            limits:
              cpu: "{{ cpu }}"
              memory: "{{ memory }}"
{% endmacro %}

spec:
  containers:
    {{ container("billing", "billing:1.2.3", 8080, cpu="1000m", memory="2Gi") }}
    {{ container("sidecar", "envoy:1.28", 9901) }}
""")

print(template.render())

# Macros can be in separate files and imported:
# {% from "macros.j2" import container, volume_mount %}
```

---

### 3.10 Solving the Firm's Problem — Multi-Service Config Generator

```python
# ============================================
# Pearson Specter Litt — Config Generator
# One template, 12 services, 3 environments
# ============================================

from jinja2 import Environment, DictLoader
import yaml

DEPLOYMENT_TEMPLATE = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ service.name }}
  namespace: {{ env.namespace }}
  labels:
    app: {{ service.name }}
    environment: {{ env.name }}
    version: "{{ service.version }}"
    managed-by: config-generator
spec:
  replicas: {{ env.replicas | default(service.replicas | default(1)) }}
  selector:
    matchLabels:
      app: {{ service.name }}
  template:
    metadata:
      labels:
        app: {{ service.name }}
        version: "{{ service.version }}"
    spec:
      containers:
        - name: {{ service.name }}
          image: {{ service.image }}:{{ service.version }}
          ports:
            - containerPort: {{ service.port }}
          {% if env.resources %}
          resources:
            limits:
              cpu: "{{ env.resources.cpu }}"
              memory: "{{ env.resources.memory }}"
          {% endif %}
          {% if service.env_vars %}
          env:
            {% for var in service.env_vars %}
            - name: {{ var.name }}
              value: "{{ var.value }}"
            {% endfor %}
          {% endif %}
"""

jinja_env = Environment(
    loader=DictLoader({"deployment.yaml.j2": DEPLOYMENT_TEMPLATE}),
    trim_blocks=True,
    lstrip_blocks=True,
)

# Service definitions
services = [
    {"name": "billing-api", "image": "billing", "version": "1.2.3", "port": 8080,
     "env_vars": [{"name": "DB_HOST", "value": "postgres.svc"}]},
    {"name": "user-service", "image": "users", "version": "2.1.0", "port": 3000,
     "env_vars": [{"name": "CACHE_HOST", "value": "redis.svc"}]},
    {"name": "search-api", "image": "search", "version": "0.9.5", "port": 9200,
     "env_vars": []},
]

# Environment definitions
environments = {
    "dev": {"name": "dev", "namespace": "development", "replicas": 1,
            "resources": {"cpu": "200m", "memory": "256Mi"}},
    "staging": {"name": "staging", "namespace": "staging", "replicas": 2,
                "resources": {"cpu": "500m", "memory": "512Mi"}},
    "prod": {"name": "prod", "namespace": "production", "replicas": 5,
             "resources": {"cpu": "2000m", "memory": "4Gi"}},
}

# Generate ALL configs
print("=" * 55)
print(f"{'🏛️  CONFIG GENERATOR':^55}")
print("=" * 55)

template = jinja_env.get_template("deployment.yaml.j2")
total = 0

for env_name, env_config in environments.items():
    print(f"\n📁 {env_name}/")
    for service in services:
        output = template.render(service=service, env=env_config)
        filename = f"{service['name']}-deployment.yaml"
        
        # Validate the generated YAML
        parsed = yaml.safe_load(output)
        replicas = parsed["spec"]["replicas"]
        image = parsed["spec"]["template"]["spec"]["containers"][0]["image"]
        
        print(f"  📄 {filename} — {replicas} replicas, {image}")
        total += 1

print(f"\n✅ Generated {total} configs ({len(services)} services × {len(environments)} environments)")
print(f"{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: YAML Indentation in Templates 🔥
```jinja
{# ❌ Wrong indentation — breaks YAML! #}
spec:
  containers:
  {% for c in containers %}
    - name: {{ c.name }}
  {% endfor %}

{# ✅ Use indent filter or careful spacing #}
spec:
  containers:
    {% for c in containers %}
    - name: {{ c.name }}
    {% endfor %}
```

### Pitfall 2: Undefined Variables Silently Disappear 🔥
```python
# ❌ Default Jinja2 renders undefined as empty string
Template("Name: {{ nme }}").render(name="Harvey")    # "Name: "  (typo!)

# ✅ Use StrictUndefined to catch typos
env = Environment(undefined=StrictUndefined)
```

### Pitfall 3: Special Characters in YAML Values 🔥
```jinja
{# ❌ If value contains colons or special chars, YAML breaks #}
key: {{ value }}

{# ✅ Always quote dynamic values in YAML templates #}
key: "{{ value }}"
```

### Pitfall 4: Whitespace Control 🔥
```jinja
{# ❌ Extra blank lines everywhere #}
{% if enabled %}
value: true
{% endif %}

{# ✅ Use minus sign to strip whitespace #}
{%- if enabled %}
value: true
{%- endif %}

{# Or set trim_blocks=True and lstrip_blocks=True in Environment #}
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
JINJA2 = THE FIRM'S LETTER TEMPLATES:
  "Dear {{ client_name }}," = variable insertion (like f-strings)
  
  {% if vip %}                = conditional sections
    "As a valued client..."
  {% else %}
    "Dear client..."
  {% endif %}

  {% for case in cases %}     = repeated sections
    "Re: Case {{ case.id }}"
  {% endfor %}

  {{ amount | currency }}     = formatting (filters)

  ONE template → 1000 unique letters
  ONE deployment.yaml.j2 → 36 Kubernetes manifests

DONNA'S RULE:
  "If you're copying and pasting, you're doing it wrong.
   Template it. One source of truth. Zero drift."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

### 🟢 Beginner
1. **Hello Template**: Render `"Hello, {{ name }}!"` with different names.
2. **Config File**: Generate a JSON config with 5+ variables from a template.
3. **Loop Practice**: Generate a list of 10 server entries using `{% for %}`.

### 🟡 Intermediate
4. **Nginx Generator**: Generate an Nginx config with upstream, SSL, and locations.
5. **K8s Manifests**: Generate Deployment + Service from one set of variables.
6. **Environment Overlay**: Render the same template for dev/staging/prod with different values.

### 🔴 Advanced
7. **Template Inheritance**: Create base + child templates for K8s resources.
8. **Multi-Service Generator**: Generate all configs for 10 services × 3 environments.
9. **Macro Library**: Build reusable macros for containers, volumes, and env vars.

---

## 9. ✅ Knowledge Check

**Q1.** What is Jinja2?
> A Python templating engine that generates text output (configs, HTML, etc.) from templates with variables, loops, and conditionals.

**Q2.** What does `{{ variable }}` do?
> Inserts the value of `variable` into the output, like an f-string.

**Q3.** How do you loop in Jinja2?
> `{% for item in list %}...{% endfor %}` — identical syntax to Python.

**Q4.** What is a filter?
> A function applied to a value with the pipe syntax: `{{ value | filter }}`. Examples: `default`, `upper`, `join`, `indent`.

**Q5.** Why use Jinja2 instead of f-strings for configs?
> Jinja2 supports loops, conditionals, inheritance, macros, filters, and file-based templates. F-strings only insert values.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Variable substitution (`{{ }}`) | ✅ |
| 2 | Conditionals (`{% if %}`) | ✅ |
| 3 | Loops (`{% for %}`) | ✅ |
| 4 | Filters (`\| default`, `\| join`, etc.) | ✅ |
| 5 | File-based templates (FileSystemLoader) | ✅ |
| 6 | Template inheritance (`{% extends %}`) | ✅ |
| 7 | Strict mode (StrictUndefined) | ✅ |
| 8 | Nginx config generation | ✅ |
| 9 | Macros (reusable template functions) | ✅ |
| 10 | Multi-service config generator | ✅ |

---

> **Next Lesson →** DevOps Lesson 6: *Build CLI Tools with Click and argparse*
