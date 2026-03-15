# 🏛️ Level 2 – Applying | Lesson 3: Use List Comprehension in Python

## *"The One-Liner"*

> **O'Reilly Skill Objective:** *Use list comprehension to create new lists efficiently.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, loops, functions, default arguments, `*args`/`**kwargs`, classes, objects, file I/O

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

*Mike is at his desk with two monitors open — one showing a 12-line loop, the other showing a 1-line equivalent.*

**Mike:** "Check this out."

```python
# 12 lines
billable_clients = []
for client in all_clients:
    if client["revenue"] >= 500000:
        if client["status"] == "active":
            formatted = {
                "name": client["name"].upper(),
                "revenue": client["revenue"],
                "tier": "Gold"
            }
            billable_clients.append(formatted)
```

**Mike:** "Same result:"

```python
# 1 line
billable_clients = [
    {"name": c["name"].upper(), "revenue": c["revenue"], "tier": "Gold"}
    for c in all_clients
    if c["revenue"] >= 500000 and c["status"] == "active"
]
```

**Harvey:** *(walking by)* "Same output. One-tenth the code. That's efficiency."

**Mike:** "List comprehensions. The most Pythonic feature in the language. Once you learn them, you'll never write `append` inside a `for` loop again."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me frame this strategically."

| Without Comprehension | With Comprehension |
|----------------------|-------------------|
| 4-6 lines minimum | 1 line |
| Create empty list → loop → append | Single expression |
| Imperative (step-by-step) | Declarative (describe what you want) |
| Slower (slightly) | Faster (C-optimized internally) |

**Harvey:** "List comprehensions aren't just shorter — they're **faster**. Python internally optimizes them. They run about 20-30% faster than the equivalent `for` loop with `append`."

**Harvey:** "But there's a catch: if the comprehension gets too complex, it becomes unreadable. The rule? If you can't understand it in 5 seconds, use a loop."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Syntax

```python
# FOR LOOP pattern:
result = []
for item in iterable:
    result.append(expression)

# LIST COMPREHENSION equivalent:
result = [expression for item in iterable]
```

```python
# Example: Square each number
numbers = [1, 2, 3, 4, 5]

# Loop
squares = []
for n in numbers:
    squares.append(n ** 2)

# Comprehension
squares = [n ** 2 for n in numbers]
print(squares)   # [1, 4, 9, 16, 25]
```

**Mike:** "Read it left-to-right: **'n squared** — **for n** — **in numbers'**"

---

### 3.2 With Filtering (`if`)

```python
# FOR LOOP with conditional:
result = []
for item in iterable:
    if condition:
        result.append(expression)

# COMPREHENSION with filter:
result = [expression for item in iterable if condition]
```

```python
clients = [
    {"name": "Wayne", "revenue": 2500000, "active": True},
    {"name": "Stark", "revenue": 800000, "active": True},
    {"name": "Oscorp", "revenue": 600000, "active": False},
    {"name": "Luthor", "revenue": 150000, "active": True},
]

# Get names of active high-value clients
vip_names = [c["name"] for c in clients if c["revenue"] >= 500000 and c["active"]]
print(vip_names)   # ['Wayne', 'Stark']

# Get revenues above the average
revenues = [c["revenue"] for c in clients]
avg = sum(revenues) / len(revenues)
above_avg = [r for r in revenues if r > avg]
print(above_avg)   # [2500000]
```

---

### 3.3 With Transformation (`if-else`)

**Mike:** "This is different from filtering! `if-else` goes BEFORE the `for`."

```python
# Filtering (if after for):      keeps or discards items
# Transformation (if-else before for):  transforms ALL items

# Filter — only keep even numbers
evens = [n for n in range(10) if n % 2 == 0]
# [0, 2, 4, 6, 8]

# Transform — label ALL numbers as even or odd
labels = ["even" if n % 2 == 0 else "odd" for n in range(6)]
# ['even', 'odd', 'even', 'odd', 'even', 'odd']
```

