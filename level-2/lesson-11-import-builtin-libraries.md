# 🏛️ Level 2 – Applying | Lesson 11: Import Python Built-In Libraries

## *"The Arsenal"*

> **O'Reilly Skill Objective:** *Import and use Python's standard library modules effectively.*

> **Previously Learned:** Variables, data types, functions, classes, objects, file I/O, JSON, dataclasses, comprehensions, generators, built-in functions

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

*Thursday. Harvey drops a thick folder on Mike's desk.*

**Harvey:** "We need to process 50 case files by end of day. Each one needs: a unique case ID, today's date, the time elapsed since filing, sorted by priority, formatted as currency, logged with timestamps, and archived in a structured folder."

**Mike:** "So I need: unique IDs, date math, sorting, string formatting, logging, and file system operations."

**Harvey:** "You don't have to build any of that from scratch."

**Mike:** "The standard library. Python ships with 200+ built-in modules. Random numbers, dates, file paths, math, regular expressions, HTTP requests, email sending — all pre-built, tested, and ready."

**Harvey:** "The best lawyers don't write their own contracts from scratch. They use templates and precedent. The best programmers don't reinvent the wheel. They `import` it."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Python's standard library is organized into three tiers."

| Tier | What | Import Needed? | Example |
|------|------|:-:|---------|
| **Built-in** | Always available | ❌ No | `print()`, `len()`, `range()`, `type()` |
| **Standard Library** | Ships with Python | ✅ `import` | `os`, `json`, `datetime`, `math` |
| **Third-Party** | Installed separately | ✅ `pip install` + `import` | `requests`, `flask`, `pandas` |

**Harvey:** "This lesson covers tier 2 — the standard library. You've already used `json` and `csv`. Now you'll learn the essential 15 modules that cover 90% of real-world needs."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Import Syntax — Five Ways ⭐

```python
# Method 1: Import the module
import os
os.path.exists("file.txt")         # Must prefix with module name

# Method 2: Import specific items
from os.path import exists, join
exists("file.txt")                  # Direct name — no prefix

# Method 3: Import with alias
import datetime as dt
now = dt.datetime.now()

# Method 4: Import everything (❌ avoid in production!)
from math import *                  # Imports ALL names — pollutes namespace
# Which sqrt? math.sqrt or numpy.sqrt?

# Method 5: Import a submodule
from collections import defaultdict, Counter
```

**Mike:** "Best practices:"

| ✅ Do | ❌ Don't |
|-------|---------|
| `import os` | `from os import *` |
| `from datetime import datetime` | `from datetime import *` |
| `import numpy as np` (conventions) | `import numpy as NUMPY` |
| Group imports at top of file | Scatter imports throughout code |
| Stdlib → third-party → local | Mix import sources randomly |

---

### 3.2 `datetime` — Dates and Times ⭐⭐

```python
from datetime import datetime, date, time, timedelta

# Current date and time
now = datetime.now()
print(f"Now: {now}")                    # 2026-03-14 08:44:55.123456
print(f"Date: {now.date()}")           # 2026-03-14
print(f"Time: {now.time()}")           # 08:44:55.123456
print(f"Year: {now.year}, Month: {now.month}, Day: {now.day}")

# Create specific dates
filing_date = datetime(2025, 6, 15, 9, 30)
print(filing_date)                      # 2025-06-15 09:30:00

# Date arithmetic with timedelta
deadline = datetime.now() + timedelta(days=30)
print(f"Deadline: {deadline.date()}")

elapsed = datetime.now() - filing_date
print(f"Days since filing: {elapsed.days}")
print(f"Total seconds: {elapsed.total_seconds():,.0f}")

# Formatting (strftime = string format time)
print(now.strftime("%Y-%m-%d"))           # 2026-03-14
print(now.strftime("%B %d, %Y"))          # March 14, 2026
print(now.strftime("%I:%M %p"))           # 08:44 AM
print(now.strftime("%A, %B %d, %Y"))      # Thursday, March 14, 2026

# Parsing (strptime = string parse time)
date_str = "2025-06-15"
parsed = datetime.strptime(date_str, "%Y-%m-%d")
print(parsed)                              # 2025-06-15 00:00:00

# Common format codes:
# %Y=4-digit year, %m=month(01-12), %d=day(01-31)
# %H=hour(00-23), %I=hour(01-12), %M=minute, %S=second
# %A=weekday name, %B=month name, %p=AM/PM
```

