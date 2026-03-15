# 🏛️ Level 2 – Applying | Lesson 5: Use Generator Expressions in Python

## *"The Assembly Line"*

> **O'Reilly Skill Objective:** *Use generator expressions to create iterators for memory efficiency.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, loops, functions, default arguments, `*args`/`**kwargs`, comprehensions, built-in functions (`enumerate`, `zip`, `map`, `filter`, `sorted`, `any`, `all`, `sum`, `min`, `max`), classes, objects, file I/O

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

*Thursday morning. Harvey's standing behind Mike, watching a progress bar crawl on-screen.*

**Harvey:** "Why is this taking so long?"

**Mike:** "We're loading 4 million billing records into a list, filtering, sorting, and then—"

**Harvey:** "How much memory?"

**Mike:** *(checking)* "About 3.2 GB. And it crashed twice last night."

*Harvey turns to you.*

**Harvey:** "Yesterday's lesson — comprehensions and built-in functions — they're fast, but they load everything into memory. A list comprehension that processes 10 million records creates a 10-million-item list. That's fine for 237 clients. For 4 million billing records? The system runs out of memory."

**Mike:** "Generators solve this. They produce one item at a time — on demand — and throw it away after processing. Instead of loading 4 million records into a list, you process them one by one. Memory usage? Near zero."

```python
# List comprehension — 4 million items in memory at once
bills = [process(record) for record in load_4m_records()]   # 💥 3.2GB RAM

# Generator expression — one item at a time
bills = (process(record) for record in load_4m_records())   # 📦 ~0 MB
```

**Harvey:** "Same syntax. Parentheses instead of brackets. 3.2 GB down to nothing."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Strategy: don't stockpile what you can produce on demand."

| List Comprehension | Generator Expression |
|-------------------|---------------------|
| `[expr for x in items]` | `(expr for x in items)` |
| Square brackets `[]` | Parentheses `()` |
| Creates full list in memory | Creates a lazy iterator |
| All items computed upfront | Items computed one at a time |
| Can index, slice, reuse | One-time, forward-only |
| Memory = O(n) | Memory = O(1) |
| Fast for small data | Essential for large data |

**Harvey:** "The analogy? A list comprehension is like printing all the case files and stacking them on your desk. A generator is like having Donna pull one file at a time when you need it — the filing cabinet stays closed until you ask."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Generator Expressions

```python
# List comprehension — all values computed and stored
squares_list = [x ** 2 for x in range(10)]
print(squares_list)      # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(type(squares_list)) # <class 'list'>

# Generator expression — values produced on demand
squares_gen = (x ** 2 for x in range(10))
print(squares_gen)       # <generator object <genexpr> at 0x...>
print(type(squares_gen))  # <class 'generator'>

# Consume the generator
for sq in squares_gen:
    print(sq, end=" ")
# 0 1 4 9 16 25 36 49 64 81
```

**Mike:** "The generator doesn't compute anything until you ask for it. Each value is produced on-the-fly, used, then discarded."

---

### 3.2 Memory Comparison — The Core Advantage ⭐

```python
import sys

# List: stores ALL items
list_comp = [x ** 2 for x in range(1_000_000)]
print(f"List:      {sys.getsizeof(list_comp):>12,} bytes")  # ~8.4 MB

# Generator: stores only the recipe
gen_expr = (x ** 2 for x in range(1_000_000))
print(f"Generator: {sys.getsizeof(gen_expr):>12,} bytes")   # ~200 bytes!

# The generator is 40,000x smaller!
```

```python
# Processing 10 million records:
# List approach → 10M items × ~28 bytes each ≈ 280 MB
# Generator approach → 1 item at a time ≈ 200 bytes total

# Total sum of 10 million squares — proof it works
total = sum(x ** 2 for x in range(10_000_000))
print(f"Sum: {total:,}")
# Computed using almost no memory!
```

---

### 3.3 Using Generators with Built-In Functions

**Mike:** "Generators work seamlessly with every built-in function from Lesson 4."

