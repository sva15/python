# Phase 2 · Lesson 7 — Jinja2 Templating
### *"The Brief Writer"*

> **DevOps Objective:** Generate nginx configs, Kubernetes manifests, Ansible playbooks, and any text file from Python templates using Jinja2.
> **DevOps connection:** Ansible uses Jinja2. Helm templates use Jinja2-like syntax. If you've ever seen `{{ variable }}` in a config file, that's Jinja2.
> **10-min sprint:** Write a Jinja2 template for an nginx server block with a variable domain, port, and backend host. Render it for 3 different services and print each result.

---

## Setup

```bash
pip install jinja2
```

---

## Why This Matters

You have 10 microservices. Each needs an nginx config, a K8s deployment manifest, and a systemd unit file. They're 95% the same — just the service name, port, and image change. Instead of maintaining 30 nearly-identical files, you write 3 templates and generate all 30 files from Python. Change a template → all 30 update.

---

## The Core Concept

### Basic Template — f-strings on steroids

```python
from jinja2 import Template

template_str = """
server {
    listen {{ port }};
    server_name {{ domain }};

    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
"""

template = Template(template_str)
result = template.render(
    port=443,
    domain="billing.psl.com",
    backend_host="10.0.1.50",
    backend_port=8080,
)
print(result)
```

**Output:**
```nginx
server {
    listen 443;
    server_name billing.psl.com;

    location / {
        proxy_pass http://10.0.1.50:8080;
    }
}
```

### Jinja2 Syntax

```
{{ variable }}          → insert value (like f-string)
{% for item in list %}  → loop
{% if condition %}      → conditional
{% endif %}             → end if
{% endfor %}            → end for
{# comment #}           → comment (not in output)
{{ value | filter }}    → apply a filter to a value
```

### Loops in Templates

```python
from jinja2 import Template

template = Template("""
upstream backend {
{% for server in servers %}
    server {{ server.host }}:{{ server.port }} weight={{ server.weight }};
{% endfor %}
}
""")

result = template.render(servers=[
    {"host": "10.0.1.10", "port": 8080, "weight": 3},
    {"host": "10.0.1.11", "port": 8080, "weight": 2},
    {"host": "10.0.1.12", "port": 8080, "weight": 1},
])
print(result)
```

### Conditionals in Templates

```python
template = Template("""
server {
    listen 80;
{% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/{{ domain }}.crt;
    ssl_certificate_key /etc/ssl/{{ domain }}.key;
{% endif %}
    server_name {{ domain }};
{% if redirect_www %}
    if ($host = www.{{ domain }}) {
        return 301 https://{{ domain }}$request_uri;
    }
{% endif %}
}
""")

print(template.render(
    domain="psl.com",
    ssl_enabled=True,
    redirect_www=True,
))
```

### Filters — transform values

```python
from jinja2 import Template

template = Template("""
Service: {{ name | upper }}
Image:   {{ image | lower }}
Slug:    {{ name | lower | replace(" ", "-") }}
Items:   {{ items | length }}
First:   {{ items | first }}
Last:    {{ items | last }}
Joined:  {{ items | join(", ") }}
Default: {{ missing_var | default("N/A") }}
""")

result = template.render(
    name="Billing API",
    image="Registry.PSL.COM/billing:1.2.3",
    items=["alpha", "beta", "gamma"],
)
print(result)
```

### Loading Templates from Files

```python
from jinja2 import Environment, FileSystemLoader

# Load templates from a directory
env = Environment(loader=FileSystemLoader("templates/"))

# Render a specific template file
template = env.get_template("deployment.yaml.j2")
result = template.render(
    service_name="billing-api",
    image="registry.psl.com/billing:1.2.3",
    replicas=3,
    namespace="production",
)

print(result)
```

**templates/deployment.yaml.j2:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ service_name }}
  namespace: {{ namespace | default("default") }}
  labels:
    app: {{ service_name }}
    version: {{ image.split(":")[-1] }}
spec:
  replicas: {{ replicas | default(1) }}
  selector:
    matchLabels:
      app: {{ service_name }}
  template:
    spec:
      containers:
        - name: {{ service_name }}
          image: {{ image }}
{% if env_vars %}
          env:
{% for key, value in env_vars.items() %}
            - name: {{ key }}
              value: "{{ value }}"
{% endfor %}
{% endif %}
```

---

## DevOps Example — Config Generator for Multiple Services

```python
import yaml
import logging
from pathlib import Path
from jinja2 import Environment, FileSystemLoader

logger = logging.getLogger(__name__)