---

### 3.3 `os` and `pathlib` — File System ⭐

```python
import os
from pathlib import Path

# --- os module (traditional) ---
print(os.getcwd())                    # Current directory
print(os.listdir("."))                 # List directory contents

os.makedirs("archives/2025", exist_ok=True)   # Create nested dirs

# Check existence
print(os.path.exists("clients.json"))
print(os.path.isfile("clients.json"))
print(os.path.isdir("archives"))

# Path manipulation
full_path = os.path.join("archives", "2025", "case_001.json")
print(os.path.basename(full_path))     # case_001.json
print(os.path.dirname(full_path))      # archives/2025
print(os.path.splitext(full_path))     # ('archives/2025/case_001', '.json')

# Environment variables
home = os.environ.get("HOME", os.environ.get("USERPROFILE"))

# --- pathlib (modern, object-oriented) ---
p = Path("archives") / "2025" / "case_001.json"
print(p.name)           # case_001.json
print(p.stem)           # case_001
print(p.suffix)         # .json
print(p.parent)         # archives/2025
print(p.exists())       # True/False

# Iterate directory
for file in Path(".").glob("*.json"):
    print(f"  Found: {file}")

# Recursive glob
for py_file in Path(".").rglob("*.py"):
    print(f"  Python file: {py_file}")

# Read/write (Python 3.5+)
p = Path("test.txt")
p.write_text("Hello from pathlib!")
content = p.read_text()
p.unlink()  # Delete
```

---

### 3.4 `random` — Random Numbers

```python
import random

# Random float [0.0, 1.0)
print(random.random())                  # 0.7349...

# Random integer [a, b] (inclusive)
case_number = random.randint(1000, 9999)
print(f"CASE-{case_number}")

# Random choice from a sequence
attorneys = ["Harvey", "Mike", "Louis", "Rachel"]
assigned = random.choice(attorneys)

# Random sample (without replacement)
team = random.sample(attorneys, 2)
print(f"Team: {team}")

# Shuffle a list IN PLACE
random.shuffle(attorneys)

# Weighted random
roles = ["Partner", "Associate", "Paralegal"]
weights = [2, 5, 3]
selected = random.choices(roles, weights=weights, k=5)    # k items with replacement

# Reproducible results
random.seed(42)
print(random.random())   # Always 0.6394... with seed 42
```

---

### 3.5 `collections` — Specialized Containers

```python
from collections import Counter, defaultdict, namedtuple, deque, OrderedDict

# Counter — count occurrences
case_types = ["Merger", "Patent", "Merger", "Litigation", "Patent", "Merger"]
counts = Counter(case_types)
print(counts)                          # Counter({'Merger': 3, 'Patent': 2, ...})
print(counts.most_common(2))           # [('Merger', 3), ('Patent', 2)]

# defaultdict — dict with automatic defaults
by_attorney = defaultdict(list)
assignments = [("Harvey", "Wayne"), ("Mike", "Stark"), ("Harvey", "Dalton")]
for atty, client in assignments:
    by_attorney[atty].append(client)   # No KeyError!

# namedtuple — lightweight data object
Client = namedtuple("Client", ["name", "revenue", "type"])
c = Client("Wayne", 2500000, "Merger")
print(c.name)     # Wayne (access by name)
print(c[0])       # Wayne (access by index)
print(c._asdict()) # {'name': 'Wayne', 'revenue': 2500000, 'type': 'Merger'}

# deque — efficient double-ended queue
recent = deque(maxlen=5)
for i in range(10):
    recent.append(f"event_{i}")
print(list(recent))   # Last 5 events only
```

---

### 3.6 `math` — Mathematical Functions

```python
import math

print(math.pi)              # 3.141592653589793
print(math.e)               # 2.718281828459045
print(math.sqrt(144))       # 12.0
print(math.ceil(4.2))       # 5 (round up)
print(math.floor(4.8))      # 4 (round down)
print(math.log(100, 10))    # 2.0 (log base 10)
print(math.log2(1024))      # 10.0
print(math.factorial(5))    # 120
print(math.gcd(48, 18))     # 6
print(math.isclose(0.1 + 0.2, 0.3))   # True (float comparison!)
print(math.comb(10, 3))     # 120 (combinations)
print(math.perm(10, 3))     # 720 (permutations)
```

