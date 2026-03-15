# 🏛️ Level 2 – Applying | Lesson 4: Use Built-In Functions in Python Loops

## *"The Power Tools"*

> **O'Reilly Skill Objective:** *Use built-in functions inside loops to enhance functionality and efficiency.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, loops, functions, default arguments, `*args`/`**kwargs`, list/dict/set comprehensions, classes, objects, file I/O

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

*You walk into Mike's office. He has a whiteboard full of loop patterns — the same patterns you've been writing for weeks — with red X marks through half of them.*

**Mike:** "We've been doing this wrong. Not wrong exactly, but... amateurish."

```python
# What we've been writing:
index = 0
for client in clients:
    print(f"{index}: {client}")
    index += 1

# What professionals write:
for index, client in enumerate(clients):
    print(f"{index}: {client}")
```

**Harvey:** *(leaning in the doorway)* "Python has dozens of built-in functions designed to work with loops. You've been building tools from scratch when there's a perfectly good toolbox sitting right there."

**Mike:** "Today we learn the power tools: `enumerate()`, `zip()`, `map()`, `filter()`, `sorted()`, `any()`, `all()`, `min()`, `max()`, `sum()`, and `reversed()`. These eliminate 80% of the manual loop logic you've been writing."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "There are two types of programmers."

| Type | Approach | Example |
|------|----------|---------|
| **Beginner** | Writes everything manually | `index = 0; for x in list: index += 1` |
| **Professional** | Uses built-in tools | `for i, x in enumerate(list)` |

**Harvey:** "Python's built-in functions are battle-tested, C-optimized, and universally understood by every Python developer. Using them isn't just shorter — it's the expected professional standard."

**The Big 12:**

| Function | Purpose | Returns |
|----------|---------|---------|
| `enumerate()` | Add index to iteration | Iterator of `(index, item)` |
| `zip()` | Pair up multiple iterables | Iterator of tuples |
| `map()` | Apply function to each item | Iterator |
| `filter()` | Keep items matching a test | Iterator |
| `sorted()` | Sort any iterable | New list |
| `reversed()` | Reverse any sequence | Iterator |
| `any()` | True if ANY item is truthy | `bool` |
| `all()` | True if ALL items are truthy | `bool` |
| `sum()` | Sum of all items | Number |
| `min()` / `max()` | Smallest / largest item | Single item |
| `len()` | Count of items | `int` |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `enumerate()` — Loop with Index ⭐

```python
# ❌ Manual index tracking
index = 0
for client in clients:
    print(f"{index}. {client}")
    index += 1

# ✅ enumerate() — clean, Pythonic, no manual counter
clients = ["Wayne", "Stark", "Oscorp", "Dalton"]

for i, client in enumerate(clients):
    print(f"{i}. {client}")
# 0. Wayne
# 1. Stark
# 2. Oscorp
# 3. Dalton

# Custom start index
for i, client in enumerate(clients, start=1):
    print(f"{i}. {client}")
# 1. Wayne
# 2. Stark
# ...
```

**Practical uses:**
```python
# Find position of a specific item
for i, client in enumerate(clients):
    if client == "Oscorp":
        print(f"Found at index {i}")
        break

# Numbered report
def print_numbered(items, title="Items"):
    print(f"\n{title}:")
    for i, item in enumerate(items, 1):
        print(f"  {i:>3}. {item}")

print_numbered(clients, "Client Roster")
```

---

### 3.2 `zip()` — Parallel Iteration ⭐

```python
# ❌ Manual index to combine lists
names = ["Harvey", "Mike", "Louis"]
rates = [450, 375, 425]
titles = ["Partner", "Associate", "Partner"]

for i in range(len(names)):
    print(f"{names[i]} ({titles[i]}): ${rates[i]}/hr")

# ✅ zip() — iterate multiple lists in parallel
for name, rate, title in zip(names, rates, titles):
    print(f"{name} ({title}): ${rate}/hr")
```

