# 🏛️ Level 3 – Building | Lesson 1: Create Generator Functions in Python

## *"The Assembly Line"*

> **O'Reilly Skill Objective:** *Create generator functions using `yield` to produce values lazily.*

> **Previously Learned:** Functions, `*args`/`**kwargs`, comprehensions, generator expressions (Level 2 Lesson 5), itertools, loops, classes

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

*Monday morning. The firm's data team has a crisis.*

**Mike:** "Harvey, the compliance team needs us to scan every contract in the database — 4 million documents. I wrote a function that loads them all into a list, processes them, and returns results."

**Harvey:** "And?"

**Mike:** "Memory spike: 12 GB. The server crashed."

**Jessica:** "Find a way to process 4 million documents without crashing the server."

**Mike:** "In Level 2, we learned generator EXPRESSIONS — `(x for x in data)`. Those produce one item at a time. But they're limited to a single expression. Generator FUNCTIONS give us the full power of `yield` — loops, conditionals, state, multiple yields, infinite sequences, and coroutine-like pipelines. Same memory efficiency, unlimited flexibility."

```python
# ❌ What crashed the server:
def get_all_contracts():
    results = []
    for doc in database.scan_all():    # 4M documents
        results.append(process(doc))    # All 4M in RAM!
    return results

# ✅ What Mike built instead:
def get_all_contracts():
    for doc in database.scan_all():
        yield process(doc)              # One at a time — O(1) memory!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Level 2 gave you generator expressions — the assembly line analogy. Generator functions are the FACTORY that builds the assembly line."

| Feature | Generator Expression | Generator Function |
|---------|:---:|:---:|
| Syntax | `(expr for x in iterable)` | `def func(): yield value` |
| Complexity | Single expression | Any Python logic |
| Multiple yields | ❌ | ✅ Multiple `yield` statements |
| State between yields | ❌ | ✅ Local variables persist |
| Parameters | ❌ | ✅ Accept arguments |
| Infinite sequences | Limited | ✅ Natural |
| `send()` / `throw()` | ❌ | ✅ Two-way communication |
| Reusable | ❌ (exhausted) | ✅ Call again = new generator |

**Harvey:** "Generator expressions are for one-liners. Generator functions are for everything else."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Your First Generator Function ⭐

```python
# A regular function uses 'return':
def get_partners():
    return ["Jessica", "Harvey", "Louis"]

# A generator function uses 'yield':
def get_partners_gen():
    yield "Jessica"
    yield "Harvey"
    yield "Louis"

# Calling a generator function returns a GENERATOR OBJECT (not the values!)
gen = get_partners_gen()
print(type(gen))    # <class 'generator'>

# Values are produced ONE AT A TIME with next()
print(next(gen))    # Jessica
print(next(gen))    # Harvey
print(next(gen))    # Louis
# print(next(gen))  # StopIteration!

# Or use in a for loop (handles StopIteration automatically)
for partner in get_partners_gen():
    print(partner)
```

**Mike:** "Here's the key insight: when a generator function hits `yield`, execution FREEZES. The value is sent out. When `next()` is called again, execution RESUMES exactly where it left off — all local variables are preserved."

---

### 3.2 How `yield` Works — Freeze and Resume ⭐⭐

```python
def countdown(n):
    print(f"Starting countdown from {n}")
    while n > 0:
        print(f"  About to yield {n}")
        yield n                          # ← FREEZE here
        print(f"  Resumed after yielding {n}")
        n -= 1
    print("Blastoff!")

gen = countdown(3)

print("--- Calling next() #1 ---")
val = next(gen)
print(f"Got: {val}\n")

print("--- Calling next() #2 ---")
val = next(gen)
print(f"Got: {val}\n")

print("--- Calling next() #3 ---")
val = next(gen)
print(f"Got: {val}\n")

print("--- Calling next() #4 ---")
try:
    next(gen)
