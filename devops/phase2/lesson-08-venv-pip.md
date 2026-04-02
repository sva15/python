# Phase 2 · Lesson 8 — Virtual Environments & pip
### *"The Clean Room"*

> **O'Reilly Skill:** Create and use virtual environments, install packages, and manage dependencies with pip.
> **DevOps connection:** Every Python tool you deploy needs an isolated environment. This is how you avoid "it works on my machine" and how CI/CD pipelines install dependencies consistently.
> **10-min sprint:** Create a virtual environment, install `requests` and `pyyaml`, freeze the requirements, then deactivate. Try running the script outside the venv and see what happens.

---

## Why This Matters (Harvey's take)

Without virtual environments, installing `requests==2.28` for one project breaks another project that needs `requests==2.25`. Virtual environments are isolated boxes — each project gets its own Python and its own packages. It's the difference between "it works on my machine" and "it works everywhere."

---

## The Core Concept (Mike's breakdown)

### Create and Use a Virtual Environment

```bash
# Create a virtual environment (run once)
python -m venv venv

# Activate it
# Linux / macOS:
source venv/bin/activate

# Windows:
venv\Scripts\activate

# You'll see (venv) in your prompt
(venv) $

# Deactivate when done
deactivate
```

### Install Packages

```bash
# Install a package
pip install requests

# Install a specific version
pip install requests==2.31.0

# Install multiple packages
pip install requests pyyaml jinja2 boto3

# Install from requirements file
pip install -r requirements.txt

# Upgrade a package
pip install --upgrade requests

# Uninstall
pip uninstall requests
```

### `requirements.txt` — Reproducible Environments

```bash
# Generate from what's currently installed
pip freeze > requirements.txt

# requirements.txt looks like:
# certifi==2024.2.2
# charset-normalizer==3.3.2
# idna==3.6
# Jinja2==3.1.3
# MarkupSafe==2.1.5
# PyYAML==6.0.1
# requests==2.31.0
# urllib3==2.2.1

# Anyone can reproduce your environment exactly:
pip install -r requirements.txt
```

### Two requirements files — common pattern

```
requirements.txt         ← production dependencies only
requirements-dev.txt     ← also includes dev/test tools
```

```
# requirements.txt
requests==2.31.0
pyyaml==6.0.1
jinja2==3.1.3
boto3==1.34.0
```

```
# requirements-dev.txt
-r requirements.txt        # includes everything from requirements.txt
pytest==8.0.0
black==24.2.0
mypy==1.8.0
```

### What's Inside a Virtual Environment

```
venv/
├── bin/               (Linux/Mac) or Scripts/ (Windows)
│   ├── python         ← local Python interpreter
│   ├── pip            ← local pip
│   └── activate       ← activation script
├── lib/
│   └── python3.12/
│       └── site-packages/
│           ├── requests/
│           ├── yaml/
│           └── ...
└── pyvenv.cfg         ← venv config
```

### Useful pip Commands

```bash
# List installed packages
pip list

# Show details about a package
pip show requests

# Check for outdated packages
pip list --outdated

# Search (limited — use pip.org instead)
pip index versions requests

# Install without cache (useful in CI)
pip install --no-cache-dir -r requirements.txt

# Install editable (for developing a package)
pip install -e .
```

---

## DevOps Patterns

### `.gitignore` — never commit the venv

```gitignore
# Python
venv/
.venv/
env/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
*.egg-info/
dist/
build/
```

### CI/CD Pipeline — standard Python setup

```yaml
# GitHub Actions example
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.12"

- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt

- name: Run tests
  run: pytest tests/
```

### Docker — don't use venv inside Docker

```dockerfile
# In Docker you don't need venv — the container IS the isolation
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
CMD ["python", "main.py"]
```

### Python Version Management

```bash
# pyenv — manage multiple Python versions
pyenv install 3.12.2
pyenv local 3.12.2        # sets .python-version file in directory
pyenv global 3.11.8       # sets system-wide default

# Check what pyenv will use
pyenv which python

# Create venv with specific version
pyenv local 3.12.2
python -m venv venv       # uses pyenv's 3.12.2
```