**Key behaviors:**
```python
# zip stops at the SHORTEST iterable
a = [1, 2, 3]
b = ["x", "y"]
print(list(zip(a, b)))   # [(1, 'x'), (2, 'y')]  — 3 is dropped!

# Use zip_longest to keep ALL items
from itertools import zip_longest
print(list(zip_longest(a, b, fillvalue="?")))
# [(1, 'x'), (2, 'y'), (3, '?')]
```

**Build dicts from parallel lists:**
```python
keys = ["name", "revenue", "type"]
values = ["Wayne", 2500000, "Merger"]

client = dict(zip(keys, values))
print(client)   # {'name': 'Wayne', 'revenue': 2500000, 'type': 'Merger'}
```

**Unzipping:**
```python
pairs = [("Wayne", 2500000), ("Stark", 800000), ("Oscorp", 600000)]
names, revenues = zip(*pairs)    # Unzip with *
print(names)      # ('Wayne', 'Stark', 'Oscorp')
print(revenues)   # (2500000, 800000, 600000)
```

---

### 3.3 `map()` — Transform Every Item

```python
# ❌ Manual loop
squared = []
for n in [1, 2, 3, 4, 5]:
    squared.append(n ** 2)

# ✅ map() — apply a function to every item
squared = list(map(lambda n: n ** 2, [1, 2, 3, 4, 5]))
print(squared)   # [1, 4, 9, 16, 25]

# With a named function
def classify(revenue):
    if revenue >= 1000000: return "Platinum"
    elif revenue >= 500000: return "Gold"
    else: return "Silver"

revenues = [2500000, 800000, 600000, 150000]
tiers = list(map(classify, revenues))
print(tiers)   # ['Platinum', 'Gold', 'Gold', 'Silver']

# map() with multiple iterables (like zip + transform)
names = ["wayne", "stark", "oscorp"]
revenues = [2500000, 800000, 600000]

labels = list(map(lambda n, r: f"{n.title()}: ${r:,.0f}", names, revenues))
print(labels)
# ['Wayne: $2,500,000', 'Stark: $800,000', 'Oscorp: $600,000']
```

**Mike:** "`map()` vs comprehension — which to use?"

```python
# These are equivalent:
result = list(map(str.upper, names))
result = [name.upper() for name in names]

# Comprehension is usually preferred because:
# 1. More readable
# 2. Can add conditions easily
# 3. Doesn't need lambda for simple expressions
# Use map() when you already have a function by name:
result = list(map(len, names))     # map is cleaner here
result = [len(name) for name in names]  # also fine
```

---

### 3.4 `filter()` — Keep Matching Items

```python
# ❌ Manual loop with condition
active = []
for client in clients:
    if client["active"]:
        active.append(client)

# ✅ filter() — keep items where function returns True
def is_active(client):
    return client["active"]

active = list(filter(is_active, clients))

# With lambda
active = list(filter(lambda c: c["active"], clients))

# Remove falsy values (None, 0, "", False, [], {})
data = [0, 1, "", "hello", None, 42, [], [1, 2], False, True]
cleaned = list(filter(None, data))   # filter(None, ...) removes falsy values
print(cleaned)   # [1, 'hello', 42, [1, 2], True]
```

**Mike:** "Again, comprehensions are usually preferred:"
```python
# Equivalent — comprehension is more Pythonic
active = [c for c in clients if c["active"]]
```

---

### 3.5 `sorted()` — Sort Any Iterable ⭐

```python
# Basic sorting
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
print(sorted(numbers))              # [1, 1, 2, 3, 4, 5, 6, 9]
print(sorted(numbers, reverse=True)) # [9, 6, 5, 4, 3, 2, 1, 1]
# Original unchanged!
print(numbers)   # [3, 1, 4, 1, 5, 9, 2, 6]

# Sort with key function
names = ["Mike", "Harvey", "Louis", "Donna", "Rachel"]
print(sorted(names))                          # Alphabetical
print(sorted(names, key=len))                 # By length
print(sorted(names, key=lambda n: n[-1]))     # By last character
```