---

### 3.7 `re` — Regular Expressions

```python
import re

text = "Contact Harvey at harvey@psl.com or Mike at mike@psl.com"

# Find all matches
emails = re.findall(r'\b[\w.]+@[\w.]+\.\w+\b', text)
print(emails)   # ['harvey@psl.com', 'mike@psl.com']

# Search for first match
match = re.search(r'(\w+)@(\w+)\.(\w+)', text)
if match:
    print(match.group(0))    # harvey@psl.com (full match)
    print(match.group(1))    # harvey (first group)
    print(match.group(2))    # psl (second group)

# Replace
cleaned = re.sub(r'\b\d{3}-\d{4}\b', '[REDACTED]', "Call 555-1234 or 555-5678")
print(cleaned)   # "Call [REDACTED] or [REDACTED]"

# Split
parts = re.split(r'[,;\s]+', "Harvey, Mike; Louis Rachel")
print(parts)     # ['Harvey', 'Mike', 'Louis', 'Rachel']

# Compile for reuse (faster when used repeatedly)
email_pattern = re.compile(r'\b[\w.]+@[\w.]+\.\w+\b')
matches = email_pattern.findall(text)
```

---

### 3.8 `sys` — System Info and Configuration

```python
import sys

print(sys.version)                   # Python version
print(sys.platform)                  # 'win32', 'linux', 'darwin'
print(sys.path)                      # Module search paths
print(sys.argv)                      # Command-line arguments
print(sys.getsizeof([1, 2, 3]))     # Memory size in bytes

# Exit the program
# sys.exit(0)    # 0 = success, non-zero = error
```

---

### 3.9 `itertools` — Advanced Iteration

```python
from itertools import chain, islice, cycle, repeat, product, combinations, permutations, groupby

# chain — combine multiple iterables
all_names = list(chain(["Harvey", "Mike"], ["Louis"], ["Jessica"]))
print(all_names)   # ['Harvey', 'Mike', 'Louis', 'Jessica']

# islice — slice an iterator (works with generators!)
from itertools import count
first_10_evens = list(islice((x for x in count() if x % 2 == 0), 10))
print(first_10_evens)   # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# combinations — all ways to choose k items
pairs = list(combinations(["Harvey", "Mike", "Louis"], 2))
print(pairs)
# [('Harvey', 'Mike'), ('Harvey', 'Louis'), ('Mike', 'Louis')]

# product — Cartesian product
assignments = list(product(["Harvey", "Mike"], ["Wayne", "Stark"]))
print(assignments)
# [('Harvey', 'Wayne'), ('Harvey', 'Stark'), ('Mike', 'Wayne'), ('Mike', 'Stark')]

# groupby — group consecutive items (must be sorted first!)
data = [("Legal", "Harvey"), ("Legal", "Mike"), ("IT", "David"), ("IT", "Sarah")]
for dept, members in groupby(data, key=lambda x: x[0]):
    print(f"  {dept}: {[m[1] for m in members]}")
```

---

### 3.10 `functools` — Higher-Order Function Tools

```python
from functools import lru_cache, partial, reduce

# lru_cache — memoization (cache results)
@lru_cache(maxsize=128)
def expensive_calculation(n):
    """Results are cached — subsequent calls with same n are instant."""
    print(f"  Computing {n}...")
    return sum(i ** 2 for i in range(n))

print(expensive_calculation(1000))    # Computes (slow)
print(expensive_calculation(1000))    # Returns cached result (instant!)
print(expensive_calculation.cache_info())  # hits, misses, size

# partial — pre-fill function arguments
def format_currency(amount, symbol="$", decimals=2):
    return f"{symbol}{amount:,.{decimals}f}"

format_usd = partial(format_currency, symbol="$")
format_eur = partial(format_currency, symbol="€")
format_round = partial(format_currency, decimals=0)

print(format_usd(2500000))    # $2,500,000.00
print(format_eur(800000))      # €800,000.00
print(format_round(600000))    # $600,000

# reduce — accumulate with a function
total = reduce(lambda a, b: a + b, [2500000, 800000, 600000])
print(total)   # 3900000
```

---

### 3.11 Other Essential Modules — Quick Reference

