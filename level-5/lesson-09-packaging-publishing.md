# 🏛️ Level 5 – Innovating | Lesson 9: Package and Publish a Python Library

## *"Going Public"*

> **O'Reilly Skill Objective:** *Package and distribute a Python library (e.g., setup.py, PyPI).*

> **Previously Learned:** Modules, imports, `__init__.py`, virtual environments, classes, decorators, testing, documentation, CLI

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

*The firm's internal billing library works perfectly. Other law firms want it. Harvey wants it public.*

```python
# Current state: a folder of Python files
billing/
    calculator.py
    reports.py
    validators.py

# PROBLEM: How does another firm use this?
# ❌ "Just copy the folder" — no versioning, no dependencies, no updates
# ❌ "Clone our repo" — requires Git knowledge, no isolation
```

**Mike:** "We package it. One command installs it anywhere in the world."

```python
# ✅ After packaging:
# pip install pearson-billing
#
# from pearson_billing import Calculator
# calc = Calculator(rate=450)
# invoice = calc.generate("Wayne Enterprises", hours=85)
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A library that can't be installed is a library nobody uses."

| Step | What | Tool |
|------|------|------|
| 1. **Structure** | Organize files properly | Directory layout |
| 2. **Configure** | Define metadata and dependencies | `pyproject.toml` |
| 3. **Build** | Create distributable archive | `python -m build` |
| 4. **Test** | Verify the package works | `pip install -e .` |
| 5. **Publish** | Upload to PyPI | `twine upload` |

| File | Purpose |
|------|---------|
| `pyproject.toml` | **The ONE config file** — metadata, deps, build system |
| `README.md` | Documentation shown on PyPI |
| `LICENSE` | Legal terms (MIT, Apache, etc.) |
| `src/package_name/` | Your actual Python code |
| `tests/` | Test suite |
| `.gitignore` | Files to exclude from version control |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Modern Project Structure ⭐⭐

```
pearson-billing/                  ← Project root (repo name)
├── pyproject.toml                ← THE config file (replaces setup.py)
├── README.md                     ← PyPI documentation
├── LICENSE                       ← MIT, Apache 2.0, etc.
├── .gitignore
├── src/                          ← Source layout (recommended)
│   └── pearson_billing/          ← Package name (importable)
│       ├── __init__.py           ← Package initialization
│       ├── calculator.py         ← Module
│       ├── reports.py            ← Module
│       ├── validators.py         ← Module
│       └── py.typed              ← Marker for type checking
├── tests/                        ← Test suite
│   ├── __init__.py
│   ├── test_calculator.py
│   └── test_reports.py
└── docs/                         ← Documentation (optional)
    └── usage.md
```

```python
# WHY src/ layout?
# 1. Prevents accidentally importing the LOCAL package during testing
# 2. Forces you to install the package to test it (catches packaging bugs)
# 3. Recommended by PyPA (Python Packaging Authority)

# Alternative: flat layout (simpler, still valid)
# pearson-billing/
# ├── pyproject.toml
# ├── pearson_billing/
# │   ├── __init__.py
# │   └── ...
# └── tests/
```

---

### 3.2 `pyproject.toml` — The One Config File ⭐⭐⭐

```toml
# pyproject.toml — ALL project configuration in ONE file
# Replaces: setup.py, setup.cfg, MANIFEST.in, requirements.txt (partially)

[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "pearson-billing"
version = "1.0.0"
description = "Professional billing calculation library for law firms"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
authors = [
    {name = "Mike Ross", email = "mike@pearsonspecterlitt.com"},
]
keywords = ["billing", "legal", "invoice", "law-firm"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Topic :: Office/Business :: Financial",
    "Typing :: Typed",
]

# Dependencies
dependencies = [
    "pydantic>=2.0",
    "rich>=13.0",
]

# Optional dependencies
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "mypy>=1.0",
    "ruff>=0.1.0",
]
docs = [
    "mkdocs>=1.5",
    "mkdocs-material>=9.0",
]