except StopIteration:
    print("Generator exhausted!")

# Output:
# --- Calling next() #1 ---
# Starting countdown from 3
#   About to yield 3
# Got: 3
# 
# --- Calling next() #2 ---
#   Resumed after yielding 3
#   About to yield 2
# Got: 2
# 
# --- Calling next() #3 ---
#   Resumed after yielding 2
#   About to yield 1
# Got: 1
# 
# --- Calling next() #4 ---
#   Resumed after yielding 1
# Blastoff!
# Generator exhausted!
```

---

### 3.3 Generators with Parameters and Logic

```python
def fibonacci(limit=None):
    """Generate Fibonacci numbers (optionally up to a limit)."""
    a, b = 0, 1
    while True:
        if limit is not None and a > limit:
            return    # Return without value = StopIteration
        yield a
        a, b = b, a + b

# First 10 Fibonacci numbers
from itertools import islice
print(list(islice(fibonacci(), 10)))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Fibonacci up to 100
print(list(fibonacci(limit=100)))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

def filter_cases(cases, min_revenue=0, case_types=None):
    """Generator that filters cases on the fly."""
    for case in cases:
        if case["revenue"] < min_revenue:
            continue
        if case_types and case["type"] not in case_types:
            continue
        yield case

cases = [
    {"client": "Wayne", "revenue": 2500000, "type": "Merger"},
    {"client": "Stark", "revenue": 800000, "type": "Patent"},
    {"client": "Oscorp", "revenue": 600000, "type": "Litigation"},
    {"client": "Dalton", "revenue": 1200000, "type": "Merger"},
    {"client": "Luthor", "revenue": 150000, "type": "Corporate"},
]

big_mergers = filter_cases(cases, min_revenue=1000000, case_types={"Merger"})
for case in big_mergers:
    print(f"  {case['client']}: ${case['revenue']:,}")
# Wayne: $2,500,000
# Dalton: $1,200,000
```

---

### 3.4 `yield` vs `return` — The Critical Difference

```python
# return: terminates the function, sends back ONE value
def get_squares_list(n):
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result               # All at once, entire list in memory

# yield: pauses the function, sends back values ONE AT A TIME
def get_squares_gen(n):
    for i in range(n):
        yield i ** 2            # One at a time, O(1) memory

# Both produce the same values:
print(list(get_squares_gen(5)))    # [0, 1, 4, 9, 16]
print(get_squares_list(5))          # [0, 1, 4, 9, 16]

# But with 10 million items, the generator barely uses memory:
import sys
squares_list = get_squares_list(10_000_000)      # ~80 MB
squares_gen = get_squares_gen(10_000_000)         # ~200 bytes!

print(f"List: {sys.getsizeof(squares_list):,} bytes")
print(f"Generator: {sys.getsizeof(squares_gen):,} bytes")
```

---

### 3.5 Multiple `yield` Statements — State Machine ⭐

```python
def case_lifecycle():
    """Model a case's lifecycle as a generator."""
    print("📁 Case FILED")
    yield "filed"
    
    print("🔍 Case in DISCOVERY")
    yield "discovery"
    
    print("📋 Taking DEPOSITIONS")
    yield "depositions"
    
    print("⚖️ Going to TRIAL")
    yield "trial"
    
    print("🏁 Case CLOSED")
    yield "closed"

# Step through the lifecycle
case = case_lifecycle()
for status in case:
    print(f"  Status: {status}")
    input("  Press Enter for next phase...")  # Pauses between yields
```

```python
def alternating_pattern(*sequences):
    """Yield items from multiple sequences in alternating order."""
    iterators = [iter(seq) for seq in sequences]
    while iterators:
        exhausted = []
        for i, it in enumerate(iterators):
            try:
                yield next(it)
            except StopIteration:
                exhausted.append(i)
        for i in reversed(exhausted):
            iterators.pop(i)