```python
# --- string ---
import string
print(string.ascii_uppercase)    # ABCDEFGHIJKLMNOPQRSTUVWXYZ
print(string.digits)             # 0123456789

# --- uuid ---
import uuid
unique_id = uuid.uuid4()
print(f"CASE-{unique_id}")       # CASE-a1b2c3d4-e5f6-...

# --- hashlib ---
import hashlib
hash_val = hashlib.sha256("Wayne".encode()).hexdigest()
print(hash_val[:16])              # First 16 chars of SHA-256

# --- time ---
import time
start = time.time()
time.sleep(0.1)
elapsed = time.time() - start
print(f"Elapsed: {elapsed:.3f}s")

# --- typing ---
from typing import List, Dict, Optional, Tuple
def get_clients(active: bool = True) -> List[Dict[str, str]]:
    ...

# --- textwrap ---
import textwrap
wrapped = textwrap.fill("A very long legal brief...", width=40)
dedented = textwrap.dedent("""
    Multi-line
    indented
    string
""")

# --- pprint ---
from pprint import pprint
complex_data = {"attorneys": [{"name": "Harvey", "cases": ["Wayne", "Stark"]}]}
pprint(complex_data, width=40)

# --- shutil ---
import shutil
# shutil.copy("src.txt", "dst.txt")          # Copy file
# shutil.copytree("src_dir", "dst_dir")      # Copy directory tree
# shutil.rmtree("dir_to_delete")             # Delete directory tree
```

---

### 3.12 Solving Harvey's Problem — Case Processing System

```python
# ============================================
# Pearson Specter Litt — Case Processing System
# Powered by the Standard Library
# ============================================

import uuid
import os
from datetime import datetime, timedelta
from pathlib import Path
from collections import Counter, defaultdict
import json
import random
import math

class CaseProcessor:
    """Processes case files using the standard library."""
    
    def __init__(self, base_dir="case_archive"):
        self.base_dir = Path(base_dir)
        self.cases = []
        self.log = []
    
    def generate_case_id(self):
        """uuid module — unique IDs"""
        return f"PSL-{uuid.uuid4().hex[:8].upper()}"
    
    def create_case(self, client, case_type, priority=3, filing_date=None):
        """datetime module — date handling"""
        case = {
            "id": self.generate_case_id(),
            "client": client,
            "type": case_type,
            "priority": priority,
            "filed": (filing_date or datetime.now()).isoformat(),
            "deadline": (datetime.now() + timedelta(days=90)).isoformat(),
            "status": "open",
        }
        self.cases.append(case)
        self._log(f"Created case {case['id']} for {client}")
        return case
    
    def get_stats(self):
        """collections module — counting and grouping"""
        type_counts = Counter(c["type"] for c in self.cases)
        by_priority = defaultdict(list)
        for c in self.cases:
            by_priority[c["priority"]].append(c["client"])
        return {"type_counts": type_counts, "by_priority": dict(by_priority)}
    
    def sort_cases(self, key="priority", reverse=False):
        """sorted() with custom keys"""
        return sorted(self.cases, key=lambda c: c[key], reverse=reverse)
    
    def archive(self):
        """os/pathlib + json — file system + serialization"""
        archive_dir = self.base_dir / datetime.now().strftime("%Y-%m")
        archive_dir.mkdir(parents=True, exist_ok=True)
        
        filepath = archive_dir / "cases.json"
        with open(filepath, "w", encoding="utf-8") as f:
            json.dump(self.cases, f, indent=2)
        
        self._log(f"Archived {len(self.cases)} cases to {filepath}")
        return filepath
    
    def calculate_metrics(self):
        """math module — analytics"""
        if not self.cases:
            return {}
        
        priorities = [c["priority"] for c in self.cases]
        return {
            "total": len(self.cases),
            "avg_priority": sum(priorities) / len(priorities),
            "variance": sum((p - sum(priorities)/len(priorities))**2 
                           for p in priorities) / len(priorities),
            "combinations_of_2": math.comb(len(self.cases), 2),
        }
    
    def _log(self, message):
        entry = f"[{datetime.now():%Y-%m-%d %H:%M:%S}] {message}"
        self.log.append(entry)
    
    def dashboard(self):
        """Display the processing dashboard."""
        print(f"\n{'='*60}")
        print(f"{'🏛️  CASE PROCESSING DASHBOARD':^60}")
        print(f"{'='*60}")
        
        stats = self.get_stats()
        metrics = self.calculate_metrics()
        
        print(f"\n📊 OVERVIEW")
        print(f"  Total Cases: {metrics.get('total', 0)}")
        print(f"  Avg Priority: {metrics.get('avg_priority', 0):.1f}")
        
        print(f"\n💼 BY TYPE")
        for case_type, count in stats["type_counts"].most_common():
            bar = "█" * (count * 4) + "░" * ((5 - count) * 4)
            print(f"  {case_type:<15} [{bar}] {count}")
        
        print(f"\n📋 SORTED BY PRIORITY")
        for case in self.sort_cases("priority"):
            days_left = (datetime.fromisoformat(case["deadline"]) - datetime.now()).days
            print(f"  P{case['priority']} | {case['id']} | {case['client']:<20} "
                  f"| {case['type']:<12} | {days_left}d left")
        
        print(f"\n📝 RECENT LOG")
        for entry in self.log[-5:]:
            print(f"  {entry}")
        
        print(f"\n{'='*60}")

# === Run it ===
processor = CaseProcessor()

# Create cases
processor.create_case("Wayne Enterprises", "Merger", priority=1)
processor.create_case("Stark Industries", "Patent", priority=2)
processor.create_case("Oscorp", "Litigation", priority=1)
processor.create_case("Dalton Industries", "Merger", priority=3)
processor.create_case("Queen Industries", "Merger", priority=2)

# Archive to disk
archive_path = processor.archive()

# Show dashboard
processor.dashboard()

# Cleanup
import shutil
if processor.base_dir.exists():
    shutil.rmtree(processor.base_dir)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `from module import *` — Namespace Pollution

```python
# ❌ Which sqrt? Which log?
from math import *
from cmath import *