```python
clients = [
    {"name": "Wayne", "revenue": 2500000, "active": True},
    {"name": "Stark", "revenue": 800000, "active": True},
    {"name": "Oscorp", "revenue": 600000, "active": False},
    {"name": "Dalton", "revenue": 1200000, "active": True},
    {"name": "Luthor", "revenue": 150000, "active": True},
]

# sum() with generator — no intermediate list!
total = sum(c["revenue"] for c in clients if c["active"])
print(f"Total active revenue: ${total:,.0f}")

# any() with generator — short-circuits!
has_platinum = any(c["revenue"] >= 1000000 for c in clients)
print(f"Has Platinum: {has_platinum}")

# all() with generator
all_active = all(c["active"] for c in clients)
print(f"All active: {all_active}")

# max() with generator + key
biggest = max(clients, key=lambda c: c["revenue"])
print(f"Biggest: {biggest['name']}")

# min() with generator
smallest_active_rev = min(
    (c["revenue"] for c in clients if c["active"]),
    default=0
)
print(f"Smallest active revenue: ${smallest_active_rev:,.0f}")
```

> 💡 **Note:** When passing a generator to a function as the **only argument**, you can omit the extra parentheses:
> ```python
> sum(x**2 for x in range(10))    # ✅ No double parens needed
> sum((x**2 for x in range(10)))  # ✅ Also works, but redundant
> ```

---

### 3.4 Chaining Generators — The Pipeline ⭐

**Mike:** "The real power — generators can feed into other generators, creating a processing pipeline that uses almost no memory."

```python
# Data pipeline: Load → Filter → Transform → Aggregate
# Each stage is a generator — only one record is in memory at a time!

raw_records = [
    "Wayne Enterprises,2500000,Merger,active",
    "Stark Industries,800000,Patent,active",
    "Oscorp,600000,Litigation,inactive",
    "Dalton Industries,1200000,Merger,active",
    "Luthor Corp,150000,Corporate,active",
    "Gillis Industries,75000,Patent,active",
    "Queen Industries,3000000,Merger,active",
]

# Stage 1: Parse (generator)
parsed = (
    {
        "name": parts[0],
        "revenue": int(parts[1]),
        "type": parts[2],
        "status": parts[3],
    }
    for line in raw_records
    for parts in [line.split(",")]
)

# Stage 2: Filter (generator feeding from generator)
active_only = (record for record in parsed if record["status"] == "active")

# Stage 3: Enrich (generator feeding from generator)
enriched = (
    {
        **record,
        "tier": "Platinum" if record["revenue"] >= 1000000 else "Gold" 
                if record["revenue"] >= 500000 else "Silver"
    }
    for record in active_only
)

# Stage 4: Consume — NOTHING has been computed until here!
for client in enriched:
    print(f"  {client['tier']:<10} {client['name']:<22} ${client['revenue']:>12,}")

# The entire pipeline processes each record end-to-end
# before moving to the next — like an assembly line
```

---

### 3.5 Generator Expressions vs `range()` — Both Are Lazy

```python
# range() is ALREADY lazy — it doesn't create a list!
r = range(1_000_000_000)    # Billion items — instant, no memory
print(sys.getsizeof(r))     # 48 bytes!
print(500_000 in r)          # True — O(1) membership test
print(r[999])                # 999 — O(1) indexing

# Generator expressions are also lazy, but:
# - No indexing (can't do gen[5])
# - No len() support
# - No membership testing
# - One-time use only

gen = (x for x in range(10))
# gen[5]      # ❌ TypeError
# len(gen)    # ❌ TypeError
# 5 in gen    # Works but CONSUMES elements up to 5!
```

---

### 3.6 Converting Between Lists and Generators

```python
# Generator → List (when you need random access or reuse)
gen = (x ** 2 for x in range(10))
lst = list(gen)         # Materializes all values
print(lst[5])            # 25 — indexing works now

# List → Generator (when you need laziness)
data = [1, 2, 3, 4, 5]
gen = (x for x in data)  # Wrap in a generator
# But usually just iterate the list directly

# When to convert:
# Generator → List: when you need len(), indexing, multiple passes
# List → Generator: rarely needed — just use generator from the start
```

---

### 3.7 Generator Expressions with Conditional Logic

```python
# Filter + Transform
clients = [
    ("Wayne", 2500000), ("Stark", 800000), ("Oscorp", 600000),
    ("Luthor", 150000), ("Gillis", 75000),
]

# Generator with if (filter)
big_clients = (name for name, rev in clients if rev >= 500000)

# Generator with if-else (transform)
labels = (
    f"💎 {name}" if rev >= 1000000 else f"🥇 {name}" if rev >= 500000 else f"🥈 {name}"
    for name, rev in clients
)

for label in labels:
    print(f"  {label}")
```

