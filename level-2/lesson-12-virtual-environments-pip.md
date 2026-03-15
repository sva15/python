# 🏛️ Level 2 – Applying | Lesson 12: Use Virtual Environments and Pip in Python

## *"The Clean Room"*

> **O'Reilly Skill Objective:** *Create virtual environments, install packages with pip, and manage project dependencies.*

> **Previously Learned:** Variables, data types, functions, classes, modules, `import`, standard library, file I/O, JSON, dataclasses

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

*Friday morning. Chaos in the IT department.*

**Louis:** "I updated `requests` for my project and now Harvey's billing script is BROKEN."

**Harvey:** "What do you mean broken? I didn't change anything!"

**Mike:** "That's the problem. Louis's project needed `requests==2.31`, but Harvey's billing script requires `requests==2.28`. Since both projects share the same global Python installation, updating one broke the other."

**Jessica:** "Fix this. And make sure it never happens again."

**Mike:** "Virtual environments. Each project gets its own isolated Python installation with its own packages. Louis can use `requests==2.31`, Harvey keeps `requests==2.28`, and neither can break the other."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Think of it as office space."

| Concept | Analogy |
|---------|---------|
| **Global Python** | One big shared open-plan office — everyone trips over each other |
| **Virtual environment** | Private offices — each team has their own space, supplies, and tools |
| **pip** | The supply ordering system — request what you need |
| **requirements.txt** | The supply manifest — a list of exactly what's in the office |

**Without virtual environments:**
```
GLOBAL PYTHON (shared by ALL projects):
  📦 requests==2.28
  📦 flask==2.3
  📦 numpy==1.24
  
  Project A needs requests==2.31 → Upgrade breaks Project B!
  Project B needs requests==2.28 → Downgrade breaks Project A!
```

**With virtual environments:**
```
Project A (own venv):           Project B (own venv):
  📦 requests==2.31              📦 requests==2.28
  📦 flask==3.0                  📦 flask==2.3
  📦 pandas==2.1                 📦 numpy==1.24
  
  ✅ Independent!                ✅ Independent!
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Creating a Virtual Environment ⭐⭐

```bash
# Navigate to your project directory
cd my_project

# Create a virtual environment named "venv"
python -m venv venv

# What this creates:
# my_project/
# ├── venv/                    ← The virtual environment
# │   ├── Scripts/             ← (Windows) or bin/ (Mac/Linux)
# │   │   ├── python.exe       ← Isolated Python interpreter
# │   │   ├── pip.exe          ← Isolated pip
# │   │   └── activate         ← Activation script
# │   ├── Lib/
# │   │   └── site-packages/   ← Packages installed HERE (isolated)
# │   └── pyvenv.cfg           ← Environment configuration
# ├── main.py                  ← Your code
# └── requirements.txt         ← Dependency list
```

---

### 3.2 Activating and Deactivating ⭐

```bash
# === WINDOWS (PowerShell) ===
.\venv\Scripts\Activate.ps1

# === WINDOWS (CMD) ===
.\venv\Scripts\activate.bat

# === macOS / Linux ===
source venv/bin/activate

# When activated, your prompt changes:
# (venv) PS C:\Projects\my_project>
#  ^^^^^ This means the venv is active!

# Verify you're using the venv's Python:
python --version
where python        # Windows
which python        # Mac/Linux
# Should point to: venv/Scripts/python.exe (or venv/bin/python)

# === DEACTIVATE ===
deactivate
# Prompt returns to normal — back to global Python
```

**Mike:** "When a venv is activated:"
- `python` → runs the venv's Python
- `pip` → installs into the venv's `site-packages`
- Packages installed globally are NOT visible (clean slate!)

---

### 3.3 `pip` — Package Installation ⭐⭐

```bash
# Always activate your venv first!
# (venv) PS C:\Projects\my_project>

# Install a package
pip install requests

# Install a specific version
pip install requests==2.31.0

# Install minimum version
pip install "requests>=2.28"

# Install multiple packages
pip install requests flask pandas

# Upgrade a package
pip install --upgrade requests

# Uninstall a package
pip install requests

# See what's installed
pip list

# Show package details
pip show requests