# Now there are TWO sqrt, TWO log — the second overwrites the first!
# Bugs are silent and confusing.

# ✅ Import specifically
from math import sqrt, log
from cmath import sqrt as complex_sqrt
```

---

### Pitfall 2: Circular Imports

```python
# file: models.py
from utils import format_name    # Imports utils
class Client:
    ...

# file: utils.py  
from models import Client        # Imports models → which imports utils → loop!
def format_name(client: Client):
    ...

# ✅ Fix: restructure or use lazy imports
# Move shared code to a third module, or import inside the function:
def format_name(client):
    from models import Client     # Import at point of use
    ...
```

---

### Pitfall 3: `datetime` Module vs `datetime` Class

```python
# The module and the class have the SAME NAME!
import datetime
now = datetime.datetime.now()    # module.class.method — verbose!

# ✅ Import the class directly
from datetime import datetime
now = datetime.now()             # class.method — clean!

# ⚠️ But now you can't access datetime.timedelta:
# timedelta = datetime.timedelta   # AttributeError!
# ✅ Import both:
from datetime import datetime, timedelta
```

---

### Pitfall 4: Mutable Default with Datetime

```python
from datetime import datetime

# ❌ Default is evaluated at DEFINITION time, not call time!
def create_event(name, timestamp=datetime.now()):
    return {"name": name, "time": timestamp}

e1 = create_event("Event 1")
import time; time.sleep(1)
e2 = create_event("Event 2")
print(e1["time"] == e2["time"])   # True! Same timestamp!

# ✅ Fix: use None sentinel
def create_event(name, timestamp=None):
    if timestamp is None:
        timestamp = datetime.now()
    return {"name": name, "time": timestamp}
```

---

### Pitfall 5: `os.path` vs `pathlib` — Pick One

```python
# ❌ Mixing styles is confusing
import os
from pathlib import Path

path_str = os.path.join("data", "clients.json")
path_obj = Path("data") / "clients.json"

# They're the same path but different types!
print(type(path_str))    # <class 'str'>
print(type(path_obj))    # <class 'PosixPath'> or <class 'WindowsPath'>

# ✅ Pick one — pathlib is the modern choice for new code
# Use os.path only for legacy compatibility
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🧰 Analogy: Standard Library = The Firm's Supply Room

```
BUILT-INS (always on your desk):
  📎 Paperclips (print, len, range, type)
  ✂️ Scissors (str, int, list, dict)
  📐 Ruler (max, min, sum, sorted)

STANDARD LIBRARY (in the supply room — just ask):
  📅 Calendar (datetime)
  📁 Filing cabinet (os, pathlib)
  🎲 Dice (random)
  📊 Calculator (math)
  🔍 Magnifying glass (re)
  📦 Organizers (collections)
  ⏱️ Stopwatch (time)
  🗂️ Sorters (itertools)
  🧠 Brain boosters (functools)

THIRD-PARTY (order from outside — pip install):
  📡 Radio (requests)
  🌐 Website builder (flask, django)
  📈 Charts (matplotlib)
  🤖 AI tools (numpy, pandas)
```