---

### 3.8 Nested Generators

```python
# Flatten nested data lazily
departments = {
    "Corporate": ["Harvey", "Jessica"],
    "Litigation": ["Louis", "Katrina"],
    "IP": ["Mike"],
}

# Nested generator — flatten dept → attorneys
all_attorneys = (
    (dept, attorney)
    for dept, attorneys in departments.items()
    for attorney in attorneys
)

for dept, name in all_attorneys:
    print(f"  {dept}: {name}")
```

---

### 3.9 File Processing with Generators ⭐

**Mike:** "Files are the #1 use case for generators — files can be gigabytes."

```python
def process_log_file(filepath):
    """Process a log file line-by-line using generators."""
    
    # Stage 1: Read lines (file objects are already lazy iterators!)
    with open(filepath, "r") as f:
        # Stage 2: Strip whitespace
        stripped = (line.strip() for line in f)
        
        # Stage 3: Skip empty/comment lines
        valid = (line for line in stripped if line and not line.startswith("#"))
        
        # Stage 4: Parse into dicts
        parsed = (
            {
                "timestamp": parts[0],
                "level": parts[1],
                "message": parts[2],
            }
            for line in valid
            for parts in [line.split(" | ")]
            if len(parts) >= 3
        )
        
        # Stage 5: Filter errors only
        errors = (entry for entry in parsed if entry["level"] == "ERROR")
        
        # Stage 6: Consume
        error_count = 0
        for error in errors:
            error_count += 1
            print(f"  ❌ {error['timestamp']}: {error['message']}")
        
        print(f"\n  Total errors: {error_count}")

# This processes a 10GB log file using ~200 bytes of memory
# Each line flows through the entire pipeline before the next is read
```

---

### 3.10 Performance Comparison

```python
import time
import sys

N = 5_000_000

# --- List Comprehension ---
start = time.time()
list_result = sum([x ** 2 for x in range(N)])
list_time = time.time() - start
list_mem = sys.getsizeof([x ** 2 for x in range(N)])

# --- Generator Expression ---
start = time.time()
gen_result = sum(x ** 2 for x in range(N))
gen_time = time.time() - start
gen_mem = sys.getsizeof(x ** 2 for x in range(N))

print(f"List:      {list_time:.3f}s | {list_mem:>12,} bytes | Result: {list_result}")
print(f"Generator: {gen_time:.3f}s  | {gen_mem:>12,} bytes | Result: {gen_result}")
print(f"\nMemory saved: {list_mem - gen_mem:,} bytes ({list_mem/gen_mem:.0f}x smaller)")
print(f"Same result: {list_result == gen_result}")

# Typical output:
# List:      0.850s | 40,448,056 bytes | Result: 41666664166667500000
# Generator: 0.680s |          112 bytes | Result: 41666664166667500000
# Memory saved: 40,447,944 bytes (361,143x smaller)
```

---

### 3.11 Solving Harvey's Problem — Streaming Analytics