def generate_configs(services_file: str, template_dir: str, output_dir: str):
    """
    Generate nginx + K8s configs for all services defined in a YAML file.
    """
    # Load service definitions
    with open(services_file) as f:
        services = yaml.safe_load(f)

    # Set up Jinja2
    env = Environment(
        loader=FileSystemLoader(template_dir),
        trim_blocks=True,      # removes first newline after block tags
        lstrip_blocks=True,    # strips leading whitespace from block tags
    )

    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    generated = []
    for service in services["services"]:
        name = service["name"]
        logger.info(f"Generating configs for: {name}")

        # Nginx config
        nginx_template = env.get_template("nginx.conf.j2")
        nginx_output = output_path / f"{name}.nginx.conf"
        nginx_output.write_text(nginx_template.render(**service))
        generated.append(str(nginx_output))

        # K8s deployment
        k8s_template = env.get_template("deployment.yaml.j2")
        k8s_output = output_path / f"{name}-deployment.yaml"
        k8s_output.write_text(k8s_template.render(**service))
        generated.append(str(k8s_output))

        logger.info(f"  Generated: {nginx_output.name}, {k8s_output.name}")

    logger.info(f"Total files generated: {len(generated)}")
    return generated


# services.yaml:
"""
services:
  - name: billing-api
    domain: billing.psl.com
    image: registry.psl.com/billing:1.2.3
    replicas: 3
    port: 8080
    namespace: production
    ssl_enabled: true
    env_vars:
      DB_HOST: postgres.prod.svc
      LOG_LEVEL: INFO

  - name: client-portal
    domain: portal.psl.com
    image: registry.psl.com/portal:2.1.0
    replicas: 2
    port: 3000
    namespace: production
    ssl_enabled: true
"""
```

---

## Useful Jinja2 Filters Cheat Sheet

| Filter | Example | Output |
|--------|---------|--------|
| `upper` | `{{ "hello" \| upper }}` | `HELLO` |
| `lower` | `{{ "HELLO" \| lower }}` | `hello` |
| `title` | `{{ "hello world" \| title }}` | `Hello World` |
| `length` | `{{ [1,2,3] \| length }}` | `3` |
| `join(sep)` | `{{ list \| join(", ") }}` | `a, b, c` |
| `first` | `{{ list \| first }}` | first item |
| `last` | `{{ list \| last }}` | last item |
| `default(val)` | `{{ x \| default("N/A") }}` | `N/A` if x is undefined |
| `replace(a, b)` | `{{ "a-b" \| replace("-", "_") }}` | `a_b` |
| `int` | `{{ "42" \| int }}` | `42` |
| `float` | `{{ "3.14" \| float }}` | `3.14` |
| `trim` | `{{ "  hi  " \| trim }}` | `hi` |
| `truncate(n)` | `{{ long_str \| truncate(20) }}` | first 20 chars |
| `sort` | `{{ list \| sort }}` | sorted list |
| `unique` | `{{ list \| unique }}` | deduplicated |
| `tojson` | `{{ data \| tojson }}` | JSON string |

---

## Gotchas (Louis's warnings)

**1. Undefined variables silently become empty string by default:**
```python
# ❌ Missing variable → empty string in output, no error
template = Template("Host: {{ hostname }}")
print(template.render())   # "Host: " — hostname is silently empty!

# ✅ Make undefined variables raise an error
from jinja2 import Environment, StrictUndefined
env = Environment(undefined=StrictUndefined)
template = env.from_string("Host: {{ hostname }}")
template.render()   # UndefinedError: 'hostname' is undefined
```

**2. Whitespace control — templates can have extra blank lines:**
```python
# Use trim_blocks=True, lstrip_blocks=True to clean up whitespace
env = Environment(
    loader=FileSystemLoader("templates/"),
    trim_blocks=True,
    lstrip_blocks=True,
)
```

**3. Don't use Jinja2 for HTML without autoescaping — XSS risk:**
```python
# For HTML templates, enable autoescaping
env = Environment(autoescape=True)   # escapes < > & " characters
```

**4. Jinja2 uses `{{ }}` which conflicts with Python f-strings:**
```python
# Don't mix Jinja2 template strings with Python f-strings in the same variable
template_str = f"{{ port }}"   # ❌ Python f-string eats the braces!
template_str = "{{ port }}"    # ✅ regular string
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Write a Jinja2 template for a systemd unit file for a Python app. Variables: `service_name`, `working_dir`, `command`, `user`. Render it for 3 different services.

**Intermediate:** Create a template for a GitHub Actions workflow that runs tests. Variables: `python_version` (list), `test_command`, `env_vars` (dict). Loop over Python versions and env vars in the template.

**DevOps-specific:** Write a multi-environment nginx config generator. Template supports: `upstream` servers (loop), `ssl_enabled` (conditional), custom `locations` (loop). Generate configs for dev, staging, and production from a YAML services file.

---

## Knowledge Check

1. What is the difference between `{{ }}` and `{% %}`?
2. How do you load templates from a directory instead of a string?
3. What does the `default` filter do?
4. What is `StrictUndefined` and when would you use it?
5. What do `trim_blocks` and `lstrip_blocks` do?

**Answers:** (1) `{{ }}` outputs a value; `{% %}` executes a statement (loop, if, etc.). (2) Use `Environment(loader=FileSystemLoader("templates/"))` then `env.get_template("file.j2")`. (3) Provides a fallback value if the variable is undefined. (4) Makes undefined variables raise an error instead of silently becoming empty — use when you want strict validation of template variables. (5) Clean up whitespace around block tags — removes the newline after `{% %}` tags and leading spaces.

---

> **Next:** Lesson 8 — Virtual Environments & pip → Manage dependencies properly so your scripts work everywhere.