**Donna:** "You wouldn't order a custom-built calculator when there's one in the supply room. Check the standard library before writing code from scratch."

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Date Calculator**
```
Using datetime:
a) Print today's date in "March 14, 2026" format
b) Calculate days until New Year's Day 2027
c) Find what day of the week you were born
d) Add 90 business days to today (skip weekends)
```

**Exercise 2: File Explorer**
```
Using pathlib:
a) List all files in the current directory
b) Find all .py files recursively
c) Get total size of all files in a directory
d) Create a nested directory structure
```

**Exercise 3: Random Data Generator**
```
Using random:
a) Generate 20 random case IDs (format: PSL-XXXX)
b) Randomly assign 10 cases to 4 attorneys
c) Generate mock revenue data (100K - 5M range)
d) Create a shuffled deck of cards
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Text Analyzer**
```
Using re, collections, string:
a) Count word frequency in a paragraph
b) Extract all email addresses
c) Validate phone numbers (multiple formats)
d) Find the most common 3-letter words
```

**Exercise 5: Performance Profiler**
```
Using time, functools:
a) Create a @timer decorator using time.time()
b) Create a @memoize decorator using lru_cache
c) Compare fibonacci with and without memoization
d) Profile 5 different sorting approaches
```

**Exercise 6: Data Pipeline**
```
Using multiple modules (datetime, json, pathlib, collections, uuid):
Build a system that:
a) Generates 100 fake records (uuid, datetime, random)
b) Saves to JSON (json, pathlib)
c) Loads and analyzes (collections.Counter, statistics)
d) Archives old records (shutil, pathlib)
```

---

### 🔴 Advanced Exercises

**Exercise 7: CLI Application**
```
Using argparse, sys, pathlib:
Build a command-line file manager:
  python filemgr.py list --dir ./data --ext .json
  python filemgr.py stats --dir ./data
  python filemgr.py search --pattern "*.py" --dir ./
  python filemgr.py clean --dir ./tmp --older-than 7
```

**Exercise 8: Log Analyzer**
```
Using re, datetime, collections, pathlib:
Parse log files and produce:
a) Error rate by hour
b) Most common error messages
c) Average response times
d) Anomaly detection (response time > 2 std deviations)
```

**Exercise 9: Module Explorer**
```
Write a script that inspects any module:
  explore("collections")
Output:
  - All classes and functions
  - Docstrings for each
  - Example usage (if available)
  - Source file location

Use: dir(), help(), inspect module
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Wrong import
import datetime
now = datetime.now()   # AttributeError!

# Bug 2: os.path.join with wrong separators
path = os.path.join("data", "file.txt")
path = "data" + "/" + "file.txt"  # ❌ Not cross-platform!

# Bug 3: random without seed (non-reproducible)
test_data = [random.randint(1, 100) for _ in range(10)]
# Different every run — tests fail randomly!

# Bug 4: re.match vs re.search
text = "Contact: harvey@psl.com"
match = re.match(r'\w+@\w+\.\w+', text)   # None!
# match only checks from the START of the string

# Bug 5: groupby without sorting
data = [("Legal", "Harvey"), ("IT", "David"), ("Legal", "Mike")]
for dept, members in groupby(data, key=lambda x: x[0]):
    print(f"{dept}: {list(members)}")
# Legal appears TWICE — not grouped properly!
```

### Fixed Code ✅

```python
# Fix 1: Import the class from the module
from datetime import datetime
now = datetime.now()   # ✅

# Fix 2: Use os.path.join or pathlib for cross-platform
path = os.path.join("data", "file.txt")    # ✅
path = Path("data") / "file.txt"           # ✅ Even better

# Fix 3: Set seed for reproducible tests
random.seed(42)
test_data = [random.randint(1, 100) for _ in range(10)]  # ✅ Same every run

# Fix 4: Use re.search for anywhere in string
text = "Contact: harvey@psl.com"
match = re.search(r'\w+@\w+\.\w+', text)  # ✅ Finds it!