```python
# ============================================
# Pearson Specter Litt — Streaming Analytics
# Zero-memory processing of billing records
# ============================================

import random
from datetime import datetime, timedelta

def generate_billing_records(count):
    """Simulate generating millions of billing records (generator)."""
    attorneys = ["Harvey", "Mike", "Louis", "Katrina", "Rachel"]
    types = ["Merger", "Patent", "Litigation", "Corporate"]
    
    base_date = datetime(2025, 1, 1)
    
    for i in range(count):
        yield {
            "id": f"INV-{i+1:07d}",
            "date": (base_date + timedelta(days=random.randint(0, 365))).strftime("%Y-%m-%d"),
            "attorney": random.choice(attorneys),
            "type": random.choice(types),
            "hours": round(random.uniform(0.5, 12.0), 1),
            "rate": random.choice([350, 375, 425, 450, 500]),
            "paid": random.random() > 0.15,
        }

def streaming_analytics(record_count=100_000):
    """Process records through a generator pipeline."""
    
    print(f"📊 Processing {record_count:,} billing records...")
    print(f"   (Using generator pipeline — near-zero memory)\n")
    
    # Stage 1: Generate records (lazy — nothing stored)
    records = generate_billing_records(record_count)
    
    # Stage 2: Compute billing amounts
    billed = (
        {**r, "amount": r["hours"] * r["rate"]}
        for r in records
    )
    
    # Process in a single pass — collecting stats
    total_amount = 0
    paid_amount = 0
    unpaid_amount = 0
    count = 0
    by_attorney = {}
    by_type = {}
    
    for record in billed:
        count += 1
        total_amount += record["amount"]
        
        if record["paid"]:
            paid_amount += record["amount"]
        else:
            unpaid_amount += record["amount"]
        
        # Attorney stats
        atty = record["attorney"]
        if atty not in by_attorney:
            by_attorney[atty] = {"hours": 0, "amount": 0, "count": 0}
        by_attorney[atty]["hours"] += record["hours"]
        by_attorney[atty]["amount"] += record["amount"]
        by_attorney[atty]["count"] += 1
        
        # Type stats
        ct = record["type"]
        if ct not in by_type:
            by_type[ct] = {"amount": 0, "count": 0}
        by_type[ct]["amount"] += record["amount"]
        by_type[ct]["count"] += 1
    
    # Display results
    print(f"{'='*60}")
    print(f"{'STREAMING ANALYTICS REPORT':^60}")
    print(f"{'='*60}")
    
    print(f"\n💰 BILLING SUMMARY")
    print(f"  Records Processed: {count:>12,}")
    print(f"  Total Billed:      ${total_amount:>14,.2f}")
    print(f"  Paid:              ${paid_amount:>14,.2f}")
    print(f"  Outstanding:       ${unpaid_amount:>14,.2f}")
    print(f"  Collection Rate:   {paid_amount/total_amount*100:>13.1f}%")
    
    print(f"\n👔 BY ATTORNEY")
    print(f"  {'Name':<12} {'Records':>8} {'Hours':>10} {'Billed':>14}")
    print(f"  {'─'*48}")
    for name in sorted(by_attorney, key=lambda a: by_attorney[a]["amount"], reverse=True):
        s = by_attorney[name]
        print(f"  {name:<12} {s['count']:>8,} {s['hours']:>10,.1f} ${s['amount']:>13,.2f}")
    
    print(f"\n💼 BY CASE TYPE")
    for ct in sorted(by_type, key=lambda t: by_type[t]["amount"], reverse=True):
        s = by_type[ct]
        pct = s["amount"] / total_amount * 100
        bar = "█" * int(pct / 2.5) + "░" * (40 - int(pct / 2.5))
        print(f"  {ct:<12} [{bar}] {pct:5.1f}%")
    
    print(f"\n{'='*60}")
    print(f"  ✅ Processed {count:,} records in a single pass")
    print(f"  ✅ Memory usage: ~constant (generator pipeline)")
    print(f"{'='*60}")

# Run it
streaming_analytics(100_000)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Generators Are One-Time Use! 🔥🔥🔥

```python
gen = (x ** 2 for x in range(5))

# First consumption — works
print(list(gen))     # [0, 1, 4, 9, 16]

# Second consumption — EMPTY!
print(list(gen))     # [] ← All consumed!
print(sum(gen))      # 0 ← Nothing left!

# ✅ Fix: Create a new generator each time
def get_squares():
    return (x ** 2 for x in range(5))

print(list(get_squares()))  # [0, 1, 4, 9, 16]
print(list(get_squares()))  # [0, 1, 4, 9, 16] — fresh generator!

# Or: Convert to list if you need multiple passes
squares = list(x ** 2 for x in range(5))
print(sum(squares))     # 30
print(max(squares))     # 16 — still works!
```

---

### Pitfall 2: `len()` and Indexing Don't Work

```python
gen = (x for x in range(10))

# ❌ These all fail
# len(gen)    # TypeError: object of type 'generator' has no len()
# gen[5]      # TypeError: 'generator' object is not subscriptable
# gen[-1]     # TypeError

# ✅ If you need length or indexing, use a list
data = list(gen)
print(len(data))    # 10
print(data[5])      # 5
```

---

### Pitfall 3: `in` Membership Test Consumes Elements

```python
gen = (x for x in range(10))