# Entry points (CLI commands)
[project.scripts]
billing = "pearson_billing.cli:main"

# URLs shown on PyPI
[project.urls]
Homepage = "https://github.com/psl/pearson-billing"
Documentation = "https://pearson-billing.readthedocs.io"
Repository = "https://github.com/psl/pearson-billing"
"Bug Tracker" = "https://github.com/psl/pearson-billing/issues"

# Tool configuration (also in pyproject.toml!)
[tool.setuptools.packages.find]
where = ["src"]

[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.mypy]
strict = true

[tool.ruff]
line-length = 100
```

---

### 3.3 Package Code — `__init__.py` and Modules ⭐⭐

```python
# src/pearson_billing/__init__.py
"""Pearson Billing — Professional billing for law firms."""

from pearson_billing.calculator import Calculator, BillingEntry
from pearson_billing.reports import ReportGenerator
from pearson_billing.validators import validate_hours, validate_rate

__version__ = "1.0.0"
__all__ = [
    "Calculator",
    "BillingEntry",
    "ReportGenerator",
    "validate_hours",
    "validate_rate",
]
```

```python
# src/pearson_billing/calculator.py
"""Billing calculation engine."""

from dataclasses import dataclass, field
from datetime import date
from typing import Optional


@dataclass
class BillingEntry:
    """A single billing line item."""
    description: str
    hours: float
    rate: float
    date: date = field(default_factory=date.today)
    
    @property
    def total(self) -> float:
        return self.hours * self.rate
    
    def __post_init__(self):
        if self.hours < 0:
            raise ValueError(f"Hours must be non-negative, got {self.hours}")
        if self.rate < 0:
            raise ValueError(f"Rate must be non-negative, got {self.rate}")


class Calculator:
    """Billing calculator with configurable rate and discount."""
    
    def __init__(self, rate: float = 450.0, discount: float = 0.0):
        if not 0 <= discount <= 1:
            raise ValueError(f"Discount must be 0-1, got {discount}")
        self.rate = rate
        self.discount = discount
        self._entries: list[BillingEntry] = []
    
    def add_entry(self, description: str, hours: float,
                  rate: Optional[float] = None) -> BillingEntry:
        """Add a billing entry."""
        entry = BillingEntry(
            description=description,
            hours=hours,
            rate=rate or self.rate,
        )
        self._entries.append(entry)
        return entry
    
    @property
    def subtotal(self) -> float:
        return sum(e.total for e in self._entries)
    
    @property
    def discount_amount(self) -> float:
        return self.subtotal * self.discount
    
    @property
    def total(self) -> float:
        return self.subtotal - self.discount_amount
    
    @property
    def entries(self) -> list[BillingEntry]:
        return list(self._entries)
    
    def generate(self, client: str, hours: float) -> dict:
        """Quick invoice generation."""
        self.add_entry(f"Legal services for {client}", hours)
        return {
            "client": client,
            "entries": len(self._entries),
            "subtotal": self.subtotal,
            "discount": self.discount_amount,
            "total": self.total,
        }
```

```python
# src/pearson_billing/validators.py
"""Input validation utilities."""

def validate_hours(hours: float) -> float:
    """Validate billing hours."""
    if not isinstance(hours, (int, float)):
        raise TypeError(f"Hours must be numeric, got {type(hours).__name__}")
    if hours < 0:
        raise ValueError(f"Hours must be non-negative, got {hours}")
    if hours > 24:
        raise ValueError(f"Hours cannot exceed 24 per day, got {hours}")
    return float(hours)

def validate_rate(rate: float) -> float:
    """Validate hourly rate."""
    if not isinstance(rate, (int, float)):
        raise TypeError(f"Rate must be numeric, got {type(rate).__name__}")
    if rate < 0:
        raise ValueError(f"Rate must be non-negative, got {rate}")
    return float(rate)
```

---

### 3.4 Writing a Good README.md ⭐

```markdown
# 📦 Pearson Billing