### `pip-tools` — Better dependency management

```bash
pip install pip-tools

# Create requirements.in (just direct dependencies, no versions)
# requests
# pyyaml
# jinja2

# Compile to requirements.txt with ALL pinned versions
pip-compile requirements.in   # generates requirements.txt

# Update all packages to latest compatible versions
pip-compile --upgrade requirements.in

# Sync environment to exactly match requirements.txt
pip-sync requirements.txt
```

---

## Setting Up a New DevOps Project (the full workflow)

```bash
# 1. Create project directory
mkdir my-devops-tool && cd my-devops-tool

# 2. Create virtual environment
python -m venv venv

# 3. Activate it
source venv/bin/activate

# 4. Upgrade pip (always do this first)
pip install --upgrade pip

# 5. Install your dependencies
pip install requests pyyaml jinja2 boto3

# 6. Create requirements files
pip freeze > requirements.txt

# 7. Create .gitignore
echo "venv/" > .gitignore
echo "__pycache__/" >> .gitignore
echo "*.pyc" >> .gitignore

# 8. Initialize git
git init
git add .
git commit -m "Initial project setup"

# Now anyone can reproduce your environment:
# git clone <repo>
# python -m venv venv
# source venv/bin/activate
# pip install -r requirements.txt
```

---

## Gotchas (Louis's warnings)

**1. Always activate before installing:**
```bash
# ❌ Installs into system Python — could break other tools
pip install requests

# ✅ Activate first, then install
source venv/bin/activate
pip install requests
```

**2. Never commit `venv/` to git:**
```bash
# Check your .gitignore includes venv/
cat .gitignore | grep venv

# Remove if accidentally committed
git rm -r --cached venv/
echo "venv/" >> .gitignore
```

**3. `pip freeze` includes EVERYTHING — use `pip-tools` for cleaner files:**
```
# pip freeze output (messy — includes all transitive deps)
certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
requests==2.31.0
urllib3==2.2.1

# requirements.in with pip-tools (clean — just what you actually need)
requests
```

**4. Don't use `sudo pip install` — ever:**
```bash
# ❌ Modifies system Python — breaks other system tools
sudo pip install requests

# ✅ Use a virtual environment (or --user as last resort)
source venv/bin/activate
pip install requests
```

**5. Different `venv` activation on Windows:**
```bash
# Linux/Mac:
source venv/bin/activate

# Windows Command Prompt:
venv\Scripts\activate.bat

# Windows PowerShell:
venv\Scripts\Activate.ps1
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Create a virtual environment for a new project. Install `requests`, `pyyaml`, and `jinja2`. Freeze to `requirements.txt`. Open the file and count how many packages got listed (including transitive dependencies).

**Intermediate:** Create two virtual environments: `venv-old` with `requests==2.28.0` and `venv-new` with `requests==2.31.0`. In each, run a script that prints the requests version. Show that they're independent.

**DevOps-specific:** Write a `setup.sh` script that: creates a venv if it doesn't exist, activates it, upgrades pip, installs from `requirements.txt`, and runs a health check script. Make it idempotent (safe to run multiple times).

---

## Knowledge Check

1. Why use virtual environments?
2. How do you activate a virtual environment on Linux?
3. What does `pip freeze > requirements.txt` do?
4. How does someone reproduce your environment on a new machine?
5. Why shouldn't you commit the `venv/` directory to git?

**Answers:** (1) Isolates each project's dependencies so they don't conflict with each other or system tools. (2) `source venv/bin/activate`. (3) Writes all currently installed packages with exact versions to `requirements.txt`. (4) `python -m venv venv` → `source venv/bin/activate` → `pip install -r requirements.txt`. (5) It's large (10-100 MB), contains binaries, and is platform-specific — it would bloat git and break on other OSes.

---

> **Next:** Lesson 9 — Debugging → When production shows wrong numbers and no one knows why, this is how you find it.