```python
# Real-world: Classify all clients
tiers = [
    "Platinum" if c["revenue"] >= 1000000
    else "Gold" if c["revenue"] >= 500000
    else "Silver" if c["revenue"] >= 100000
    else "Bronze"
    for c in clients
]
print(tiers)   # ['Platinum', 'Gold', 'Gold', 'Silver']
```

**Mike:** "Position matters!"

| Pattern | Position | Purpose | Items in result |
|---------|----------|---------|----------------|
| `[expr for x in items if cond]` | `if` AFTER `for` | **Filter** | Fewer items (only matching) |
| `[expr_if if cond else expr_else for x in items]` | `if-else` BEFORE `for` | **Transform** | Same count (all items) |

---

### 3.4 Nested Loops in Comprehensions

```python
# Nested for loop:
pairs = []
for x in [1, 2, 3]:
    for y in ["a", "b"]:
        pairs.append((x, y))

# Comprehension — read left-to-right, same nesting order
pairs = [(x, y) for x in [1, 2, 3] for y in ["a", "b"]]
print(pairs)
# [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b'), (3, 'a'), (3, 'b')]
```

```python
# Flatten a list of lists
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

flat = [num for row in matrix for num in row]
print(flat)   # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Read as: for row in matrix → for num in row → num
```

```python
# Practical: All attorney-client pairings
attorneys = ["Harvey", "Mike", "Louis"]
case_types = ["Merger", "Patent", "Litigation"]

assignments = [
    f"{atty} → {case}"
    for atty in attorneys
    for case in case_types
    if not (atty == "Louis" and case == "Merger")  # Louis doesn't do mergers
]
for a in assignments:
    print(f"  {a}")
```

---

### 3.5 Dictionary Comprehensions ⭐

```python
# Syntax: {key_expr: value_expr for item in iterable}

# Convert list to lookup dictionary
clients = [
    {"name": "Wayne", "revenue": 2500000},
    {"name": "Stark", "revenue": 800000},
    {"name": "Oscorp", "revenue": 600000},
]

# Name → Revenue mapping
revenue_map = {c["name"]: c["revenue"] for c in clients}
print(revenue_map)
# {'Wayne': 2500000, 'Stark': 800000, 'Oscorp': 600000}

# With filtering
big_clients = {c["name"]: c["revenue"] for c in clients if c["revenue"] >= 1000000}
print(big_clients)   # {'Wayne': 2500000}

# Invert a dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)   # {1: 'a', 2: 'b', 3: 'c'}

# Classify into tiers
tier_map = {
    c["name"]: ("Platinum" if c["revenue"] >= 1000000 else "Gold")
    for c in clients
    if c["revenue"] >= 500000
}
print(tier_map)   # {'Wayne': 'Platinum', 'Stark': 'Gold', 'Oscorp': 'Gold'}
```

---

### 3.6 Set Comprehensions

```python
# Syntax: {expr for item in iterable}

# Get unique case types
cases = ["Merger", "Patent", "Merger", "Litigation", "Patent", "Merger"]
unique_types = {case for case in cases}
print(unique_types)   # {'Merger', 'Patent', 'Litigation'}

# Unique first letters
names = ["Harvey", "Mike", "Harvey", "Louis", "Donna", "Mike"]
initials = {name[0] for name in names}
print(initials)   # {'H', 'M', 'L', 'D'}

# With filtering
long_names = {name for name in names if len(name) > 4}
print(long_names)   # {'Harvey', 'Louis', 'Donna'}
```

---

### 3.7 Nested Comprehensions (Matrix Operations)

```python
# Create a matrix
matrix = [[i * 3 + j + 1 for j in range(3)] for i in range(3)]
print(matrix)
# [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# Transpose a matrix (swap rows and columns)
transposed = [[row[i] for row in matrix] for i in range(3)]
print(transposed)
# [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# Multiply each element by 2
doubled = [[cell * 2 for cell in row] for row in matrix]
print(doubled)
# [[2, 4, 6], [8, 10, 12], [14, 16, 18]]
```

---

### 3.8 Comprehensions with Functions