**Sorting complex objects:**
```python
clients = [
    {"name": "Wayne", "revenue": 2500000},
    {"name": "Stark", "revenue": 800000},
    {"name": "Oscorp", "revenue": 600000},
    {"name": "Dalton", "revenue": 1200000},
]

# Sort by revenue (descending)
by_revenue = sorted(clients, key=lambda c: c["revenue"], reverse=True)
for c in by_revenue:
    print(f"  {c['name']}: ${c['revenue']:,.0f}")

# Sort by multiple criteria (revenue desc, then name asc)
from operator import itemgetter
by_multi = sorted(clients, key=lambda c: (-c["revenue"], c["name"]))
```

**`sorted()` vs `.sort()`:**

| Feature | `sorted(iterable)` | `list.sort()` |
|---------|-------------------|--------------|
| Returns | New sorted list | `None` (sorts in-place) |
| Works on | Any iterable | Lists only |
| Original | Unchanged | Modified |
| Use when | Need original preserved | Don't need original |

---

### 3.6 `reversed()` — Iterate in Reverse

```python
# reversed() returns an iterator — doesn't create a new list
names = ["Harvey", "Mike", "Louis"]

for name in reversed(names):
    print(name)
# Louis, Mike, Harvey

# Convert to list if needed
rev_list = list(reversed(names))
print(rev_list)   # ['Louis', 'Mike', 'Harvey']

# Original unchanged
print(names)   # ['Harvey', 'Mike', 'Louis']

# Works with range too
for i in reversed(range(5)):
    print(i)   # 4, 3, 2, 1, 0

# Alternative: slicing (creates a new list)
rev_list = names[::-1]    # Also reverses, but creates a copy
```

---

### 3.7 `any()` and `all()` — Boolean Aggregation ⭐

```python
# any() — True if ANY element is truthy
# all() — True if ALL elements are truthy

scores = [85, 92, 78, 95, 88]

print(any(s > 90 for s in scores))    # True  — at least one > 90
print(all(s > 70 for s in scores))    # True  — all > 70
print(all(s > 80 for s in scores))    # False — 78 is not > 80
```

**Practical uses:**
```python
clients = [
    {"name": "Wayne", "active": True, "revenue": 2500000},
    {"name": "Stark", "active": True, "revenue": 800000},
    {"name": "Oscorp", "active": False, "revenue": 600000},
]

# Are ALL clients active?
all_active = all(c["active"] for c in clients)
print(f"All active: {all_active}")   # False

# Is ANY client a Platinum tier?
has_platinum = any(c["revenue"] >= 1000000 for c in clients)
print(f"Has Platinum: {has_platinum}")   # True

# Validation: check all required fields present
required = ["name", "revenue", "active"]
client = {"name": "Wayne", "revenue": 2500000}
all_present = all(field in client for field in required)
print(f"All fields present: {all_present}")   # False — 'active' missing

# Short-circuit behavior
# any() stops at the FIRST True
# all() stops at the FIRST False
# This makes them efficient for large datasets
```

---

### 3.8 `sum()`, `min()`, `max()` — Numeric Aggregation

```python
revenues = [2500000, 800000, 600000, 1200000, 150000]

print(f"Total:   ${sum(revenues):,.0f}")       # $5,250,000
print(f"Minimum: ${min(revenues):,.0f}")       # $150,000
print(f"Maximum: ${max(revenues):,.0f}")       # $2,500,000
print(f"Average: ${sum(revenues)/len(revenues):,.0f}")  # $1,050,000

# With key function — find item with min/max of a computed value
clients = [
    {"name": "Wayne", "revenue": 2500000},
    {"name": "Stark", "revenue": 800000},
    {"name": "Oscorp", "revenue": 600000},
]

biggest = max(clients, key=lambda c: c["revenue"])
print(f"Biggest client: {biggest['name']}")   # Wayne

smallest = min(clients, key=lambda c: c["revenue"])
print(f"Smallest client: {smallest['name']}")   # Oscorp

# sum() with generator expressions — memory efficient
total = sum(c["revenue"] for c in clients)
print(f"Total: ${total:,.0f}")

# sum() with a start value
existing_total = 1000000
grand_total = sum(revenues, start=existing_total)
```