Professional billing calculation library for law firms.

## Installation

```bash
pip install pearson-billing
```

## Quick Start

```python
from pearson_billing import Calculator

calc = Calculator(rate=450, discount=0.1)
calc.add_entry("Contract review", hours=8.5)
calc.add_entry("Court preparation", hours=12.0)

print(f"Subtotal: ${calc.subtotal:,.2f}")
print(f"Discount: ${calc.discount_amount:,.2f}")
print(f"Total:    ${calc.total:,.2f}")
```

## Features

- ✅ Flexible billing calculations
- ✅ Discount support
- ✅ Input validation
- ✅ Type hints throughout
- ✅ 100% test coverage

## License

MIT
```

---

### 3.5 Writing Tests ⭐

```python
# tests/test_calculator.py
import pytest
from pearson_billing import Calculator, BillingEntry

class TestBillingEntry:
    def test_basic_entry(self):
        entry = BillingEntry("Review", hours=8.0, rate=450.0)
        assert entry.total == 3600.0
    
    def test_negative_hours_raises(self):
        with pytest.raises(ValueError, match="non-negative"):
            BillingEntry("Bad", hours=-1, rate=450)
    
    def test_negative_rate_raises(self):
        with pytest.raises(ValueError, match="non-negative"):
            BillingEntry("Bad", hours=1, rate=-100)

class TestCalculator:
    def test_empty_calculator(self):
        calc = Calculator()
        assert calc.total == 0.0
    
    def test_single_entry(self):
        calc = Calculator(rate=450)
        calc.add_entry("Review", hours=10)
        assert calc.total == 4500.0
    
    def test_multiple_entries(self):
        calc = Calculator(rate=400)
        calc.add_entry("Research", hours=5)
        calc.add_entry("Drafting", hours=8)
        assert calc.subtotal == 5200.0
    
    def test_discount(self):
        calc = Calculator(rate=450, discount=0.1)
        calc.add_entry("Review", hours=10)
        assert calc.subtotal == 4500.0
        assert calc.discount_amount == 450.0
        assert calc.total == 4050.0
    
    def test_custom_rate_per_entry(self):
        calc = Calculator(rate=450)
        calc.add_entry("Partner work", hours=5, rate=600)
        assert calc.total == 3000.0
    
    def test_invalid_discount(self):
        with pytest.raises(ValueError):
            Calculator(discount=1.5)
    
    def test_generate_invoice(self):
        calc = Calculator(rate=450)
        invoice = calc.generate("Wayne Enterprises", hours=85)
        assert invoice["client"] == "Wayne Enterprises"
        assert invoice["total"] == 38250.0

# Run: pytest tests/ -v --cov=pearson_billing
```

---

### 3.6 Building the Package ⭐⭐

```bash
# === Prerequisites ===
pip install build twine

# === Build the package ===
# Creates dist/ with two files:
#   - .tar.gz (source distribution)
#   - .whl (wheel — pre-built, preferred)

python -m build

# Output:
# dist/
#   pearson_billing-1.0.0.tar.gz       ← Source dist
#   pearson_billing-1.0.0-py3-none-any.whl  ← Wheel

# === Test locally ===
# Install from the built wheel
pip install dist/pearson_billing-1.0.0-py3-none-any.whl

# Or install in "editable" mode for development
pip install -e ".[dev]"    # Installs package + dev dependencies
```

```python
# Verify the install works
import pearson_billing
print(pearson_billing.__version__)    # 1.0.0

from pearson_billing import Calculator
calc = Calculator(rate=450)
calc.add_entry("Test", hours=1)
print(f"Total: ${calc.total}")    # $450.0
```

---

### 3.7 Publishing to PyPI ⭐⭐

```bash
# === Step 1: Create PyPI account ===
# https://pypi.org/account/register/

# === Step 2: Create API token ===
# PyPI → Account Settings → API Tokens → Add API Token