```python
# Using functions in expressions
names = ["wayne enterprises", "stark industries", "oscorp"]
formatted = [name.title() for name in names]
print(formatted)   # ['Wayne Enterprises', 'Stark Industries', 'Oscorp']

# With custom functions
def classify_revenue(rev):
    if rev >= 1000000: return "💎"
    elif rev >= 500000: return "🥇"
    elif rev >= 100000: return "🥈"
    return "🥉"

clients = [("Wayne", 2500000), ("Stark", 800000), ("Luthor", 150000), ("Gillis", 50000)]

report = [f"{classify_revenue(rev)} {name}: ${rev:,.0f}" for name, rev in clients]
for line in report:
    print(f"  {line}")
# 💎 Wayne: $2,500,000
# 🥇 Stark: $800,000
# 🥈 Luthor: $150,000
# 🥉 Gillis: $50,000
```

---

### 3.9 Walrus Operator in Comprehensions (Python 3.8+) ⭐

```python
# Without walrus — calculates expensive_operation twice
results = [
    (x, x**2 + x**3)
    for x in range(10)
    if x**2 + x**3 > 100        # Calculated once for filter
]                                 # And again for the expression!

# With walrus operator := — calculate once, reuse
results = [
    (x, y)
    for x in range(10)
    if (y := x**2 + x**3) > 100  # Calculate once, assign to y
]
print(results)   # [(5, 150), (6, 252), (7, 392), (8, 576), (9, 810)]
```

```python
# Practical: process and filter in one pass
import re

raw_data = ["  Wayne: 2500000  ", "  Stark: bad_data  ", "  Oscorp: 600000  "]

# Parse, validate, and collect in one comprehension
valid = [
    (name, int(revenue))
    for line in raw_data
    if (match := re.match(r"\s*(\w+):\s*(\d+)\s*", line))
    for name, revenue in [match.groups()]
]
print(valid)   # [('Wayne', 2500000), ('Oscorp', 600000)]
```

---

### 3.10 Comprehension vs Loop — Readability Guidelines

**Mike:** "The decision framework:"

| Scenario | Use | Why |
|----------|-----|-----|
| Simple transform | Comprehension | `[x*2 for x in items]` — clear |
| Simple filter | Comprehension | `[x for x in items if x > 0]` — clear |
| Filter + transform | Comprehension | `[x.upper() for x in items if x]` — still clear |
| Nested loops (2 deep) | Maybe comprehension | Depends on complexity |
| Nested loops (3+ deep) | **Loop** | Too hard to read |
| Side effects (print, append to external) | **Loop** | Comprehensions shouldn't have side effects |
| Complex logic (multiple ifs, try/except) | **Loop** | Readability first |
| Accumulate (running total) | **Loop** | Comprehensions can't track state |

```python
# ✅ GOOD comprehension — clear and readable
active = [c for c in clients if c["active"]]

# ✅ ACCEPTABLE — a bit complex but still readable
report = [
    f"{c['name']}: {classify(c['revenue'])}"
    for c in clients
    if c["active"] and c["revenue"] >= 500000
]

# ❌ BAD comprehension — too complex, use a loop
# result = [transform(x) for group in data for x in group.items() if validate(x) and x not in seen and (seen.add(x) or True)]

# ✅ Use a loop instead
result = []
seen = set()
for group in data:
    for x in group.items():
        if validate(x) and x not in seen:
            seen.add(x)
            result.append(transform(x))
```

---

### 3.11 Solving Mike's Problem — Data Processing Pipeline