# Interleave attorneys and clients
print(list(alternating_pattern(
    ["Harvey", "Mike", "Louis"],
    ["Wayne", "Stark", "Oscorp", "Dalton"]
)))
# ['Harvey', 'Wayne', 'Mike', 'Stark', 'Louis', 'Oscorp', 'Dalton']
```

---

### 3.6 `yield from` — Delegation ⭐

```python
# Without yield from:
def all_attorneys():
    for name in get_partners():
        yield name
    for name in get_associates():
        yield name
    for name in get_paralegals():
        yield name

# With yield from — cleaner!
def all_attorneys():
    yield from get_partners()
    yield from get_associates()
    yield from get_paralegals()

# yield from works with any iterable:
def flatten(nested):
    """Flatten a nested list using yield from."""
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)    # Recursion + delegation!
        else:
            yield item

data = [1, [2, 3, [4, 5]], 6, [7, [8, 9]]]
print(list(flatten(data)))
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

### 3.7 Generator Pipelines — Data Processing Chains ⭐⭐

```python
def read_records(filepath):
    """Stage 1: Read lines from a file lazily."""
    with open(filepath) as f:
        for line in f:
            yield line.strip()

def parse_records(lines):
    """Stage 2: Parse CSV lines into dicts."""
    headers = None
    for line in lines:
        fields = line.split(",")
        if headers is None:
            headers = fields
        else:
            yield dict(zip(headers, fields))

def filter_active(records):
    """Stage 3: Keep only active records."""
    for record in records:
        if record.get("status") == "active":
            yield record

def enrich(records):
    """Stage 4: Add computed fields."""
    for record in records:
        record["revenue"] = float(record.get("revenue", 0))
        record["tier"] = "Platinum" if record["revenue"] >= 1000000 else "Standard"
        yield record

def format_output(records):
    """Stage 5: Format for display."""
    for record in records:
        yield f"{record['name']:<20} ${record['revenue']:>12,.2f}  {record['tier']}"

# Connect the pipeline — NOTHING executes until consumed!
# pipeline = format_output(enrich(filter_active(parse_records(read_records("clients.csv")))))
# for line in pipeline:
#     print(line)

# Each record flows through ALL stages one at a time.
# Only ONE record is in memory at any moment!
```

---

### 3.8 `send()` — Two-Way Communication ⭐⭐

```python
def adjustable_counter(start=0, step=1):
    """Counter that can receive new step values via send()."""
    value = start
    while True:
        new_step = yield value    # Yield current value, receive new step
        if new_step is not None:
            step = new_step
        value += step

counter = adjustable_counter(0, 1)

# Must prime the generator with next() first!
print(next(counter))          # 0
print(next(counter))          # 1
print(next(counter))          # 2
print(counter.send(10))       # 12 (step changed to 10!)
print(next(counter))          # 22
print(counter.send(-5))       # 17 (step changed to -5!)
print(next(counter))          # 12
```

```python
def running_average():
    """Calculate running average — receives values via send()."""
    total = 0
    count = 0
    average = None
    
    while True:
        value = yield average        # Send out current avg, receive new value
        if value is not None:
            total += value
            count += 1
            average = total / count

avg = running_average()
next(avg)                    # Prime the generator

print(avg.send(100))         # 100.0
print(avg.send(200))         # 150.0
print(avg.send(300))         # 200.0
print(avg.send(50))          # 162.5
```

---

### 3.9 `throw()` and `close()` — Error Handling

```python
def resilient_processor():
    """Generator that can handle injected exceptions."""
    while True:
        try:
            value = yield
            print(f"  Processing: {value}")
        except ValueError as e:
            print(f"  ⚠️ Skipped bad value: {e}")
        except GeneratorExit:
            print("  🛑 Generator shutting down — cleanup here")
            return

proc = resilient_processor()
next(proc)                           # Prime

proc.send("Valid data")               # Processing: Valid data
proc.throw(ValueError, "bad input")   # ⚠️ Skipped bad value: bad input
proc.send("More valid data")          # Processing: More valid data
proc.close()                          # 🛑 Generator shutting down
```