print(5 in gen)     # True — found it!
print(3 in gen)     # False?! Wait...

# What happened? Checking "5 in gen" consumed elements 0-5.
# Now the generator starts at 6. So 3 is already gone!
print(list(gen))    # [6, 7, 8, 9] — only the remaining elements!
```

---

### Pitfall 4: Generators with Side Effects Are Deferred

```python
# ⚠️ Side effects don't happen until you consume the generator!
files = ["data1.csv", "data2.csv", "data3.csv"]

# This generator DEFINES the processing but doesn't DO it yet
processed = (print(f"Processing {f}...") for f in files)

print("Generator created — but nothing is printed yet!")
# ...
# Now consume it:
list(processed)
# NOW it prints:
# Processing data1.csv...
# Processing data2.csv...
# Processing data3.csv...
```

---

### Pitfall 5: Late Binding in Generator Closures

```python
# ⚠️ Generators use late binding for variables
funcs = (lambda: x for x in range(5))
print([f() for f in funcs])    # [4, 4, 4, 4, 4] — all 4!

# Why? All lambdas reference the SAME variable x,
# which equals 4 by the time the lambdas are called

# ✅ Fix: capture x with a default argument
funcs = (lambda x=x: x for x in range(5))
print([f() for f in funcs])    # [0, 1, 2, 3, 4] ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏭 Analogy: Generator = Assembly Line vs Warehouse

```
LIST COMPREHENSION = Warehouse Model:
  🏭 Factory produces ALL items
  📦📦📦📦📦📦📦📦📦📦 → Stack them ALL in the warehouse
  🏗️ Now process them from the warehouse
  💰 Cost: rent a HUGE warehouse (RAM)

GENERATOR EXPRESSION = Assembly Line Model:
  🏭 Factory produces ONE item
  📦 → Process it immediately → 🚚 Ship it
  🏭 Factory produces NEXT item
  📦 → Process it → 🚚 Ship it
  💰 Cost: one tiny workstation (almost no RAM)
```

**Donna:** "A list is like cooking all 50 dishes for a banquet and keeping them warm on the counter. A generator is like cooking each dish fresh as guests order it — one at a time, served immediately."

| If You Need... | Use |
|:---|:---|
| Random access (`items[5]`) | List |
| Multiple passes over data | List |
| Know the length upfront | List |
| Process once, forward only | **Generator** |
| Data that won't fit in memory | **Generator** |
| Pipeline of transformations | **Generator** |
| Feed into `sum/any/all/max/min` | **Generator** |

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Generator Basics**
```
a) Create a generator that yields squares of 1 to 20
b) Print the first 5 values using next()
c) Consume the rest with a for loop
d) Try to consume it again — what happens?
```

**Exercise 2: Memory Comparison**
```
Compare memory usage for N = 1,000,000:
a) List comprehension: [x**2 for x in range(N)]
b) Generator expression: (x**2 for x in range(N))

Use sys.getsizeof() to measure.
Print the ratio.
```

**Exercise 3: Built-In Integration**
```
Given: numbers = range(1, 101)

Use generator expressions with:
a) sum() — total of all even numbers
b) max() — largest number divisible by 7
c) any() — is any number > 50 also divisible by 13?
d) all() — are all numbers positive?
```

---

### 🟡 Intermediate Exercises

**Exercise 4: File Pipeline**
```
Create a text file with 20+ lines of data (name,score,grade).
Build a generator pipeline:
  Stage 1: Read lines (file is already an iterator)
  Stage 2: Strip whitespace
  Stage 3: Skip comments and blank lines
  Stage 4: Parse into dicts
  Stage 5: Filter scores above 80
  Stage 6: Format as report strings

Print the final output.
```

**Exercise 5: Infinite Counter**
```
Create a generator expression that represents:
  a) All even numbers (no upper bound)
  b) Powers of 2 (2, 4, 8, 16, ...)
  c) Fibonacci-like: use itertools or a generator function

Use itertools.islice to take the first 20 from each.
(Hint: use itertools.count() for infinite sequences)
```