# Check for outdated packages
pip list --outdated
```

---

### 3.4 `requirements.txt` — The Dependency Manifest ⭐⭐

```bash
# Generate requirements.txt from current environment
pip freeze > requirements.txt

# Install all dependencies from requirements.txt
pip install -r requirements.txt
```

**Contents of `requirements.txt`:**
```
# requirements.txt — Pearson Specter Litt Billing System
# Generated with: pip freeze > requirements.txt

certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
requests==2.31.0
urllib3==2.2.1
flask==3.0.2
Werkzeug==3.0.1
```

**Mike:** "Two styles of requirements files:"

```
# Style 1: Exact versions (pip freeze output) — for deployment
requests==2.31.0
flask==3.0.2

# Style 2: Flexible versions — for libraries/development
requests>=2.28,<3.0
flask>=3.0
pandas~=2.1        # Compatible release (~= means >=2.1, <3.0)
```

| Specifier | Meaning | Example |
|-----------|---------|---------|
| `==2.31.0` | Exactly this version | `requests==2.31.0` |
| `>=2.28` | This version or newer | `requests>=2.28` |
| `<3.0` | Older than this version | `requests<3.0` |
| `>=2.28,<3.0` | Range | `requests>=2.28,<3.0` |
| `~=2.1` | Compatible (≥2.1, <3.0) | `pandas~=2.1` |
| `!=2.30.0` | Not this specific version | `requests!=2.30.0` |

---

### 3.5 The Complete Workflow ⭐

```bash
# ============================================
# STARTING A NEW PROJECT — Full Workflow
# ============================================

# 1. Create project directory
mkdir billing_system
cd billing_system

# 2. Create virtual environment
python -m venv venv

# 3. Activate it
.\venv\Scripts\Activate.ps1          # Windows PowerShell
# source venv/bin/activate           # Mac/Linux

# 4. Install packages
pip install requests flask python-dotenv

# 5. Freeze dependencies
pip freeze > requirements.txt

# 6. Create .gitignore (NEVER commit the venv!)
echo venv/ > .gitignore

# 7. Write your code
# ... develop ...

# 8. When done, deactivate
deactivate

# ============================================
# JOINING AN EXISTING PROJECT — Clone & Setup
# ============================================

# 1. Clone the repo
git clone https://github.com/psl/billing_system.git
cd billing_system

# 2. Create YOUR virtual environment
python -m venv venv

# 3. Activate it
.\venv\Scripts\Activate.ps1

# 4. Install from requirements.txt
pip install -r requirements.txt

# 5. Start working!
python main.py
```

---

### 3.6 `.gitignore` — What NOT to Commit

```gitignore
# .gitignore for Python projects

# Virtual environment — NEVER commit this!
venv/
.venv/
env/
.env/

# Python cache
__pycache__/
*.pyc
*.pyo

# Distribution / packaging
dist/
build/
*.egg-info/

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Environment variables
.env
```

**Mike:** "The venv folder is typically 50-200 MB. NEVER commit it to git. Anyone can recreate it from `requirements.txt`."

---

### 3.7 Multiple Requirements Files

```bash
# requirements.txt — production dependencies
flask==3.0.2
requests==2.31.0
gunicorn==21.2.0

# requirements-dev.txt — development extras
-r requirements.txt       # Include production deps
pytest==8.0.1
black==24.1.1
flake8==7.0.0
mypy==1.8.0

# Install for development:
pip install -r requirements-dev.txt
```

---

### 3.8 `pip` Deep Dive — Advanced Usage

```bash
# Install from a Git repository
pip install git+https://github.com/user/repo.git

# Install in editable/development mode (your changes are live)
pip install -e .

# Install with extras
pip install "requests[security]"

# Download without installing
pip download requests -d ./packages

# Install from local wheel
pip install ./packages/requests-2.31.0-py3-none-any.whl

# Check for dependency conflicts
pip check

# Show dependency tree
pip install pipdeptree
pipdeptree
```

---

### 3.9 Understanding `pip install` — What Actually Happens

```python
"""
When you run: pip install requests

1. pip contacts PyPI (Python Package Index — pypi.org)
2. Downloads the package (wheel or source distribution)
3. Resolves dependencies (requests needs urllib3, certifi, etc.)
4. Downloads dependencies too
5. Installs everything into:
   - If venv active: venv/Lib/site-packages/
   - If no venv: global Python site-packages/ (AVOID!)
6. Creates a metadata directory for uninstallation

Package index:
  pypi.org = "The Python supply warehouse"
  300,000+ packages available
  Free and open source
"""