---

### 3.10 Infinite Generators

```python
def unique_ids(prefix="PSL"):
    """Generate unique IDs forever."""
    counter = 1
    while True:
        yield f"{prefix}-{counter:06d}"
        counter += 1

id_gen = unique_ids()
for _ in range(5):
    print(next(id_gen))
# PSL-000001
# PSL-000002
# PSL-000003
# PSL-000004
# PSL-000005

# Use with islice to take a specific number
from itertools import islice
batch = list(islice(unique_ids("CASE"), 100))
print(f"Generated {len(batch)} IDs: {batch[0]} to {batch[-1]}")

def clock(interval=1):
    """Infinite clock — yields current time at intervals."""
    import time
    from datetime import datetime
    while True:
        yield datetime.now().strftime("%H:%M:%S")
        time.sleep(interval)

# for t in clock():
#     print(f"\r⏰ {t}", end="")    # Live clock
```

---

### 3.11 Solving Jessica's Problem — Document Scanner

```python
# ============================================
# Pearson Specter Litt — Streaming Document Scanner
# Generator functions for memory-efficient processing
# ============================================

import time
import random
from itertools import islice
from collections import Counter

def generate_mock_documents(n):
    """Simulate a database of n documents (generator)."""
    types = ["Contract", "Brief", "Motion", "Deposition", "NDA"]
    statuses = ["active", "archived", "pending", "expired"]
    attorneys = ["Harvey", "Mike", "Louis", "Jessica", "Rachel"]
    
    for i in range(n):
        yield {
            "id": f"DOC-{i+1:07d}",
            "type": random.choice(types),
            "status": random.choice(statuses),
            "attorney": random.choice(attorneys),
            "pages": random.randint(1, 200),
            "flagged": random.random() < 0.05,   # 5% flagged
        }

def scan_for_compliance(documents):
    """Stage 1: Check compliance flags."""
    flagged_count = 0
    for doc in documents:
        if doc["flagged"]:
            flagged_count += 1
            doc["compliance_note"] = f"FLAGGED (#{flagged_count})"
            yield doc

def enrich_with_priority(documents):
    """Stage 2: Assign priority based on type."""
    priority_map = {
        "Contract": 1, "NDA": 2, "Motion": 3,
        "Brief": 4, "Deposition": 5
    }
    for doc in documents:
        doc["priority"] = priority_map.get(doc["type"], 99)
        yield doc

def batch(iterable, size):
    """Utility: yield items in fixed-size batches."""
    it = iter(iterable)
    while True:
        batch_items = list(islice(it, size))
        if not batch_items:
            return
        yield batch_items

# === Run the scanner ===
print("=" * 60)
print(f"{'🏛️  DOCUMENT COMPLIANCE SCANNER':^60}")
print("=" * 60)

DOC_COUNT = 100_000    # 100K documents

# Build the pipeline — nothing executes yet!
all_docs = generate_mock_documents(DOC_COUNT)
flagged = scan_for_compliance(all_docs)
prioritized = enrich_with_priority(flagged)

# Process in batches of 100
start = time.time()
total_flagged = 0
type_counter = Counter()
attorney_counter = Counter()

print(f"\n📡 Scanning {DOC_COUNT:,} documents...\n")

for batch_num, batch_items in enumerate(batch(prioritized, 100), 1):
    for doc in batch_items:
        total_flagged += 1
        type_counter[doc["type"]] += 1
        attorney_counter[doc["attorney"]] += 1
    
    # Progress update every 10 batches
    if batch_num % 10 == 0:
        print(f"  Batch {batch_num}: {total_flagged} flagged so far...")

elapsed = time.time() - start

# Results
print(f"\n{'='*60}")
print(f"{'SCAN RESULTS':^60}")
print(f"{'='*60}")
print(f"\n  Documents scanned: {DOC_COUNT:,}")
print(f"  Flagged: {total_flagged:,} ({total_flagged/DOC_COUNT*100:.1f}%)")
print(f"  Time: {elapsed:.2f}s")
print(f"  Memory: ~constant (streaming pipeline)")

print(f"\n  📊 Flagged by Type:")
for doc_type, count in type_counter.most_common():
    bar = "█" * (count // max(1, max(type_counter.values()) // 20))
    print(f"    {doc_type:<12} [{bar:<20}] {count}")

print(f"\n  👤 Flagged by Attorney:")
for atty, count in attorney_counter.most_common():
    bar = "█" * (count // max(1, max(attorney_counter.values()) // 20))
    print(f"    {atty:<12} [{bar:<20}] {count}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Generators Are One-Shot — Cannot Reuse! 🔥

```python
def squares(n):
    for i in range(n):
        yield i ** 2