# Fix 5: Sort FIRST, then groupby
data = [("Legal", "Harvey"), ("IT", "David"), ("Legal", "Mike")]
data_sorted = sorted(data, key=lambda x: x[0])   # ✅ Sort first
for dept, members in groupby(data_sorted, key=lambda x: x[0]):
    print(f"{dept}: {[m[1] for m in members]}")
# IT: ['David']
# Legal: ['Harvey', 'Mike']  ✅
```

---

## 8. 🌍 Real-World Application

### The Essential 15 Standard Library Modules

| Module | What For | Used In |
|--------|----------|---------|
| `datetime` | Dates, times, durations | Logging, scheduling, timestamps |
| `os` / `pathlib` | File system operations | File management, config |
| `json` | Data serialization | APIs, config files, data exchange |
| `csv` | Tabular data | Reports, data import/export |
| `re` | Pattern matching | Validation, parsing, search |
| `collections` | Specialized containers | Counting, grouping, queues |
| `math` | Mathematical operations | Analytics, calculations |
| `random` | Random generation | Testing, simulation, IDs |
| `sys` | System interaction | CLI args, platform info |
| `itertools` | Advanced iteration | Data processing, combinations |
| `functools` | Function tools | Caching, decorators |
| `uuid` | Unique identifiers | Database IDs, reference numbers |
| `hashlib` | Hashing | Checksums, integrity |
| `typing` | Type hints | Code documentation, IDE support |
| `dataclasses` | Data containers | Models, DTOs, configs |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between `import os` and `from os import path`?

**Q2.** Why is `from module import *` discouraged?

**Q3.** How do you format a `datetime` object as "March 14, 2026"?

**Q4.** What is the difference between `os.path` and `pathlib`?

**Q5.** What does `@lru_cache` do and when would you use it?

**Q6.** What is the difference between `re.match()` and `re.search()`?

**Q7.** How does `collections.Counter` work?

**Q8.** Why must data be sorted before using `itertools.groupby()`?

**Q9.** How do you make `random` results reproducible?

**Q10.** Name 5 standard library modules and what each does.

---

### 📋 Answer Key

> **Q1.** `import os` imports the module (use as `os.path`). `from os import path` imports just `path` (use directly).

> **Q2.** Namespace pollution — you don't know what names were imported, and they can overwrite existing names.

> **Q3.** `datetime.now().strftime("%B %d, %Y")`

> **Q4.** `os.path` uses strings, `pathlib` uses `Path` objects with operator overloading (`/`). `pathlib` is the modern approach.

> **Q5.** Caches function results — subsequent calls with the same arguments return the cached value instantly. Use for expensive, deterministic computations.

> **Q6.** `match()` checks from the start of the string. `search()` checks anywhere in the string.

> **Q7.** A dict subclass that counts occurrences: `Counter(["a","b","a"])` → `{'a': 2, 'b': 1}`.

> **Q8.** `groupby` only groups **consecutive** elements. Unsorted data creates multiple groups for the same key.

> **Q9.** `random.seed(42)` — same seed produces the same sequence every time.

> **Q10.** `datetime` (dates/times), `os`/`pathlib` (file system), `json` (serialization), `re` (regex), `collections` (containers).

---

## 🎬 End of Lesson Scene

**Harvey:** "50 case files processed. IDs generated, dates calculated, priorities sorted, archived to disk."

**Mike:** "And we didn't write a single utility from scratch. `uuid` for IDs, `datetime` for dates, `sorted` with `collections` for priorities, `pathlib` and `json` for archiving."

**Harvey:** "Next: virtual environments and pip. Installing third-party libraries without breaking your system."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Import syntax (5 methods) | ✅ |
| 2 | `datetime` — dates, times, formatting, parsing, arithmetic | ✅ |
| 3 | `os` / `pathlib` — file system operations | ✅ |
| 4 | `random` — random generation | ✅ |
| 5 | `collections` — Counter, defaultdict, namedtuple, deque | ✅ |
| 6 | `math` — mathematical functions | ✅ |
| 7 | `re` — regular expressions | ✅ |
| 8 | `sys` — system info | ✅ |
| 9 | `itertools` — advanced iteration | ✅ |
| 10 | `functools` — lru_cache, partial, reduce | ✅ |

---

> **Next Lesson →** Level 2, Lesson 12: *Use Virtual Environments and Pip in Python*