# See where packages are installed:
import site
print(site.getsitepackages())

# See where a specific package lives:
import requests
print(requests.__file__)
```

---

### 3.10 Naming Conventions

```bash
# Virtual environment naming:
python -m venv venv         # Most common — simple "venv"
python -m venv .venv        # Hidden directory (dot prefix)
python -m venv env          # Also common
python -m venv .env         # Hidden — but conflicts with dotenv!

# Recommendation: use "venv" or ".venv"
```

---

### 3.11 Solving the Firm's Problem — Isolated Project Setup

```python
"""
============================================
Pearson Specter Litt — Project Setup Script
Automated virtual environment management
============================================

Save this as: setup_project.py
Run: python setup_project.py
"""

import subprocess
import sys
import os
from pathlib import Path

def setup_project(project_name, packages=None):
    """Set up a new Python project with virtual environment."""
    
    project_dir = Path(project_name)
    venv_dir = project_dir / "venv"
    
    print(f"\n{'='*50}")
    print(f"{'🏛️  PROJECT SETUP':^50}")
    print(f"{'='*50}")
    
    # Step 1: Create project directory
    print(f"\n📁 Creating project: {project_name}")
    project_dir.mkdir(exist_ok=True)
    
    # Step 2: Create virtual environment
    print(f"🐍 Creating virtual environment...")
    subprocess.run([sys.executable, "-m", "venv", str(venv_dir)], check=True)
    
    # Step 3: Determine pip path
    if sys.platform == "win32":
        pip_path = venv_dir / "Scripts" / "pip.exe"
        python_path = venv_dir / "Scripts" / "python.exe"
    else:
        pip_path = venv_dir / "bin" / "pip"
        python_path = venv_dir / "bin" / "python"
    
    # Step 4: Upgrade pip
    print(f"⬆️  Upgrading pip...")
    subprocess.run([str(pip_path), "install", "--upgrade", "pip"], 
                   capture_output=True, check=True)
    
    # Step 5: Install packages
    if packages:
        print(f"📦 Installing packages: {', '.join(packages)}")
        subprocess.run([str(pip_path), "install"] + packages, check=True)
    
    # Step 6: Freeze requirements
    req_file = project_dir / "requirements.txt"
    result = subprocess.run([str(pip_path), "freeze"], capture_output=True, text=True)
    req_file.write_text(result.stdout)
    print(f"📋 Created requirements.txt ({len(result.stdout.strip().splitlines())} packages)")
    
    # Step 7: Create .gitignore
    gitignore = project_dir / ".gitignore"
    gitignore.write_text(
        "venv/\n__pycache__/\n*.pyc\n.env\n*.egg-info/\ndist/\nbuild/\n"
    )
    print(f"🚫 Created .gitignore")
    
    # Step 8: Create main.py
    main_file = project_dir / "main.py"
    main_file.write_text(
        f'"""\n{project_name} — Pearson Specter Litt\n"""\n\n'
        f'def main():\n    print("Welcome to {project_name}!")\n\n'
        f'if __name__ == "__main__":\n    main()\n'
    )
    print(f"📄 Created main.py")
    
    # Step 9: Create README
    readme = project_dir / "README.md"
    readme.write_text(
        f"# {project_name}\n\n"
        f"## Setup\n\n"
        f"```bash\n"
        f"# Create and activate virtual environment\n"
        f"python -m venv venv\n"
        f"{'.' + chr(92) + 'venv' + chr(92) + 'Scripts' + chr(92) + 'Activate.ps1' if sys.platform == 'win32' else 'source venv/bin/activate'}\n\n"
        f"# Install dependencies\n"
        f"pip install -r requirements.txt\n\n"
        f"# Run\n"
        f"python main.py\n"
        f"```\n"
    )
    print(f"📖 Created README.md")
    
    # Summary
    print(f"\n{'='*50}")
    print(f"✅ Project '{project_name}' is ready!")
    print(f"\nTo get started:")
    print(f"  cd {project_name}")
    if sys.platform == "win32":
        print(f"  .\\venv\\Scripts\\Activate.ps1")
    else:
        print(f"  source venv/bin/activate")
    print(f"  python main.py")
    print(f"{'='*50}")
    
    return project_dir