---

### 3.9 Combining Built-Ins — The Power Chain

**Mike:** "The real magic happens when you chain these together."

```python
attorneys = [
    {"name": "Harvey", "rate": 450, "hours": [85, 42, 63], "active": True},
    {"name": "Mike", "rate": 375, "hours": [62, 38, 55], "active": True},
    {"name": "Louis", "rate": 425, "hours": [70, 45], "active": True},
    {"name": "Katrina", "rate": 350, "hours": [48, 52, 30], "active": False},
]

# Chain 1: numbered list of active attorneys sorted by total billing
for i, atty in enumerate(
    sorted(
        filter(lambda a: a["active"], attorneys),
        key=lambda a: sum(a["hours"]) * a["rate"],
        reverse=True
    ),
    start=1
):
    billing = sum(atty["hours"]) * atty["rate"]
    print(f"  {i}. {atty['name']}: ${billing:,.0f}")

# Chain 2: parallel summary of names and their billing
names_and_billing = [
    (a["name"], sum(a["hours"]) * a["rate"])
    for a in attorneys
    if a["active"]
]
names, billings = zip(*names_and_billing)
print(f"\n  Total billing: ${sum(billings):,.0f}")
print(f"  Best performer: {names[billings.index(max(billings))]}")
```

---

### 3.10 Solving Harvey's Problem — Analytics Dashboard