gen = squares(5)
print(list(gen))    # [0, 1, 4, 9, 16]
print(list(gen))    # [] ← EMPTY! Generator exhausted!

# ✅ Fix: Call the function again to get a NEW generator
print(list(squares(5)))    # [0, 1, 4, 9, 16] — fresh!
```

---

### Pitfall 2: Forgetting to Prime `send()` Generators

```python
def receiver():
    while True:
        value = yield
        print(f"Got: {value}")

gen = receiver()
# gen.send("hello")   # TypeError: can't send to a just-started generator!

# ✅ Must prime with next() first!
next(gen)           # Advance to first yield
gen.send("hello")   # Now it works: Got: hello
```

---

### Pitfall 3: `return` in a Generator Doesn't Return a Value Normally

```python
def gen_with_return():
    yield 1
    yield 2
    return "done"    # Sets the StopIteration value, doesn't return directly!

g = gen_with_return()
print(next(g))    # 1
print(next(g))    # 2
try:
    next(g)
except StopIteration as e:
    print(f"Stopped with value: {e.value}")   # "done"

# The return value is INSIDE the StopIteration exception!
# yield from captures it:
def wrapper():
    result = yield from gen_with_return()
    print(f"Inner generator returned: {result}")    # "done"
```

---

### Pitfall 4: Generator Inside a List Comprehension

```python
# ❌ This creates a list (not lazy!)
result = [x for x in expensive_generator()]    # All in memory!

# ✅ Keep it as a generator (lazy)
result = (x for x in expensive_generator())    # Stays lazy

# ✅ Or consume lazily
for x in expensive_generator():
    process(x)
```

---

### Pitfall 5: Resource Cleanup in Generators

```python
# ❌ File might not close if generator isn't fully consumed
def read_lines(path):
    f = open(path)
    for line in f:
        yield line.strip()
    f.close()           # Only runs if generator is fully consumed!

# ✅ Use try/finally or context manager
def read_lines(path):
    try:
        f = open(path)
        for line in f:
            yield line.strip()
    finally:
        f.close()       # Always runs, even if generator is abandoned

# ✅ Even better: use with statement
def read_lines(path):
    with open(path) as f:
        for line in f:
            yield line.strip()
    # File is closed when the with block exits
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### ⚙️ Analogy: Generator Functions = Custom Assembly Lines

```
REGULAR FUNCTION (warehouse):
  🏭 Build ALL products → 📦 Ship entire warehouse
  return [item1, item2, item3, ..., item1000000]
  💥 Warehouse explodes if too many products!

GENERATOR EXPRESSION (simple conveyor belt):
  🔧 (process(x) for x in raw_materials)
  Limited to ONE operation per item

GENERATOR FUNCTION (full factory):
  🏭 def factory():
      setup_machinery()        ← Complex setup
      while raw_materials:
          item = get_next()
          if quality_check(item):
              ← yield item     ← Send ONE product, PAUSE
          log(item)             ← Resume here next time
      cleanup()                 ← Cleanup when done
  
  ✅ Complex logic, state, multiple yield points
  ✅ O(1) memory regardless of output size
  ✅ Can be paused, resumed, and stopped
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Basic Generator**
```
Write a generator function: evens(start, stop)
that yields even numbers in the range.