# === Create firm projects with isolated environments ===
if __name__ == "__main__":
    # Example: Set up the billing system
    setup_project(
        "billing_system",
        packages=["requests", "python-dotenv"]
    )
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting to Activate the venv

```bash
# ❌ Installs into GLOBAL Python!
pip install requests

# ✅ Activate first, THEN install
.\venv\Scripts\Activate.ps1
pip install requests

# How to check: look at your prompt
# (venv) PS C:\...>     ← Activated ✅
# PS C:\...>             ← NOT activated ❌

# Or check where pip points:
pip --version
# Should show: .../venv/lib/... (not global)
```

---

### Pitfall 2: Committing the venv to Git 🔥

```bash
# ❌ The venv/ folder is 50-200 MB of binary files!
git add .
git commit -m "Added project"    # Commits entire venv!

# ✅ Add .gitignore BEFORE your first commit
echo "venv/" > .gitignore
git add .gitignore
git add .
git commit -m "Initial commit"
```

---

### Pitfall 3: PowerShell Execution Policy (Windows)

```bash
# ❌ Error: cannot be loaded because running scripts is disabled
.\venv\Scripts\Activate.ps1

# ✅ Fix: Allow script execution (run once as admin)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# ✅ Alternative: use CMD instead
.\venv\Scripts\activate.bat
```

---

### Pitfall 4: `pip freeze` Captures Everything

```bash
# pip freeze includes ALL installed packages, including dependencies
pip install requests
pip freeze > requirements.txt

# requirements.txt now has:
# certifi==2024.2.2
# charset-normalizer==3.3.2
# idna==3.6
# requests==2.31.0        ← You wanted this
# urllib3==2.2.1

# 5 packages! You only installed 1!
# This is usually fine — but for cleaner files:

# ✅ Option: Manually maintain a minimal requirements.txt
# requirements.txt:
# requests>=2.31

# And use a separate requirements-lock.txt for exact pinning
pip freeze > requirements-lock.txt
```

---

### Pitfall 5: Python Version Mismatch

```bash
# ❌ Created venv with Python 3.11, but now running Python 3.12
python -m venv venv    # Uses current Python version

# If you upgrade Python, the old venv may break!
# ✅ Recreate the venv with the new Python
rm -rf venv                              # Delete old venv
python -m venv venv                      # Create new with current Python
pip install -r requirements.txt          # Reinstall packages
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏢 Analogy: Private Offices vs Open Plan

```
WITHOUT VIRTUAL ENVIRONMENTS (open plan office):
  ┌──────────────────────────────┐
  │  🏢 ONE SHARED SPACE         │
  │  📦 requests==2.28           │
  │  📦 flask==2.3               │
  │                              │
  │  Harvey's project: needs 2.28│
  │  Louis's project: needs 2.31 │
  │  💥 CONFLICT!                │
  └──────────────────────────────┘

WITH VIRTUAL ENVIRONMENTS (private offices):
  ┌──────────────┐ ┌──────────────┐
  │ 🚪 Harvey's  │ │ 🚪 Louis's   │
  │ venv/        │ │ venv/        │
  │ 📦 req 2.28  │ │ 📦 req 2.31  │
  │ 📦 flask 2.3 │ │ 📦 flask 3.0 │
  │ ✅ Isolated! │ │ ✅ Isolated! │
  └──────────────┘ └──────────────┘
  
  requirements.txt = Supply order form
  pip install = Delivery to your office
  pip freeze = Inventory of your office
  .gitignore = "Don't ship the office, ship the order form"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: First Virtual Environment**
```
a) Create a new project directory
b) Create a virtual environment
c) Activate it
d) Verify it's active (check prompt + which python)
e) Install requests
f) Run: python -c "import requests; print(requests.__version__)"
g) Freeze requirements
h) Deactivate
```

**Exercise 2: Clone and Setup**
```
Pretend you're joining an existing project:
a) Create a project with a requirements.txt containing:
   requests==2.31.0
   flask==3.0.2
b) Create a new venv
c) Install from requirements.txt
d) Verify all packages are installed
e) Run pip check for conflicts
```