```python
# ============================================
# Pearson Specter Litt — Analytics Dashboard
# Powered by Built-In Functions
# ============================================

clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "type": "Merger", "active": True, "attorney": "Harvey", "hours": 190},
    {"name": "Stark Industries", "revenue": 800000, "type": "Patent", "active": True, "attorney": "Mike", "hours": 155},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation", "active": False, "attorney": "Louis", "hours": 175},
    {"name": "Dalton Industries", "revenue": 1200000, "type": "Merger", "active": True, "attorney": "Harvey", "hours": 112},
    {"name": "Luthor Corp", "revenue": 150000, "type": "Corporate", "active": True, "attorney": "Louis", "hours": 43},
    {"name": "Queen Industries", "revenue": 3000000, "type": "Merger", "active": True, "attorney": "Harvey", "hours": 305},
    {"name": "Palmer Tech", "revenue": 400000, "type": "Patent", "active": False, "attorney": "Mike", "hours": 77},
    {"name": "Gillis Industries", "revenue": 75000, "type": "Patent", "active": True, "attorney": "Mike", "hours": 10},
]

rate = 450

print("=" * 65)
print(f"{'🏛️  PSL ANALYTICS DASHBOARD':^65}")
print("=" * 65)

# ---- Portfolio Overview (sum, len, min, max, any, all) ----
active = list(filter(lambda c: c["active"], clients))
revenues = [c["revenue"] for c in active]

print(f"\n📊 PORTFOLIO OVERVIEW")
print(f"  Total Clients:    {len(clients)} ({len(active)} active)")
print(f"  Total Revenue:    ${sum(revenues):>14,.2f}")
print(f"  Average Revenue:  ${sum(revenues)/len(revenues):>14,.2f}")
print(f"  Highest Revenue:  ${max(revenues):>14,.2f}  "
      f"({max(active, key=lambda c: c['revenue'])['name']})")
print(f"  Lowest Revenue:   ${min(revenues):>14,.2f}  "
      f"({min(active, key=lambda c: c['revenue'])['name']})")
print(f"  All Active?       {'Yes' if all(c['active'] for c in clients) else 'No'}")
print(f"  Any Platinum?     {'Yes' if any(r >= 1000000 for r in revenues) else 'No'}")

# ---- Ranked Client List (sorted, enumerate, zip) ----
print(f"\n📋 CLIENTS RANKED BY REVENUE")
print(f"  {'#':>3}  {'Name':<22} {'Revenue':>14}  {'Type':<12}  {'Attorney'}")
print(f"  {'─'*65}")

for rank, client in enumerate(
    sorted(active, key=lambda c: c["revenue"], reverse=True),
    start=1
):
    tier = "💎" if client["revenue"] >= 1000000 else "🥇" if client["revenue"] >= 500000 else "🥈"
    print(f"  {rank:>3}  {tier} {client['name']:<20} ${client['revenue']:>13,.2f}  "
          f"{client['type']:<12}  {client['attorney']}")

# ---- Attorney Performance (zip, map, sorted) ----
attorney_names = sorted({c["attorney"] for c in active})

print(f"\n👔 ATTORNEY PERFORMANCE")
print(f"  {'Attorney':<12} {'Clients':>8} {'Total Hours':>12} {'Revenue':>14}")
print(f"  {'─'*50}")

for atty in attorney_names:
    atty_clients = [c for c in active if c["attorney"] == atty]
    total_hours = sum(c["hours"] for c in atty_clients)
    total_rev = sum(c["revenue"] for c in atty_clients)
    print(f"  {atty:<12} {len(atty_clients):>8} {total_hours:>12} ${total_rev:>13,.2f}")

# ---- Case Type Breakdown (sorted, sum, map) ----
case_types = sorted({c["type"] for c in active})

print(f"\n💼 CASE TYPE BREAKDOWN")
for ct in case_types:
    ct_clients = [c for c in active if c["type"] == ct]
    ct_rev = sum(c["revenue"] for c in ct_clients)
    pct = ct_rev / sum(revenues) * 100
    bar = "█" * int(pct / 5) + "░" * (20 - int(pct / 5))
    print(f"  {ct:<12} [{bar}] {pct:5.1f}%  (${ct_rev:,.0f})")

# ---- Alerts (any, all, filter) ----
print(f"\n⚠️  ALERTS")
inactive = list(filter(lambda c: not c["active"], clients))
if inactive:
    print(f"  🔴 {len(inactive)} inactive client(s): "
          f"{', '.join(c['name'] for c in inactive)}")

low_rev = list(filter(lambda c: c["revenue"] < 100000, active))
if low_rev:
    print(f"  🟡 {len(low_rev)} below-threshold client(s): "
          f"{', '.join(c['name'] for c in low_rev)}")

if all(c["revenue"] >= 500000 for c in active):
    print(f"  🟢 All active clients meet minimum revenue threshold")

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `map()` and `filter()` Return Iterators, Not Lists

```python
result = map(str.upper, ["hello", "world"])
print(result)        # <map object at 0x...> — NOT a list!
print(list(result))  # ['HELLO', 'WORLD']

# ⚠️ Iterators are exhausted after one use!
result = filter(lambda x: x > 0, [-1, 2, -3, 4])
print(list(result))  # [2, 4]
print(list(result))  # [] — EMPTY! Already consumed!
```

---

### Pitfall 2: `sorted()` vs `.sort()` — Return Values

```python
numbers = [3, 1, 4, 1, 5]

# sorted() returns a NEW list — original unchanged
new_list = sorted(numbers)
print(numbers)     # [3, 1, 4, 1, 5] — unchanged
print(new_list)    # [1, 1, 3, 4, 5]

# .sort() modifies IN-PLACE — returns None!
result = numbers.sort()
print(result)      # None ← TRAP!
print(numbers)     # [1, 1, 3, 4, 5] — modified

# ❌ Common bug
# for x in numbers.sort():  # TypeError: cannot iterate over None!
```

---

### Pitfall 3: `zip()` Stops at Shortest

```python
names = ["Harvey", "Mike", "Louis", "Donna"]    # 4 items
scores = [95, 88, 72]                             # 3 items

paired = list(zip(names, scores))
print(paired)   # [('Harvey', 95), ('Mike', 88), ('Louis', 72)]
# Donna is SILENTLY DROPPED!