```python
# ============================================
# Pearson Specter Litt — Data Processing Pipeline
# All comprehension-based transformations
# ============================================

raw_clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "type": "Merger", "active": True, "hours": [85, 42, 63]},
    {"name": "Stark Industries", "revenue": 800000, "type": "Patent", "active": True, "hours": [62, 38]},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation", "active": False, "hours": [55, 90, 30]},
    {"name": "Dalton Industries", "revenue": 1200000, "type": "Merger", "active": True, "hours": [70]},
    {"name": "Luthor Corp", "revenue": 150000, "type": "Corporate", "active": True, "hours": [28, 15]},
    {"name": "Gillis Industries", "revenue": 75000, "type": "Patent", "active": True, "hours": [10]},
    {"name": "Queen Industries", "revenue": 3000000, "type": "Merger", "active": True, "hours": [95, 88, 72, 50]},
    {"name": "Palmer Tech", "revenue": 400000, "type": "Patent", "active": False, "hours": [45, 32]},
]

rate = 450.00

# ---- Step 1: Active clients only ----
active = [c for c in raw_clients if c["active"]]
print(f"Active clients: {len(active)}/{len(raw_clients)}")

# ---- Step 2: Enrich with calculated fields ----
enriched = [
    {
        **c,
        "total_hours": sum(c["hours"]),
        "billing": sum(c["hours"]) * rate,
        "tier": (
            "💎 Platinum" if c["revenue"] >= 1000000
            else "🥇 Gold" if c["revenue"] >= 500000
            else "🥈 Silver" if c["revenue"] >= 100000
            else "🥉 Bronze"
        ),
        "avg_case_hours": sum(c["hours"]) / len(c["hours"]) if c["hours"] else 0,
    }
    for c in active
]

# ---- Step 3: Sort by revenue (using sorted + comprehension result) ----
top_clients = sorted(enriched, key=lambda c: c["revenue"], reverse=True)

# ---- Step 4: Build display strings ----
report_lines = [
    f"  {c['tier']:<15} {c['name']:<22} ${c['revenue']:>12,.2f}  "
    f"{c['total_hours']:>4}h  ${c['billing']:>12,.2f}"
    for c in top_clients
]

# ---- Step 5: Aggregate stats with dict comprehension ----
tier_stats = {
    tier: {
        "count": len([c for c in enriched if c["tier"] == tier]),
        "total_revenue": sum(c["revenue"] for c in enriched if c["tier"] == tier),
        "total_billing": sum(c["billing"] for c in enriched if c["tier"] == tier),
    }
    for tier in sorted({c["tier"] for c in enriched})  # Unique tiers via set comp
}

# ---- Step 6: Group by case type ----
case_types = {c["type"] for c in enriched}   # Set comprehension: unique types
by_type = {
    ct: [c["name"] for c in enriched if c["type"] == ct]
    for ct in sorted(case_types)
}

# ---- Display ----
print(f"\n{'='*75}")
print(f"{'PSL CLIENT PIPELINE — COMPREHENSION-POWERED':^75}")
print(f"{'='*75}")

print(f"\n  {'Tier':<15} {'Name':<22} {'Revenue':>12}  {'Hours':>5}  {'Billing':>12}")
print(f"  {'─'*70}")
for line in report_lines:
    print(line)

total_rev = sum(c["revenue"] for c in enriched)
total_bill = sum(c["billing"] for c in enriched)
print(f"  {'─'*70}")
print(f"  {'TOTAL':<15} {'':<22} ${total_rev:>12,.2f}  "
      f"{sum(c['total_hours'] for c in enriched):>4}h  ${total_bill:>12,.2f}")

print(f"\n📊 Tier Summary:")
for tier, stats in tier_stats.items():
    print(f"  {tier}: {stats['count']} clients | ${stats['total_revenue']:,.2f} revenue")

print(f"\n📋 By Case Type:")
for ct, names in by_type.items():
    print(f"  {ct}: {', '.join(names)}")

print(f"\n{'='*75}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Side Effects in Comprehensions 🔥

```python
# ❌ BAD — comprehension used for side effects, result thrown away
[print(x) for x in range(5)]

# This creates a list of None values: [None, None, None, None, None]
# The print happens, but the comprehension itself is wasted

# ✅ Use a loop for side effects
for x in range(5):
    print(x)
```

---

### Pitfall 2: Variable Leaks (Python 2 vs 3)

```python
# In Python 3, comprehension variables DON'T leak
x = "before"
squares = [x for x in range(5)]
print(x)     # "before" — x is unchanged in Python 3 ✅

# But regular for loops DO leak
x = "before"
for x in range(5):
    pass