for n in evens(1, 20):
    print(n)   # 2, 4, 6, 8, 10, 12, 14, 16, 18, 20
```

**Exercise 2: File Line Generator**
```
Write: read_nonblank(filepath) that yields non-blank,
non-comment lines from a text file (strip whitespace,
skip lines starting with #).
```

**Exercise 3: Chunked Generator**
```
Write: chunks(iterable, size) that groups items into
fixed-size chunks.

list(chunks(range(10), 3))
# [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9]]
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Sliding Window**
```
Write: sliding_window(iterable, size) that yields
overlapping windows.

list(sliding_window([1,2,3,4,5], 3))
# [(1,2,3), (2,3,4), (3,4,5)]
```

**Exercise 5: Tree Traversal**
```
Build a tree structure and write generators for:
  a) preorder(node) — yield node, then children
  b) postorder(node) — yield children, then node
  c) breadth_first(node) — level by level

Use yield from for recursion.
```

**Exercise 6: Pipeline Builder**
```
Build a Pipeline class:

p = (Pipeline()
     .map(lambda x: x * 2)
     .filter(lambda x: x > 5)
     .take(10))

result = list(p.run(range(100)))

Each stage is a generator. Chain them lazily.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Coroutine-Style Chat**
```
Build a chat system using send():
  - bot = chatbot()
  - response = bot.send("What cases are open?")
  - The generator maintains conversation state
  - Different responses based on context
```

**Exercise 8: Infinite Data Structures**
```
Implement:
  a) primes() — infinite prime generator (Sieve of Eratosthenes)
  b) merge_sorted(*generators) — merge infinite sorted generators
  c) look_and_say() — the look-and-say sequence (infinite)
```

**Exercise 9: Streaming ETL Framework**
```
Build an ETL framework:
  Source: reads from file/database/API (generator)
  Transform: chain of generator stages
  Load: writes to file/database

@source
def csv_reader(path): ...

@transform  
def parse_dates(records): ...

@transform
def filter_active(records): ...

@sink
def json_writer(path, records): ...

etl = csv_reader("data.csv") | parse_dates | filter_active | json_writer("out.json")
etl.run()
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Generator exhausted unexpectedly
def get_clients():
    yield "Wayne"
    yield "Stark"

clients = get_clients()
print(f"Count: {len(list(clients))}")   # 2
for c in clients:                        # Empty! Already consumed!
    print(c)

# Bug 2: Trying to index a generator
gen = (x**2 for x in range(10))
print(gen[3])    # TypeError!

# Bug 3: send() without priming
def echo():
    while True:
        val = yield
        print(val)

e = echo()
e.send("hello")   # TypeError!

# Bug 4: yield inside a nested function
def outer():
    def inner():
        yield 1    # This makes INNER a generator, not OUTER!
    inner()        # Returns a generator object, never iterated!
    return 2

# Bug 5: Generator with side effects assumed to run immediately
def log_everything(items):
    for item in items:
        print(f"Logging: {item}")
        yield item

logged = log_everything([1, 2, 3])   # Nothing prints yet!
```

### Fixed Code ✅

```python
# Fix 1: Call the generator function again for a fresh generator
def get_clients():
    yield "Wayne"
    yield "Stark"

print(f"Count: {len(list(get_clients()))}")   # 2
for c in get_clients():                        # Fresh generator!
    print(c)

# Fix 2: Generators don't support indexing — convert or use islice
gen = (x**2 for x in range(10))
from itertools import islice
print(list(islice(gen, 3, 4))[0])    # 9 ← 4th element (index 3)