# ✅ Use strict=True (Python 3.10+) to catch mismatches
# list(zip(names, scores, strict=True))  # ValueError!

# ✅ Or use zip_longest
from itertools import zip_longest
paired = list(zip_longest(names, scores, fillvalue=0))
# [('Harvey', 95), ('Mike', 88), ('Louis', 72), ('Donna', 0)]
```

---

### Pitfall 4: `min()` / `max()` on Empty Iterables

```python
# ❌ Crashes on empty!
# min([])   # ValueError: min() arg is an empty sequence
# max([])   # ValueError

# ✅ Use default parameter
print(min([], default=0))     # 0
print(max([], default=None))  # None

# ✅ Or check first
items = []
if items:
    print(min(items))
else:
    print("No items")
```

---

### Pitfall 5: `sum()` Only Works with Numbers

```python
# ✅ Numbers work
print(sum([1, 2, 3]))        # 6
print(sum([1.5, 2.5, 3.0]))  # 7.0

# ❌ Strings don't work with sum()
# sum(["a", "b", "c"])  # TypeError

# ✅ Use join for strings
print("".join(["a", "b", "c"]))   # "abc"

# ✅ sum() with start for list flattening
nested = [[1, 2], [3, 4], [5, 6]]
flat = sum(nested, start=[])       # Works but slow for large lists!
# Better: [x for sub in nested for x in sub]
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🧰 Analogy: Built-In Functions = Professional Kitchen Equipment

```
Amateur Kitchen (Manual Loops):
  🔪 Chop each vegetable by hand (for loop + counter)
  🍳 Pan-fry each piece individually (manual filter)
  📝 Count ingredients one by one (manual sum)

Professional Kitchen (Built-Ins):
  🔪 Food processor (map) — transform everything at once
  🧹 Sieve (filter) — separate what you want automatically
  🔢 Scale (sum/min/max) — instant measurements
  📋 Label maker (enumerate) — auto-number everything
  🤝 Plating station (zip) — pair items perfectly
  ⚖️ Quality check (any/all) — instant pass/fail
  📊 Sorting tray (sorted) — organize in any order
```

**Donna:** "A professional chef doesn't hand-chop 200 carrots. They use a food processor. Same result, fraction of the time, and everyone understands what 'food processor' means."

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: enumerate() Practice**
```
Given: fruits = ["apple", "banana", "cherry", "date", "elderberry"]

a) Print each fruit with its 1-based index
b) Find the index of "cherry"
c) Create a dict mapping index → fruit using enumerate
```

**Exercise 2: zip() Practice**
```
Given: 
  students = ["Alice", "Bob", "Charlie"]
  math = [88, 92, 75]
  science = [91, 85, 88]

a) Print each student with both scores
b) Create a list of dicts: [{"name": ..., "math": ..., "science": ...}]
c) Calculate each student's average using zip
```

**Exercise 3: Aggregation Practice**
```
Given: scores = [78, 92, 45, 88, 100, 63, 71, 95, 82, 57]

Calculate using ONLY built-ins (no loops):
  a) Total, minimum, maximum, count
  b) Average (using sum and len)
  c) Are all scores passing (>= 60)?
  d) Is any score perfect (100)?
  e) Sorted ascending and descending
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Data Pipeline**
```
Given a list of employee dicts (name, department, salary, active):

Using ONLY built-in functions (no manual loops):
a) Filter active employees
b) Sort by salary descending
c) Get the top 3 earners
d) Calculate total and average salary
e) Check if any employee earns > 200k
f) Get the department of the highest earner
```

**Exercise 5: Parallel Processing**
```
Given three lists: names, scores, grades
(grades need to be calculated from scores)

Using zip, map, enumerate, sorted:
a) Create student records by zipping all three
b) Add calculated grades using map
c) Sort by score descending
d) Print a numbered report using enumerate
```

**Exercise 6: Validation Engine**
```
Write a validate(data, rules) function where:
  - data is a dict
  - rules is a list of (field, check_func, error_msg)