**Exercise 3: Multiple Environments**
```
Create two projects with DIFFERENT versions:
  Project A: requests==2.28.0
  Project B: requests==2.31.0

Verify they're truly independent:
  Activate A, check version
  Activate B, check version
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Dev vs Production**
```
Create a project with two requirements files:
  requirements.txt (production):
    flask>=3.0
    gunicorn>=21.0
    
  requirements-dev.txt (development):
    -r requirements.txt
    pytest>=8.0
    black>=24.0
    mypy>=1.8

a) Install dev requirements
b) Verify all packages (prod + dev) are present
c) Create a new "production" venv with only prod requirements
d) Verify dev tools are NOT in production
```

**Exercise 5: Package Explorer**
```
Write a script that:
a) Lists all installed packages and their versions
b) Shows the dependency tree for a given package
c) Checks which packages are outdated
d) Calculates total size of site-packages

Use: pkg_resources or importlib.metadata
```

**Exercise 6: Automated Setup Script**
```
Write setup.py that:
a) Creates venv if it doesn't exist
b) Activates (subprocess)
c) Installs from requirements.txt
d) Runs basic tests
e) Reports success/failure

The user should run ONE command to set up everything.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Dependency Resolver**
```
Build a tool that:
a) Reads requirements.txt
b) Checks for version conflicts
c) Suggests compatible versions
d) Generates a lock file (exact versions)
e) Compares two requirements files for differences
```

**Exercise 8: Project Template Generator**
```
Build a CLI tool that creates project templates:
  python create_project.py --name myapp --type web

Generated structure:
  myapp/
  ├── venv/
  ├── src/
  │   └── myapp/
  │       ├── __init__.py
  │       └── main.py
  ├── tests/
  │   └── test_main.py
  ├── requirements.txt
  ├── requirements-dev.txt
  ├── .gitignore
  ├── README.md
  └── setup.py

Support multiple project types: web, cli, library, data-science
```

**Exercise 9: Environment Manager**
```
Build an environment manager (mini conda/pyenv):
  envmgr create myproject --python 3.11
  envmgr list
  envmgr activate myproject
  envmgr install myproject requests flask
  envmgr freeze myproject
  envmgr delete myproject

Track all environments in a central config file.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```bash
# Bug 1: ModuleNotFoundError despite pip install
pip install requests
python main.py
# ModuleNotFoundError: No module named 'requests'
# Reason: pip installed globally, but python runs from venv (or vice versa)

# Bug 2: Permission denied
pip install flask
# ERROR: Could not install packages due to an EnvironmentError

# Bug 3: Wrong Python version
python -m venv venv
# Error: Python 3.6 doesn't support some features

# Bug 4: Broken venv after Python upgrade
.\venv\Scripts\python.exe
# Fatal error: Python installation not found

# Bug 5: Packages from other project leaking in
# Can import pandas even though it's not in requirements.txt
```

### Fixed Code ✅

```bash
# Fix 1: Ensure pip and python are from the SAME environment
.\venv\Scripts\Activate.ps1    # Activate first!
pip install requests            # Now installs in venv
python main.py                  # Now runs from venv — finds requests!
# Verify: pip --version and python --version should both show venv paths

# Fix 2: Use --user flag or fix venv permissions
pip install --user flask        # Install for current user only
# Or better: use a venv where you have full permissions

# Fix 3: Specify Python version
py -3.11 -m venv venv          # Windows: use py launcher
python3.11 -m venv venv        # Mac/Linux: specify version

# Fix 4: Recreate the venv
rm -rf venv                    # Delete broken venv
python -m venv venv            # Create fresh
pip install -r requirements.txt # Reinstall from manifest

# Fix 5: Venv needs --no-site-packages (default since Python 3)
python -m venv venv            # Already isolated by default
# If you see global packages, check:
# venv/pyvenv.cfg — "include-system-site-packages = false"
```

---

## 8. 🌍 Real-World Application

### The Professional Workflow

```
SOLO DEVELOPER:
  1. python -m venv venv
  2. source venv/bin/activate
  3. pip install <packages>
  4. pip freeze > requirements.txt
  5. Add venv/ to .gitignore
  6. git commit