**Exercise 6: Multi-Stage Filter**
```
Given a list of 1000 random integers (1-10000):

Build a generator pipeline that in ONE PASS:
  1. Squares each number
  2. Filters out values below 1000
  3. Keeps only even results
  4. Takes only the first 50

Calculate the sum of those 50 numbers.
Use no intermediate lists.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Streaming Word Counter**
```
Build a streaming word frequency counter:
  1. Read a file line by line (generator)
  2. Split each line into words (generator)
  3. Normalize to lowercase (generator)
  4. Count with a single-pass accumulator

Process a 1000+ line file without loading it all.
Compare memory with a list-based approach.
```

**Exercise 8: Generator-Based ETL**
```
Build an ETL (Extract, Transform, Load) pipeline:
  Extract: Generator that reads CSV records
  Transform: Generator chain (validate → enrich → format)
  Load: Single-pass consumer that writes to output file

Process 10,000+ records. Measure memory.
Show that the pipeline uses constant memory.
```

**Exercise 9: Lazy Database Query Simulator**
```
Build a LazyQuery class that chains generator operations:

q = LazyQuery(data)
q = q.where(lambda r: r["revenue"] > 500000)
q = q.select("name", "revenue")
q = q.order_by("revenue", desc=True)
q = q.limit(5)

results = q.execute()   # Only NOW does computation happen

All intermediate steps should be generators.
execute() materializes the final result.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Generator exhaustion
gen = (x * 10 for x in range(5))
total = sum(gen)
avg = total / len(gen)    # Error!

# Bug 2: Trying to reuse a generator
data_gen = (x for x in range(10))
first_five = list(data_gen)[:5]
rest = list(data_gen)
print(f"First: {first_five}, Rest: {rest}")   # Rest is empty?

# Bug 3: Subscripting a generator
squares = (x**2 for x in range(10))
third = squares[2]    # Error!

# Bug 4: Generator in boolean context
gen = (x for x in [])
if gen:   # Is this True or False?
    print("Has items")
else:
    print("Empty")

# Bug 5: Deferred evaluation surprise
values = [1, 2, 3]
gen = (x for x in values)
values.append(4)
print(list(gen))    # What does this print?
```

### Fixed Code ✅

```python
# Fix 1: Materialize to list if you need both sum and len
data = list(x * 10 for x in range(5))
total = sum(data)
avg = total / len(data)    # ✅ Works on list
print(f"Total: {total}, Avg: {avg}")

# Fix 2: Create a new generator for each use
def get_data():
    return (x for x in range(10))

first_five = list(get_data())[:5]
rest = list(get_data())[5:]     # ✅ Fresh generator
print(f"First: {first_five}, Rest: {rest}")

# Fix 3: Convert to list for indexing
squares = list(x**2 for x in range(10))
third = squares[2]    # ✅ 4
print(f"Third square: {third}")

# Fix 4: Generators are ALWAYS truthy (even if empty!)
gen = (x for x in [])
# if gen:  ← This is ALWAYS True! 
# ✅ Convert to list to check, or use try/next
items = list(gen)
if items:
    print("Has items")
else:
    print("Empty")    # ✅ Correctly detects empty

# Fix 5: Generators capture the reference, not the value
# This is expected — gen sees the modified list
values = [1, 2, 3]
gen = (x for x in values)
values.append(4)
print(list(gen))    # [1, 2, 3, 4] — includes 4!
# If you want a snapshot, use a list comp: [x for x in values]
```

---

## 8. 🌍 Real-World Application

### Where Generators Are Used in Production

| Library/Tool | Generator Usage |
|-------------|----------------|
| **File I/O** | Files are iterators — line-by-line reading |
| **Django ORM** | `.iterator()` returns a generator for large querysets |
| **pandas** | `pd.read_csv(chunksize=1000)` returns a generator of DataFrames |
| **requests** | `response.iter_lines()` / `response.iter_content()` streams data |
| **logging** | Log processors stream through millions of entries |
| **ETL pipelines** | Extract → Transform → Load, all as generators |

### Production Example: CSV Stream Processor

```python
import csv
from io import StringIO

def stream_csv(filepath_or_data, *, delimiter=","):
    """Stream CSV records as dicts — constant memory."""
    if isinstance(filepath_or_data, str) and "\n" in filepath_or_data:
        source = StringIO(filepath_or_data)
    else:
        source = open(filepath_or_data, "r")
    
    try:
        reader = csv.DictReader(source, delimiter=delimiter)
        for row in reader:
            yield row
    finally:
        source.close()