Uses all(), any(), map(), filter() to:
  a) Check all required fields exist
  b) Run all validation functions
  c) Return (is_valid, list_of_errors)

Test with client data and 5+ validation rules.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Custom sorted() Implementation**
```
Implement your own sorted function:
  my_sorted(iterable, key=None, reverse=False)

Use it to sort:
  a) Numbers
  b) Strings by length
  c) Dicts by a field value
  d) Compare performance with built-in sorted()
```

**Exercise 8: MapReduce Pattern**
```
Implement a simple MapReduce using built-ins:

  def map_reduce(data, mapper, reducer):
      mapped = map(mapper, data)
      return reduce(reducer, mapped)

Use it to:
  a) Calculate total revenue (map to revenue, reduce by sum)
  b) Find longest name (map to length, reduce by max)
  c) Concatenate all names (map to name, reduce by join)
```

**Exercise 9: Full Analytics Suite**
```
Build a complete analytics class using ONLY built-in functions:

class Analytics:
    def __init__(self, data):
    def summary(self) → dict with count, sum, mean, min, max, range
    def top_n(self, n, key) → top N items by key
    def bottom_n(self, n, key) → bottom N items
    def filter_by(self, **criteria) → filtered data
    def group_by(self, field) → dict of field → [items]
    def percentile(self, field, p) → value at pth percentile
    def correlate(self, field1, field2) → "positive", "negative", "none"
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Iterator exhaustion
mapped = map(lambda x: x * 2, [1, 2, 3])
print(sum(mapped))        # 12
print(list(mapped))       # Expected: [2, 4, 6]

# Bug 2: .sort() returns None
data = [3, 1, 4]
for x in data.sort():
    print(x)

# Bug 3: zip silent truncation
names = ["A", "B", "C"]
scores = [100, 90]
report = dict(zip(names, scores))
print(report)    # Expected all 3 names

# Bug 4: sum with strings
words = ["hello", "world"]
result = sum(words)

# Bug 5: min/max on empty
def get_cheapest(items):
    prices = [i["price"] for i in items if i["in_stock"]]
    return min(prices)

get_cheapest([])   # What happens?
```

### Fixed Code ✅

```python
# Fix 1: Convert to list FIRST, or don't reuse
mapped = list(map(lambda x: x * 2, [1, 2, 3]))  # ✅ Convert once
print(sum(mapped))        # 12
print(mapped)             # [2, 4, 6]

# Fix 2: sorted() returns a new list; .sort() returns None
data = [3, 1, 4]
for x in sorted(data):    # ✅ Use sorted()
    print(x)

# Fix 3: Use zip_longest or validate lengths
from itertools import zip_longest
names = ["A", "B", "C"]
scores = [100, 90]
report = dict(zip_longest(names, scores, fillvalue=0))  # ✅
print(report)   # {'A': 100, 'B': 90, 'C': 0}

# Fix 4: Use join for strings
words = ["hello", "world"]
result = " ".join(words)   # ✅ "hello world"

# Fix 5: Handle empty with default
def get_cheapest(items):
    prices = [i["price"] for i in items if i.get("in_stock")]
    return min(prices, default=None)   # ✅ default for empty
```

---

## 8. 🌍 Real-World Application

### Production Example: Data Validation Pipeline