# === Step 3: Test on TestPyPI first! ===
# TestPyPI is a separate instance for testing
twine upload --repository testpypi dist/*

# Install from TestPyPI to verify:
pip install --index-url https://test.pypi.org/simple/ pearson-billing

# === Step 4: Upload to real PyPI ===
twine upload dist/*

# Enter credentials:
#   Username: __token__
#   Password: pypi-AgEIcH... (your API token)

# === Alternative: use trusted publishing (GitHub Actions) ===
# No credentials needed — GitHub Actions authenticates with PyPI directly!
```

---

### 3.8 Versioning — Semantic Versioning ⭐

```python
# Semantic Versioning: MAJOR.MINOR.PATCH
#
# MAJOR: Breaking changes (1.0.0 → 2.0.0)
#   - Removed a function
#   - Changed a function signature
#   - Renamed a module
#
# MINOR: New features, backward-compatible (1.0.0 → 1.1.0)
#   - Added a new function
#   - Added optional parameters
#   - New module
#
# PATCH: Bug fixes, backward-compatible (1.0.0 → 1.0.1)
#   - Fixed a bug
#   - Updated documentation
#   - Performance improvement

# === Version in one place ===
# Option 1: __init__.py
# __version__ = "1.0.0"

# Option 2: pyproject.toml (single source of truth)
# [project]
# version = "1.0.0"

# Option 3: Dynamic version from git tags
# [project]
# dynamic = ["version"]
# [tool.setuptools.dynamic]
# version = {attr = "pearson_billing.__version__"}

# === Pre-release versions ===
# 1.0.0a1   — Alpha 1
# 1.0.0b1   — Beta 1
# 1.0.0rc1  — Release Candidate 1
# 1.0.0     — Stable release
```

---

### 3.9 Entry Points — CLI Commands ⭐

```python
# pyproject.toml:
# [project.scripts]
# billing = "pearson_billing.cli:main"

# src/pearson_billing/cli.py
"""Command-line interface for Pearson Billing."""

import argparse
import sys
from pearson_billing import Calculator, __version__


def main():
    parser = argparse.ArgumentParser(
        description="Pearson Billing — Generate invoices from the command line"
    )
    parser.add_argument("--version", action="version", version=f"%(prog)s {__version__}")
    
    subparsers = parser.add_subparsers(dest="command")
    
    # 'invoice' command
    invoice_parser = subparsers.add_parser("invoice", help="Generate an invoice")
    invoice_parser.add_argument("client", help="Client name")
    invoice_parser.add_argument("hours", type=float, help="Billable hours")
    invoice_parser.add_argument("--rate", type=float, default=450, help="Hourly rate")
    invoice_parser.add_argument("--discount", type=float, default=0, help="Discount (0-1)")
    
    args = parser.parse_args()
    
    if args.command == "invoice":
        calc = Calculator(rate=args.rate, discount=args.discount)
        invoice = calc.generate(args.client, args.hours)
        
        print(f"╔══════════════════════════════════════╗")
        print(f"║       PEARSON SPECTER LITT           ║")
        print(f"║           INVOICE                    ║")
        print(f"╠══════════════════════════════════════╣")
        print(f"║ Client:   {invoice['client']:<26}║")
        print(f"║ Hours:    {args.hours:<26}║")
        print(f"║ Rate:     ${args.rate:<25,.2f}║")
        print(f"║ Subtotal: ${invoice['subtotal']:<25,.2f}║")
        if invoice['discount'] > 0:
            print(f"║ Discount: -${invoice['discount']:<24,.2f}║")
        print(f"║ TOTAL:    ${invoice['total']:<25,.2f}║")
        print(f"╚══════════════════════════════════════╝")
    else:
        parser.print_help()

if __name__ == "__main__":
    main()

# After pip install:
# $ billing invoice "Wayne Enterprises" 85 --rate 450 --discount 0.1
```

---

### 3.10 CI/CD — Automated Publishing with GitHub Actions ⭐

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  release:
    types: [published]

permissions:
  id-token: write    # Required for trusted publishing

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -e ".[dev]"
      - run: pytest tests/ -v --cov=pearson_billing --cov-report=term-missing

  publish:
    needs: test
    runs-on: ubuntu-latest
    environment: pypi    # Protected environment
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install build
      - run: python -m build
      - uses: pypa/gh-action-pypi-publish@release/v1
        # No credentials needed — trusted publishing!
```

```yaml
# .github/workflows/ci.yml — Run tests on every push
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -e ".[dev]"
      - run: ruff check src/
      - run: mypy src/
      - run: pytest tests/ -v --cov
```

---

### 3.11 The Complete Publishing Workflow

```bash
# === ONE-TIME SETUP ===
# 1. Create project structure
mkdir pearson-billing && cd pearson-billing
mkdir -p src/pearson_billing tests

# 2. Write pyproject.toml, __init__.py, modules, tests, README, LICENSE

# 3. Initialize git
git init
git add .
git commit -m "Initial release v1.0.0"

# 4. Create virtual environment
python -m venv .venv
source .venv/bin/activate    # or .venv\Scripts\activate on Windows

# 5. Install in editable mode
pip install -e ".[dev]"

# 6. Run tests
pytest tests/ -v --cov

# === RELEASE ===
# 7. Update version in pyproject.toml / __init__.py
# 8. Update CHANGELOG.md

# 9. Build
python -m build

# 10. Check the package
twine check dist/*

# 11. Upload to TestPyPI
twine upload --repository testpypi dist/*

# 12. Test install from TestPyPI
pip install --index-url https://test.pypi.org/simple/ pearson-billing

# 13. Upload to real PyPI
twine upload dist/*

# 14. Tag the release
git tag v1.0.0
git push origin v1.0.0

# 🎉 Anyone can now: pip install pearson-billing
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Package Name vs Import Name 🔥

```python
# ❌ Confusing naming
# PyPI name: pearson-billing (hyphen)
# Import name: pearson_billing (underscore)
# These are DIFFERENT namespaces!

# pip install pearson-billing
# import pearson_billing     ← underscores!

# ✅ Convention: 
# PyPI: lowercase with hyphens (pearson-billing)
# Import: lowercase with underscores (pearson_billing)
# Keep them consistent (hyphens → underscores)
```

---

### Pitfall 2: Forgetting to Include Package Data

```toml
# ❌ Non-Python files not included by default!
# Templates, config files, data files get left out

# ✅ Include them explicitly in pyproject.toml
[tool.setuptools.package-data]
pearson_billing = ["py.typed", "templates/*.html", "data/*.json"]
```

---

### Pitfall 3: Importing During Build

```python
# ❌ setup.py/pyproject.toml tries to import the package
# But dependencies aren't installed yet!

# setup.py:
# from pearson_billing import __version__    # ImportError!

# ✅ Keep version as a string in pyproject.toml
# Or use importlib.metadata at runtime:
from importlib.metadata import version
__version__ = version("pearson-billing")
```

---

### Pitfall 4: Missing `__init__.py`

```python
# ❌ Directory without __init__.py is NOT a package!
# src/pearson_billing/utils/helpers.py  ← Can't import!

# ✅ Add __init__.py to every package directory
# src/pearson_billing/__init__.py
# src/pearson_billing/utils/__init__.py    ← Required!
# src/pearson_billing/utils/helpers.py     ← Now importable
```

---

### Pitfall 5: Pinning Dependencies Too Tightly

```toml
# ❌ Over-pinned — breaks other packages
# dependencies = ["pydantic==2.5.3"]

# ✅ Use compatible ranges
# dependencies = ["pydantic>=2.0,<3.0"]
# Or: "pydantic>=2.0"  (minimum only — most flexible)
# Or: "pydantic~=2.5"  (compatible release: >=2.5, <3.0)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📦 Analogy: Packaging = Launching a Product

```
YOUR CODE = THE LEGAL SERVICE
  The actual value you provide.

PYPROJECT.TOML = THE BUSINESS PLAN
  Name, description, what you need (dependencies),
  who you are, how to reach you.

BUILD = MANUFACTURING
  Turn raw code into a distributable product.
  .whl = the finished package, ready to ship.

PYPI = THE MARKETPLACE
  Where 500,000+ packages are listed.
  Anyone can find and install yours.

PIP INSTALL = THE CUSTOMER PURCHASE
  One command. Downloaded, installed, ready to use.

VERSIONING = PRODUCT UPDATES
  1.0.0 → stable release
  1.1.0 → new features
  2.0.0 → breaking changes (new contract)

DONNA'S RULE:
  "A brilliant service nobody can access is worthless.
   Package it. Publish it. Let the world use it.
   And always — ALWAYS — test before you ship."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Project Skeleton**
```
Create a complete project structure:
  my-package/
  ├── pyproject.toml
  ├── README.md
  ├── LICENSE
  ├── src/my_package/__init__.py
  └── tests/test_basic.py
Install in editable mode. Import and verify.
```

**Exercise 2: Version and Metadata**
```
Fill out pyproject.toml with:
  - Name, version, description, author
  - Python version requirement
  - Two dependencies
  - Two optional dev dependencies
  - A classifier list
```

**Exercise 3: Write a README**
```
Write a README.md with:
  - Title, description
  - Installation command
  - Quick start code example
  - Features list
  - License
```

---

### 🟡 Intermediate Exercises

**Exercise 4: CLI Entry Point**
```
Add a CLI command to your package:
  [project.scripts]
  mycli = "my_package.cli:main"
After install, 'mycli --help' should work.
```

**Exercise 5: Build and Inspect**
```
Build your package with 'python -m build'.
List the contents of the .whl file (it's a zip).
Verify all expected files are included.
```

**Exercise 6: TestPyPI Dry Run**
```
Upload your package to TestPyPI.
Install it from TestPyPI in a fresh venv.
Verify imports and CLI work correctly.
```

---

### 🔴 Advanced Exercises

**Exercise 7: CI/CD Pipeline**
```
Create GitHub Actions workflows:
  - CI: test on every push (Python 3.10-3.13)
  - Publish: auto-publish to PyPI on release tag
  - Use trusted publishing (no API tokens)
```

**Exercise 8: Plugin System**
```
Build a package that supports plugins via entry points:
  [project.entry-points."my_package.plugins"]
  csv = "my_plugins.csv:CSVPlugin"
Discover and load plugins at runtime.
```

**Exercise 9: Monorepo with Multiple Packages**
```
Create a monorepo with 2 related packages:
  packages/core/pyproject.toml
  packages/cli/pyproject.toml (depends on core)
Build and publish both. Verify dependency resolution.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```toml
# Bug 1: Wrong build system
[build-system]
requires = ["setuptools"]
# Missing build-backend!

# Bug 2: Package not found after install
[tool.setuptools.packages.find]
where = ["lib"]    # Code is in src/, not lib/!

# Bug 3: Missing dependency
[project]
dependencies = []
# But code does: import pydantic  → ImportError for users!
```

```python
# Bug 4: Relative import fails after install
# src/pearson_billing/reports.py
from .calculator import Calculator    # Works in dev
from calculator import Calculator     # ❌ Fails when installed!

# Bug 5: __init__.py doesn't export anything
# __init__.py is empty
# Users can't: from pearson_billing import Calculator
```

### Fixed Code ✅

```toml
# Fix 1: Add build-backend
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

# Fix 2: Correct source directory
[tool.setuptools.packages.find]
where = ["src"]

# Fix 3: Declare ALL dependencies
[project]
dependencies = ["pydantic>=2.0"]
```

```python
# Fix 4: Use relative imports consistently
from .calculator import Calculator    # ✅ Always relative within package

# Fix 5: Export public API in __init__.py
from pearson_billing.calculator import Calculator
from pearson_billing.reports import ReportGenerator
__all__ = ["Calculator", "ReportGenerator"]
```

---

## 8. 🌍 Real-World Application

### Popular Packages and Their Structure

| Package | Structure | Notable |
|---------|-----------|---------|
| **requests** | Flat layout | Simple, clean API |
| **Flask** | src layout | Entry points for CLI |
| **pydantic** | src layout | Extensive pyproject.toml |
| **rich** | src layout | py.typed for type hints |
| **black** | src layout | Console scripts entry point |
| **pytest** | src layout | Plugin entry points |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is `pyproject.toml`?

**Q2.** What is the difference between a source distribution and a wheel?

**Q3.** What is PyPI?

**Q4.** What does `pip install -e .` do?

**Q5.** What is semantic versioning?

**Q6.** Why use the `src/` layout?

**Q7.** What does `twine` do?

**Q8.** What is an entry point in packaging?

**Q9.** How do you include non-Python files in a package?

**Q10.** What should you always do before publishing?

---

### 📋 Answer Key

> **Q1.** The unified configuration file for Python projects. Defines build system, project metadata, dependencies, and tool settings — replacing `setup.py`, `setup.cfg`, etc.

> **Q2.** Source distribution (`.tar.gz`) = raw source requiring build steps. Wheel (`.whl`) = pre-built, installs immediately without compilation.

> **Q3.** The Python Package Index — the official repository where 500,000+ packages are hosted. `pip install` downloads from PyPI by default.

> **Q4.** Installs the package in "editable" (development) mode. Changes to source files take effect immediately without reinstalling.

> **Q5.** MAJOR.MINOR.PATCH versioning: MAJOR = breaking changes, MINOR = new features (backward-compatible), PATCH = bug fixes.

> **Q6.** Prevents accidentally importing the local source during testing (forces install-then-test), which catches packaging bugs early.

> **Q7.** Uploads built packages to PyPI (or TestPyPI). Validates package metadata and handles authentication.

> **Q8.** A mapping from a name to a Python function, defined in `pyproject.toml`. Used for CLI commands (`[project.scripts]`) and plugin systems.

> **Q9.** Declare them in `pyproject.toml` under `[tool.setuptools.package-data]` with glob patterns.

> **Q10.** Test! Run your test suite, build the package, install it in a clean virtual environment, verify imports and CLI work, and upload to TestPyPI before real PyPI.

---

## 🎬 End of Lesson Scene

**Harvey:** "So anyone in the world can now use our billing library?"

**Mike:** "`pip install pearson-billing`. One command. Any machine. Any platform. Versioned, tested, documented."

**Harvey:** "From variables and integers in Level 1 to publishing a professional library to the world. Five levels. Sixty lessons. From zero to production."

**Mike:** "And every concept built on the last. No shortcuts, no gaps."

**Harvey:** *leans back* "That's how you practice law. And that's how you learn Python."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Modern project structure (src layout) | ✅ |
| 2 | `pyproject.toml` (metadata, deps, tools) | ✅ |
| 3 | Package code and `__init__.py` exports | ✅ |
| 4 | Writing a good README | ✅ |
| 5 | Writing and running tests | ✅ |
| 6 | Building with `python -m build` | ✅ |
| 7 | Publishing to PyPI with `twine` | ✅ |
| 8 | Semantic versioning | ✅ |
| 9 | CLI entry points | ✅ |
| 10 | CI/CD with GitHub Actions | ✅ |

---

> 🎉 **CURRICULUM COMPLETE!**
>
> **60 lessons across 5 levels.** From `x = 42` to `pip install your-library`.
>
> Level 1: Learning — Variables, types, control flow, functions, classes
> Level 2: Growing — Data structures, comprehensions, decorators, file I/O
> Level 3: Building — Generators, OOP, testing, logging, packaging basics
> Level 4: Advancing — Decorators deep-dive, inheritance, regex, ORMs, CPython
> Level 5: Innovating — Sockets, metaclasses, async, design patterns, PyPI