print(x)     # 4 — x is changed! ⚠️
```

---

### Pitfall 3: Memory with Large Comprehensions

```python
# ❌ Creates the ENTIRE list in memory
huge_list = [x ** 2 for x in range(10_000_000)]
# This allocates memory for 10 million integers!

# ✅ Use a generator expression for large data (covered in Lesson 5)
huge_gen = (x ** 2 for x in range(10_000_000))
# Produces values on-demand — almost no memory!
```

---

### Pitfall 4: `if-else` Position Confusion

```python
# ❌ WRONG position — SyntaxError
# result = [x for x in items if x > 0 else -x]

# ✅ Ternary goes BEFORE the for
result = [x if x > 0 else -x for x in items]

# ✅ Filter goes AFTER the for
result = [x for x in items if x > 0]
```

---

### Pitfall 5: Over-Complex Comprehensions

```python
# ❌ Unreadable — nobody can maintain this
data = {k: [v for v in vals if v > threshold] for k, vals in {k: [int(x) for x in v.split(",")] for k, v in raw.items() if v}.items() if any(int(x) > threshold for x in vals.split(","))}

# ✅ Break it down
parsed = {k: [int(x) for x in v.split(",")]
          for k, v in raw.items() if v}

filtered = {k: [v for v in vals if v > threshold]
            for k, vals in parsed.items()}

data = {k: v for k, v in filtered.items() if v}
```

**Louis:** "Three simple comprehensions that anyone can read beat one monstrosity nobody can."

---

### Pitfall 6: Forgetting That Comprehensions Return New Objects

```python
original = [1, 2, 3, 4, 5]
doubled = [x * 2 for x in original]

# original is UNCHANGED
print(original)   # [1, 2, 3, 4, 5]
print(doubled)    # [2, 4, 6, 8, 10]

# This is DIFFERENT from modifying in-place:
for i in range(len(original)):
    original[i] *= 2
# Now original IS changed: [2, 4, 6, 8, 10]
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏭 Analogy: Comprehension = Assembly Line with Quality Control

```
FOR LOOP (manual process):
  📥 Get item from box
  🔍 Check if it passes quality control (if condition)
  🔧 Transform it (expression)
  📦 Put it in the output box (append)
  🔁 Repeat

COMPREHENSION (automated line):
  [🔧transformed_item  📥for item in box  🔍if quality_check]
  
  The whole line is described in ONE instruction.
```

**Donna:** "Think of it as ordering at a deli:"
```
"I'll take a [sandwich] [for each bread] [if it's fresh]"
         ↕              ↕                ↕
"Give me  [x.upper()]  [for x in items] [if x.isalpha()]"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Transform Practice**
```
Given: numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Use list comprehensions to create:
a) Squares of all numbers
b) Cubes of even numbers only
c) Strings: "1 is odd", "2 is even", etc. for all numbers
d) Absolute values of [-5, -3, 0, 3, 7, -2]
```

**Exercise 2: String Processing**
```
Given: words = ["Hello", " World ", "PYTHON", "  coding  ", "Fun"]

Use comprehensions to create:
a) Lowercase version of all words
b) Stripped and title-cased versions
c) Only words longer than 4 characters (after stripping)
d) First character of each word (stripped)
```

**Exercise 3: Dictionary Building**
```
Given: names = ["Harvey", "Mike", "Louis", "Donna"]
       ages  = [42, 30, 45, 38]

Use comprehensions to create:
a) name → age dictionary (dict comp + zip)
b) name → name_length dictionary
c) Only entries where age > 35
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Matrix Operations**
```
Given:
  matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

Use comprehensions to:
a) Flatten to a single list
b) Transpose the matrix
c) Get all elements > 5
d) Create a new matrix where each cell = original * 10
e) Get the diagonal elements [1, 5, 9]
```

**Exercise 5: Data Grouping**
```
Given a list of employee dicts with: name, department, salary
Use dict comprehension to create:
a) department → [list of names] mapping
b) department → average salary mapping
c) department → highest earner mapping

Use at least 8 employees across 3 departments.
```