# Fix 3: Prime with next() before send()
e = echo()
next(e)            # Prime!
e.send("hello")   # Now works ✅

# Fix 4: Use yield from or iterate the inner generator
def outer():
    def inner():
        yield 1
    yield from inner()   # ✅ Delegates to inner's yields

# Fix 5: Generators are lazy — must be consumed to execute!
logged = log_everything([1, 2, 3])
for item in logged:    # NOW the prints happen
    pass
```

---

## 8. 🌍 Real-World Application

### Where Generator Functions Shine

| Scenario | Why Generators |
|----------|---------------|
| **Large file processing** | Read line by line, constant memory |
| **Database cursors** | Fetch rows lazily |
| **API pagination** | Request next page only when needed |
| **Streaming data** | Process infinite event streams |
| **Pipeline architecture** | Composable data transformations |
| **Backpressure** | Consumer controls the pace |
| **State machines** | Natural multi-yield state transitions |
| **Testing** | Generate test data on demand |

---

## 9. ✅ Knowledge Check

---

**Q1.** What keyword makes a function into a generator function?

**Q2.** What does calling a generator function return?

**Q3.** How does `yield` differ from `return`?

**Q4.** What is the memory advantage of generators over lists?

**Q5.** What does `yield from` do?

**Q6.** Can a generator be iterated more than once?

**Q7.** What does `send()` do and what's required before using it?

**Q8.** How do you flatten a deeply nested structure with generators?

**Q9.** What happens when a generator function has a `return` statement?

**Q10.** How do generator pipelines achieve memory efficiency?

---

### 📋 Answer Key

> **Q1.** `yield`. Any function containing `yield` is a generator function.

> **Q2.** A generator object (an iterator). The function body does NOT execute yet.

> **Q3.** `return` terminates and sends back one value. `yield` pauses and sends back a value — execution resumes on the next `next()` call.

> **Q4.** O(1) — generators produce one item at a time regardless of total items. Lists store all items in memory (O(n)).

> **Q5.** Delegates iteration to another generator or iterable. Yields each item from the sub-generator.

> **Q6.** No. Once exhausted, calling `next()` raises `StopIteration`. Call the function again for a fresh generator.

> **Q7.** `send(value)` sends a value INTO the generator (received as the result of `yield`). You must call `next()` first to prime it.

> **Q8.** Recursive generator with `yield from`: `if isinstance(item, list): yield from flatten(item) else: yield item`.

> **Q9.** Triggers `StopIteration`. The return value is stored in `StopIteration.value` (captured by `yield from`).

> **Q10.** Each stage holds only one item at a time. Data flows through the pipeline one record at a time — never fully materialized.

---

## 🎬 End of Lesson Scene

**Jessica:** "4 million documents scanned. Memory usage: flat. Server: stable."

**Mike:** "Generator functions. `yield` freezes execution, sends one result. `next()` resumes. The pipeline processes one document at a time through every stage — scan, filter, enrich, report — and only ONE document is in memory at any moment."

**Harvey:** "Level 2 gave us generator expressions for one-liners. Generator functions give us the full factory."

**Jessica:** "Next: instance methods. How objects DO things, not just HOLD things."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `yield` keyword — freeze and resume | ✅ |
| 2 | Generator function vs regular function | ✅ |
| 3 | Generator objects and `next()` | ✅ |
| 4 | Parameters, loops, and conditionals with yield | ✅ |
| 5 | Multiple yield statements (state machines) | ✅ |
| 6 | `yield from` for delegation and flattening | ✅ |
| 7 | Generator pipelines | ✅ |
| 8 | `send()` — two-way communication | ✅ |
| 9 | `throw()` and `close()` | ✅ |
| 10 | Infinite generators with `islice` | ✅ |

---

> **Next Lesson →** Level 3, Lesson 2: *Define Instance Methods in Python*