def pipeline(records):
    """Transform pipeline — all generators."""
    # Stage 1: Type conversion
    typed = (
        {**r, "revenue": float(r.get("revenue", 0)), 
              "hours": float(r.get("hours", 0))}
        for r in records
    )
    
    # Stage 2: Enrich
    enriched = (
        {**r, "billing": r["revenue"] * r["hours"],
              "tier": "Platinum" if r["revenue"] >= 1000000 else "Gold"}
        for r in typed
    )
    
    # Stage 3: Filter
    return (r for r in enriched if r["billing"] > 0)

# Usage — constant memory regardless of file size
csv_data = """name,revenue,hours
Wayne,2500000,85
Stark,800000,62
Oscorp,600000,55"""

for record in pipeline(stream_csv(csv_data)):
    print(f"  {record['name']}: {record['tier']} — ${record['billing']:,.0f}")
```

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the syntax difference between a list comprehension and a generator expression?

**Q2.** How much memory does `(x for x in range(10_000_000))` use compared to `[x for x in range(10_000_000)]`?

**Q3.** What happens when you try to iterate over a generator a second time?

**Q4.** Can you use `len()` on a generator? Why or why not?

**Q5.** Why do generators work well with `sum()`, `any()`, `all()`?

**Q6.** What is a generator pipeline?

**Q7.** What is the output?
```python
gen = (x for x in range(5))
print(next(gen))
print(next(gen))
print(list(gen))
```

**Q8.** Why is a generator expression ALWAYS truthy even when empty?

**Q9.** When should you prefer a list comprehension over a generator?

**Q10.** Write a generator pipeline that reads lines from a list, strips whitespace, filters empty lines, and converts to uppercase.

---

### 📋 Answer Key

> **Q1.** List: `[expr for x in items]` (brackets). Generator: `(expr for x in items)` (parentheses).

> **Q2.** Generator: ~200 bytes. List: ~80–400 MB. Generators use constant memory regardless of size.

> **Q3.** Nothing — it's exhausted. Returns no items.

> **Q4.** No. Generators don't know their length ahead of time — they produce values lazily.

> **Q5.** These functions consume iterators one item at a time, and generators produce one item at a time. No intermediate list needed. `any()`/`all()` can also short-circuit.

> **Q6.** Multiple generators chained together — output of one feeds into the next. Each record flows through the entire pipeline before the next is started.

> **Q7.** `0`, `1`, `[2, 3, 4]` — `next()` consumes values, `list()` gets the rest.

> **Q8.** A generator object exists (it's a real object) — Python checks if the object is `None`, not whether it still has values. Use `list()` or `next()` with `StopIteration` to check for emptiness.

> **Q9.** When you need: random access, `len()`, multiple passes, slicing, or the data fits in memory and speed is more important.

> **Q10.**
```python
lines = ["  Hello  ", "", "  World  ", "  ", "  Python  "]
result = (
    line.upper()
    for line in (l.strip() for l in lines)
    if line
)
# Or chained:
stripped = (line.strip() for line in lines)
non_empty = (line for line in stripped if line)
uppered = (line.upper() for line in non_empty)
```

---

## 🎬 End of Lesson Scene

*Harvey watches the analytics dashboard process 100,000 records in under a second, using barely any memory.*

**Harvey:** "3.2 GB down to nothing. Same results. Faster."

**Mike:** "Generators are the reason Python can process datasets that don't fit in RAM. Netflix, Instagram, Spotify — they all use generator pipelines for their data processing."

**Harvey:** "Next lesson: choosing the right data structure. Lists, dicts, or sets — and when each one wins."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Generator expression syntax — `()` vs `[]` | ✅ |
| 2 | Memory efficiency — O(1) vs O(n) | ✅ |
| 3 | Lazy evaluation — compute on demand | ✅ |
| 4 | Generators with built-in functions | ✅ |
| 5 | Generator pipelines — chained stages | ✅ |
| 6 | One-time consumption (exhaustion) | ✅ |
| 7 | No len/index/membership on generators | ✅ |
| 8 | File processing with generators | ✅ |
| 9 | Performance comparison | ✅ |
| 10 | When to use generators vs lists | ✅ |

---

> **Next Lesson →** Level 2, Lesson 6: *Choose Lists, Dicts, or Sets in Python*