**Exercise 6: File Processing**
```
Read a text file line by line. Using comprehensions:
a) Get all non-empty, non-comment lines (stripped)
b) Parse "key=value" lines into a dictionary
c) Count word frequencies using a dict comprehension
d) Extract all words longer than 6 characters (unique, sorted)
```

---

### 🔴 Advanced Exercises

**Exercise 7: Cartesian Product Generator**
```
Write a function: cartesian(*iterables)

Uses nested comprehension to produce all combinations:
  cartesian([1,2], ['a','b'], ['x','y'])
  → [(1,'a','x'), (1,'a','y'), (1,'b','x'), ...]

Handle any number of input iterables.
```

**Exercise 8: Comprehension Pipeline**
```
Build a data pipeline using ONLY comprehensions (no loops):

1. Parse raw CSV text into list of dicts
2. Filter by multiple criteria
3. Enrich with calculated fields
4. Sort by a computed value
5. Format as report strings
6. Group by category (dict comprehension)

Process at least 10 records through all 6 stages.
```

**Exercise 9: Sieve of Eratosthenes**
```
Implement the prime number sieve using comprehensions:

1. Start with candidates = list(range(2, n+1))
2. For each prime found, remove its multiples
3. Return all remaining numbers = primes

Find all primes up to 100.
Compare performance with a loop-based version.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: if-else position
numbers = [1, -2, 3, -4, 5]
absolutes = [x for x in numbers if x >= 0 else -x]

# Bug 2: Missing iterable
result = [x * 2 if x > 0]

# Bug 3: Nested comprehension confusion
matrix = [[1,2],[3,4],[5,6]]
flat = [x for x in row for row in matrix]

# Bug 4: Dict comprehension with duplicate keys
data = [("a", 1), ("b", 2), ("a", 3)]
result = {k: v for k, v in data}
print(result)   # What happened to ("a", 1)?

# Bug 5: Side effect in comprehension
total = 0
[total := total + x for x in range(5)]
# What is total? Is this readable?
```

### Fixed Code ✅

```python
# Fix 1: if-else goes BEFORE for
numbers = [1, -2, 3, -4, 5]
absolutes = [x if x >= 0 else -x for x in numbers]   # ✅ [1, 2, 3, 4, 5]

# Fix 2: Need a for clause
result = [x * 2 for x in range(10) if x > 0]   # ✅

# Fix 3: Outer loop first, inner loop second
matrix = [[1,2],[3,4],[5,6]]
flat = [x for row in matrix for x in row]   # ✅ [1,2,3,4,5,6]

# Fix 4: Later duplicates overwrite — this is expected behavior
# If you need all values, use a different structure:
from collections import defaultdict
result = defaultdict(list)
for k, v in data:
    result[k].append(v)
# {'a': [1, 3], 'b': [2]}

# Fix 5: Use a loop for accumulation — comprehensions shouldn't track state
total = sum(range(5))   # ✅ Use sum() — clean and Pythonic
```

---

## 8. 🌍 Real-World Application

### Where Comprehensions Are Used in Production

| Library | Comprehension Usage |
|---------|-------------------|
| **Django** | `[field.name for field in model._meta.fields]` |
| **pandas** | Column operations, filtering |
| **FastAPI** | `{k: v for k, v in headers.items() if k.startswith("x-")}` |
| **pytest** | `@pytest.mark.parametrize` data generation |
| **Data pipelines** | ETL transforms, cleaning, validation |

### Production Example: Log Processor