TEAM COLLABORATION:
  Developer A:
    1. Creates project, sets up venv
    2. Installs packages
    3. pip freeze > requirements.txt
    4. Commits requirements.txt (NOT venv/)
    5. Pushes to git
  
  Developer B:
    1. git clone <repo>
    2. python -m venv venv
    3. pip install -r requirements.txt
    4. Identical environment! ✅

CI/CD PIPELINE:
  1. Clone repo
  2. python -m venv venv
  3. pip install -r requirements.txt
  4. python -m pytest
  5. If tests pass → deploy
```

### Tool Comparison

| Tool | Purpose | Complexity |
|------|---------|:---:|
| **venv** | Built-in, simple | ⭐ |
| **pip** | Package installer | ⭐ |
| **virtualenv** | Enhanced venv (legacy) | ⭐⭐ |
| **pipenv** | pip + venv combined | ⭐⭐⭐ |
| **poetry** | Modern dependency management | ⭐⭐⭐ |
| **conda** | Python + non-Python packages | ⭐⭐⭐⭐ |
| **uv** | Ultra-fast pip replacement | ⭐⭐ |

> **Mike:** "Start with `venv` + `pip` + `requirements.txt`. They're built into Python and work everywhere. Graduate to `poetry` or `uv` when you need more power."

---

## 9. ✅ Knowledge Check

---

**Q1.** What problem do virtual environments solve?

**Q2.** What command creates a virtual environment?

**Q3.** How do you activate a venv on Windows vs Mac/Linux?

**Q4.** What is the difference between `pip install` with and without an active venv?

**Q5.** What does `pip freeze > requirements.txt` do?

**Q6.** How does a teammate recreate your exact environment?

**Q7.** Why should you NEVER commit the `venv/` folder to git?

**Q8.** What is the difference between `requirements.txt` and `requirements-dev.txt`?

**Q9.** What happens if you delete the `venv/` folder?

**Q10.** What's the first thing you should do when cloning a Python project?

---

### 📋 Answer Key

> **Q1.** Dependency isolation — prevents version conflicts between projects sharing the same Python installation.

> **Q2.** `python -m venv venv`

> **Q3.** Windows: `.\venv\Scripts\Activate.ps1` (PowerShell) or `.\venv\Scripts\activate.bat` (CMD). Mac/Linux: `source venv/bin/activate`.

> **Q4.** With venv: installs into `venv/Lib/site-packages/` (isolated). Without: installs into global Python (shared, risky).

> **Q5.** Captures the exact versions of all installed packages and writes them to `requirements.txt`.

> **Q6.** `python -m venv venv && pip install -r requirements.txt` — creates a fresh venv and installs the same packages.

> **Q7.** It's 50-200 MB of binary files, platform-specific, and can be recreated from `requirements.txt`.

> **Q8.** `requirements.txt` = production dependencies only. `requirements-dev.txt` = production + development tools (testing, linting, formatting).

> **Q9.** Nothing bad! Your source code is untouched. Recreate it: `python -m venv venv && pip install -r requirements.txt`.

> **Q10.** Create a virtual environment and install from `requirements.txt`.

---

## 🎬 End of Lesson Scene

**Louis:** "My project uses `requests==2.31`. Harvey's uses `requests==2.28`. Both work. Neither breaks the other."

**Harvey:** "Clean rooms. Isolated environments. No contamination."

**Mike:** "And if anyone new joins the team, one command: `pip install -r requirements.txt`. Identical setup, guaranteed. Next: debugging Python code with print statements and tools."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Why virtual environments matter | ✅ |
| 2 | Creating venvs with `python -m venv` | ✅ |
| 3 | Activating and deactivating | ✅ |
| 4 | `pip install`, `pip freeze`, `pip list` | ✅ |
| 5 | `requirements.txt` creation and usage | ✅ |
| 6 | Version specifiers (`==`, `>=`, `~=`) | ✅ |
| 7 | Complete project workflow | ✅ |
| 8 | `.gitignore` for Python | ✅ |
| 9 | Dev vs production requirements | ✅ |
| 10 | Troubleshooting common issues | ✅ |

---

> **Next Lesson →** Level 2, Lesson 13: *Debug Python Code with Print Statements and Tools*