```python
def validate_client_data(clients):
    """Validate client records using built-in functions."""
    
    required_fields = ["name", "revenue", "type"]
    valid_types = {"Merger", "Patent", "Litigation", "Corporate"}
    
    errors = []
    
    for i, client in enumerate(clients, 1):
        # Check required fields (all)
        if not all(field in client for field in required_fields):
            missing = [f for f in required_fields if f not in client]
            errors.append(f"Record {i}: Missing fields: {missing}")
            continue
        
        # Check revenue is positive (filter + any)
        if client["revenue"] < 0:
            errors.append(f"Record {i}: Negative revenue")
        
        # Check valid type
        if client["type"] not in valid_types:
            errors.append(f"Record {i}: Invalid type '{client['type']}'")
    
    return {
        "total": len(clients),
        "valid": len(clients) - len(errors),
        "errors": errors,
        "all_valid": len(errors) == 0,
        "error_rate": len(errors) / len(clients) * 100 if clients else 0,
    }

# Test
test_data = [
    {"name": "Wayne", "revenue": 2500000, "type": "Merger"},
    {"name": "Stark", "revenue": -100, "type": "Patent"},      # Bad revenue
    {"revenue": 600000, "type": "Litigation"},                  # Missing name
    {"name": "Dalton", "revenue": 1200000, "type": "Whodunit"}, # Bad type
]

result = validate_client_data(test_data)
print(f"Valid: {result['valid']}/{result['total']} ({100-result['error_rate']:.0f}%)")
for err in result["errors"]:
    print(f"  ❌ {err}")
```

---

## 9. ✅ Knowledge Check

---

**Q1.** What does `enumerate()` return? How do you start numbering from 1?

**Q2.** What happens when `zip()` is given iterables of different lengths?

**Q3.** What is the difference between `map()` + `lambda` and a list comprehension?

**Q4.** What does `filter(None, iterable)` do?

**Q5.** What is the difference between `sorted()` and `.sort()`?

**Q6.** What do `any()` and `all()` do? When does each short-circuit?

**Q7.** What happens if you call `min([])` or `max([])`?

**Q8.** Why can't you iterate over a `map()` result twice?

**Q9.** Write one line using `sum()` and a generator to total revenues > 500000.

**Q10.** Chain `filter`, `sorted`, and `enumerate` to: filter active clients, sort by revenue, and number them starting from 1.

---

### 📋 Answer Key

> **Q1.** An iterator of `(index, item)` tuples. `enumerate(items, start=1)`.

> **Q2.** Stops at the shortest. Use `itertools.zip_longest` for padding, or `strict=True` (3.10+) to raise an error.

> **Q3.** Functionally equivalent. Comprehensions are usually more readable; `map()` is cleaner when you already have a named function.

> **Q4.** Removes all falsy values (`None`, `0`, `""`, `False`, `[]`, `{}`).

> **Q5.** `sorted()` returns a new list. `.sort()` modifies in-place and returns `None`.

> **Q6.** `any()` = True if any element is truthy (short-circuits on first True). `all()` = True if all are truthy (short-circuits on first False).

> **Q7.** `ValueError: min() arg is an empty sequence`. Fix: `min([], default=0)`.

> **Q8.** `map()` returns an iterator — iterators are consumed on first pass.

> **Q9.** `total = sum(c["revenue"] for c in clients if c["revenue"] > 500000)`

> **Q10.**
```python
for i, c in enumerate(sorted(filter(lambda c: c["active"], clients), key=lambda c: c["revenue"], reverse=True), 1):
    print(f"{i}. {c['name']}")
```

---

## 🎬 End of Lesson Scene

**Mike:** "Twelve built-in functions. Each one replaces 3-10 lines of manual loop code."

**Harvey:** "And the chains — filter then sort then enumerate — that's how analytics tools are built."

**Mike:** "Next: generator expressions. Same syntax as comprehensions, but they don't store everything in memory. Perfect for processing millions of records."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `enumerate()` — loop with index | ✅ |
| 2 | `zip()` — parallel iteration + unzipping | ✅ |
| 3 | `map()` — transform every item | ✅ |
| 4 | `filter()` — keep matching items | ✅ |
| 5 | `sorted()` — sort any iterable with `key` | ✅ |
| 6 | `reversed()` — iterate backwards | ✅ |
| 7 | `any()` / `all()` — boolean aggregation | ✅ |
| 8 | `sum()` / `min()` / `max()` — numeric aggregation | ✅ |
| 9 | Chaining built-ins together | ✅ |
| 10 | Iterator exhaustion awareness | ✅ |

---

> **Next Lesson →** Level 2, Lesson 5: *Use Generator Expressions in Python*