```python
def process_logs(log_lines):
    """Process log entries using comprehensions."""
    
    # Parse structured logs
    entries = [
        {
            "timestamp": parts[0],
            "level": parts[1],
            "message": parts[2].strip()
        }
        for line in log_lines
        if line.strip() and not line.startswith("#")
        for parts in [line.strip().split(" | ")]
        if len(parts) >= 3
    ]
    
    # Stats via comprehensions
    level_counts = {
        level: len([e for e in entries if e["level"] == level])
        for level in {e["level"] for e in entries}
    }
    
    errors = [e["message"] for e in entries if e["level"] == "ERROR"]
    
    return {
        "total": len(entries),
        "by_level": level_counts,
        "errors": errors,
        "unique_messages": len({e["message"] for e in entries}),
    }

# Test
logs = [
    "2026-03-13 09:00 | INFO | System started",
    "2026-03-13 09:01 | INFO | Loading clients",
    "2026-03-13 09:02 | ERROR | Database timeout",
    "2026-03-13 09:03 | WARNING | Cache miss",
    "2026-03-13 09:04 | ERROR | Connection refused",
    "2026-03-13 09:05 | INFO | Retry successful",
]

stats = process_logs(logs)
print(f"Total: {stats['total']}")
print(f"By level: {stats['by_level']}")
print(f"Errors: {stats['errors']}")
```

---

## 9. ✅ Knowledge Check

---

**Q1.** Rewrite using a list comprehension:
```python
result = []
for x in range(20):
    if x % 3 == 0:
        result.append(x ** 2)
```

**Q2.** Where does `if` go for filtering? Where does `if-else` go for transformation?

**Q3.** How do you flatten `[[1,2],[3,4],[5,6]]` with a comprehension?

**Q4.** What is a dictionary comprehension? Give the syntax.

**Q5.** When should you NOT use a comprehension?

**Q6.** What is a set comprehension? How does it differ from a list comprehension?

**Q7.** What is the output?
```python
print([x*y for x in [1,2,3] for y in [10,100]])
```

**Q8.** How does the walrus operator (`:=`) help in comprehensions?

**Q9.** Why are comprehensions slightly faster than equivalent `for` loops?

**Q10.** Write a dict comprehension that maps each word in a sentence to its length.

---

### 📋 Answer Key

> **Q1.** `result = [x ** 2 for x in range(20) if x % 3 == 0]`

> **Q2.** Filter `if`: AFTER the `for`. Transform `if-else`: BEFORE the `for`.

> **Q3.** `[x for row in matrix for x in row]`

> **Q4.** `{key_expr: value_expr for item in iterable if condition}` — creates a dictionary.

> **Q5.** Side effects (print, write), complex logic (3+ nested, try/except), accumulating state (running totals).

> **Q6.** `{expr for item in iterable}` — uses `{}` but no colon. Returns a set (unique, unordered) instead of a list.

> **Q7.** `[10, 100, 20, 200, 30, 300]`

> **Q8.** Assigns a value within the expression: `[y for x in data if (y := compute(x)) > threshold]` — avoids recomputing.

> **Q9.** Python internally optimizes the bytecode — it uses a special `LIST_APPEND` instruction that's faster than calling `.append()`.

> **Q10.** `sentence = "the quick brown fox"; {w: len(w) for w in sentence.split()}`

---

## 🎬 End of Lesson Scene

**Mike:** *(gesturing at the data pipeline)* "Twelve lines of comprehensions. Replaces sixty lines of loops. Same result, faster execution, cleaner code."

**Harvey:** "Now THAT is efficient."

**Louis:** "As long as they stay readable. I see one more triple-nested comprehension in production code and someone is getting a performance review."

**Mike:** "Next up: built-in functions in loops. `map()`, `filter()`, `enumerate()`, `zip()`, `sorted()` — the power tools that make comprehensions even more powerful."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | List comprehension basics — `[expr for x in items]` | ✅ |
| 2 | Filtering with `if` (after `for`) | ✅ |
| 3 | Transformation with `if-else` (before `for`) | ✅ |
| 4 | Nested loops in comprehensions | ✅ |
| 5 | Dictionary comprehensions | ✅ |
| 6 | Set comprehensions | ✅ |
| 7 | Nested comprehensions (matrices) | ✅ |
| 8 | Comprehensions with functions | ✅ |
| 9 | Walrus operator in comprehensions | ✅ |
| 10 | Readability guidelines — when NOT to use | ✅ |

---

> **Next Lesson →** Level 2, Lesson 4: *Use Built-In Functions in Python Loops*
