# 🏛️ Level 4 – Advancing | Lesson 7: Create Iterators with `__iter__` and `__next__` in Python

## *"One at a Time"*

> **O'Reilly Skill Objective:** *Create a custom iterable class using `__iter__` and `__next__`, and use Python's iterator protocol explicitly to create custom iterators.*

> **Previously Learned:** `for` loops, generators, `yield`, generator expressions, `__len__`, `__getitem__`, duck typing, dunder methods

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

*The firm needs to page through 2 million case records, but loading them all into a list crashes the system.*

```python
# ❌ Loads everything into memory
all_cases = database.fetch_all()    # 2 million rows → RAM explodes!
for case in all_cases:
    process(case)
```

**Mike:** "We don't need all 2 million at once. We need them ONE AT A TIME."

```python
# ✅ Iterator fetches one record at a time
case_stream = CaseIterator(database)
for case in case_stream:
    process(case)    # Only ONE case in memory at any moment
```

**Harvey:** "A list is a truck carrying the entire shipment. An iterator is a conveyor belt — one item at a time, on demand."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Iterators are the engine behind every `for` loop in Python."

| Concept | What It Is |
|---------|-----------|
| **Iterable** | An object that CAN be looped over — has `__iter__()` |
| **Iterator** | An object that DOES the looping — has `__next__()` |
| **`__iter__()`** | Returns the iterator object |
| **`__next__()`** | Returns the next value, raises `StopIteration` when done |
| **`StopIteration`** | Signal that the iterator is exhausted |
| **`iter()`** | Built-in that calls `__iter__()` |
| **`next()`** | Built-in that calls `__next__()` |

```python
# What 'for' actually does behind the scenes:
my_list = [1, 2, 3]

# for x in my_list:
#     print(x)

# IS EQUIVALENT TO:
iterator = iter(my_list)        # calls my_list.__iter__()
while True:
    try:
        x = next(iterator)      # calls iterator.__next__()
        print(x)
    except StopIteration:
        break
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Iterable vs Iterator — The Distinction ⭐

```python
# ITERABLE: has __iter__, returns an iterator
my_list = [10, 20, 30]
print(hasattr(my_list, '__iter__'))    # True — it's iterable!

# ITERATOR: has __iter__ AND __next__
my_iter = iter(my_list)
print(hasattr(my_iter, '__iter__'))    # True
print(hasattr(my_iter, '__next__'))    # True — it's an iterator!

# Using the iterator manually
print(next(my_iter))    # 10
print(next(my_iter))    # 20
print(next(my_iter))    # 30
# print(next(my_iter))  # StopIteration!

# Key difference:
# - An iterable can be looped over MULTIPLE times (list, tuple, dict)
# - An iterator can only be consumed ONCE!

my_iter2 = iter(my_list)
print(list(my_iter2))    # [10, 20, 30]
print(list(my_iter2))    # [] — exhausted!
```

---

### 3.2 Your First Custom Iterator ⭐⭐

```python
class CountUp:
    """Iterator that counts from start to stop."""
    
    def __init__(self, start, stop):
        self.current = start
        self.stop = stop
    
    def __iter__(self):
        return self    # Iterator returns itself
    
    def __next__(self):
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# Usage in a for loop
for n in CountUp(1, 6):
    print(n, end=" ")
# 1 2 3 4 5

# Manual usage
counter = CountUp(10, 13)
print(next(counter))    # 10
print(next(counter))    # 11
print(next(counter))    # 12
# next(counter)         # StopIteration!
```

---

### 3.3 Separate Iterable and Iterator ⭐⭐

```python
# Best practice: separate the ITERABLE (container) from the ITERATOR (cursor)
# This allows multiple independent iterations over the same data!

class ClientRoster:
    """ITERABLE: the collection of clients."""
    
    def __init__(self, clients):
        self._clients = list(clients)
    
    def __iter__(self):
        return ClientRosterIterator(self._clients)
    
    def __len__(self):
        return len(self._clients)

class ClientRosterIterator:
    """ITERATOR: the cursor that walks through clients."""
    
    def __init__(self, clients):
        self._clients = clients
        self._index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self._index >= len(self._clients):
            raise StopIteration
        client = self._clients[self._index]
        self._index += 1
        return client

# Create the iterable
roster = ClientRoster(["Wayne", "Stark", "Oscorp", "Queen"])

# Iterate MULTIPLE times — fresh iterator each time!
print("First pass:")
for client in roster:
    print(f"  {client}")

print("Second pass:")
for client in roster:
    print(f"  {client}")

# Two simultaneous iterators!
iter1 = iter(roster)
iter2 = iter(roster)
print(next(iter1))    # Wayne
print(next(iter1))    # Stark
print(next(iter2))    # Wayne ← independent!
```

---

### 3.4 Infinite Iterators ⭐

```python
class InfiniteCounter:
    """Counts forever — never raises StopIteration."""
    
    def __init__(self, start=0, step=1):
        self.current = start
        self.step = step
    
    def __iter__(self):
        return self
    
    def __next__(self):
        value = self.current
        self.current += self.step
        return value

# ⚠️ Don't use in a plain for loop — it never stops!
# for n in InfiniteCounter():    # Infinite loop!

# ✅ Use with manual control or itertools.islice
from itertools import islice

counter = InfiniteCounter(start=100, step=5)
first_ten = list(islice(counter, 10))
print(first_ten)    # [100, 105, 110, 115, 120, 125, 130, 135, 140, 145]

# ✅ Or with a break condition
for n in InfiniteCounter():
    if n > 20:
        break
    print(n, end=" ")
# 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
```

---

### 3.5 Iterator with Filtering and Transformation ⭐⭐

```python
class FilteredIterator:
    """Iterator that filters items from a source."""
    
    def __init__(self, source, predicate):
        self._source = iter(source)
        self._predicate = predicate
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while True:
            item = next(self._source)    # Raises StopIteration when exhausted
            if self._predicate(item):
                return item

class MappedIterator:
    """Iterator that transforms items from a source."""
    
    def __init__(self, source, transform):
        self._source = iter(source)
        self._transform = transform
    
    def __iter__(self):
        return self
    
    def __next__(self):
        item = next(self._source)
        return self._transform(item)

# Chain them!
numbers = range(1, 21)
evens = FilteredIterator(numbers, lambda x: x % 2 == 0)
doubled = MappedIterator(evens, lambda x: x * 2)

print(list(doubled))    # [4, 8, 12, 16, 20, 24, 28, 32, 36, 40]
```

---

### 3.6 The Iterator Protocol and Built-in Support ⭐

```python
# Python's built-in functions work with any iterator/iterable:

class Fibonacci:
    """First n Fibonacci numbers."""
    
    def __init__(self, n):
        self.n = n
        self.a, self.b = 0, 1
        self.count = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.n:
            raise StopIteration
        self.count += 1
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        return value

fib = Fibonacci(10)

# Works with ALL built-in functions!
print(list(Fibonacci(10)))           # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
print(sum(Fibonacci(10)))            # 88
print(max(Fibonacci(10)))            # 34
print(min(Fibonacci(10)))            # 0
print(sorted(Fibonacci(10)))         # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
print(tuple(Fibonacci(5)))           # (0, 1, 1, 2, 3)

# Unpacking
a, b, c, d, e = Fibonacci(5)
print(a, b, c, d, e)                  # 0 1 1 2 3

# 'in' operator
print(8 in Fibonacci(10))             # True
print(7 in Fibonacci(10))             # False

# enumerate
for i, val in enumerate(Fibonacci(5)):
    print(f"  F({i}) = {val}")
```

---

### 3.7 `__getitem__` — The Legacy Iterator Protocol ⭐

```python
# Before __iter__, Python used __getitem__ for iteration
# It still works! Python falls back to it.

class OldStyleIterable:
    """Iterable via __getitem__ — no __iter__ needed."""
    
    def __init__(self, data):
        self._data = data
    
    def __getitem__(self, index):
        if index >= len(self._data):
            raise IndexError
        return self._data[index]

old = OldStyleIterable(["Wayne", "Stark", "Oscorp"])

# for loop works!
for item in old:
    print(f"  {item}")

# Behind the scenes: Python calls __getitem__(0), __getitem__(1), ...
# until IndexError is raised.

# ✅ But prefer __iter__/__next__ — it's explicit and more flexible
```

---

### 3.8 Iterators vs Generators — Comparison ⭐⭐

```python
# ITERATOR CLASS — more control, more code
class CountUp:
    def __init__(self, start, stop):
        self.current = start
        self.stop = stop
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# GENERATOR FUNCTION — same result, less code!
def count_up(start, stop):
    current = start
    while current < stop:
        yield current
        current += 1

# Both work identically
print(list(CountUp(1, 6)))       # [1, 2, 3, 4, 5]
print(list(count_up(1, 6)))      # [1, 2, 3, 4, 5]
```

**When to use which:**

| Criteria | Iterator Class | Generator |
|----------|:-:|:-:|
| Simple sequences | ❌ Verbose | ✅ Use this |
| Complex state management | ✅ Multiple attributes | ⚠️ Gets messy |
| Need reset/rewind | ✅ Can implement | ❌ One-shot |
| Multiple simultaneous cursors | ✅ Separate instances | ❌ Single flow |
| Need `__len__` or `__contains__` | ✅ Add dunders | ❌ Not available |
| Lazy data pipelines | Either works | ✅ Simpler |

---

### 3.9 Building a Reusable Pipeline

```python
class Pipeline:
    """Chainable iterator pipeline for data processing."""
    
    def __init__(self, source):
        self._source = source
    
    def __iter__(self):
        return iter(self._source)
    
    def filter(self, predicate):
        """Return a new Pipeline with filtered items."""
        return Pipeline(item for item in self if predicate(item))
    
    def map(self, transform):
        """Return a new Pipeline with transformed items."""
        return Pipeline(transform(item) for item in self)
    
    def take(self, n):
        """Return only the first n items."""
        return Pipeline(islice(self, n))
    
    def collect(self):
        """Materialize the pipeline into a list."""
        return list(self)
    
    def reduce(self, func, initial=None):
        """Reduce the pipeline to a single value."""
        from functools import reduce as _reduce
        if initial is not None:
            return _reduce(func, self, initial)
        return _reduce(func, self)
    
    def first(self, default=None):
        """Get the first item."""
        return next(iter(self), default)

from itertools import islice

# Usage
data = range(1, 101)

result = (
    Pipeline(data)
    .filter(lambda x: x % 2 == 0)      # Even numbers
    .map(lambda x: x ** 2)              # Squared
    .filter(lambda x: x > 100)          # Above 100
    .take(5)                             # First 5
    .collect()
)

print(result)    # [144, 196, 256, 324, 400]

total = (
    Pipeline(range(1, 11))
    .map(lambda x: x * 100)
    .reduce(lambda a, b: a + b)
)
print(total)     # 5500
```

---

### 3.10 Solving the Firm's Problem — Streaming Case Iterator

```python
# ============================================
# Pearson Specter Litt — Streaming Case System
# Iterators for memory-efficient data processing
# ============================================

from itertools import islice
from datetime import datetime, timedelta
import random

class CaseRecord:
    """A single case record."""
    __slots__ = ('case_id', 'client', 'attorney', 'hours', 'status', 'date')
    
    def __init__(self, case_id, client, attorney, hours, status, date):
        self.case_id = case_id
        self.client = client
        self.attorney = attorney
        self.hours = hours
        self.status = status
        self.date = date
    
    def __repr__(self):
        return f"Case({self.case_id}, '{self.client}', {self.hours}h)"


class CaseDatabase:
    """Simulated database with 2 million records."""
    
    CLIENTS = ["Wayne", "Stark", "Oscorp", "Queen", "Luthor", "Palmer", "Allen"]
    ATTORNEYS = ["Harvey", "Mike", "Louis", "Katrina", "Samantha"]
    STATUSES = ["open", "closed", "pending", "review"]
    
    def __init__(self, total_records=100_000):
        self.total = total_records
    
    def _generate_record(self, index):
        """Generate a single record on-the-fly (simulates DB fetch)."""
        return CaseRecord(
            case_id=f"CASE-{index:07d}",
            client=random.choice(self.CLIENTS),
            attorney=random.choice(self.ATTORNEYS),
            hours=random.randint(1, 200),
            status=random.choice(self.STATUSES),
            date=datetime.now() - timedelta(days=random.randint(0, 365)),
        )
    
    def __iter__(self):
        """Yields records one at a time — never loads all in memory."""
        return CaseDatabaseIterator(self)
    
    def __len__(self):
        return self.total


class CaseDatabaseIterator:
    """Streams records one at a time."""
    
    def __init__(self, db):
        self._db = db
        self._index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self._index >= self._db.total:
            raise StopIteration
        record = self._db._generate_record(self._index)
        self._index += 1
        return record


class CaseAnalyzer:
    """Analyze cases using iterator pipelines."""
    
    def __init__(self, database):
        self.db = database
    
    def by_attorney(self, attorney_name):
        """Filter cases by attorney — lazy!"""
        return (case for case in self.db if case.attorney == attorney_name)
    
    def by_status(self, status):
        """Filter cases by status — lazy!"""
        return (case for case in self.db if case.status == status)
    
    def high_hours(self, threshold=100):
        """Cases above hour threshold — lazy!"""
        return (case for case in self.db if case.hours >= threshold)
    
    def summary(self, cases, limit=None):
        """Summarize cases (materializes a limited set)."""
        stream = islice(cases, limit) if limit else cases
        
        total_hours = 0
        count = 0
        for case in stream:
            total_hours += case.hours
            count += 1
        
        return {"count": count, "total_hours": total_hours,
                "avg_hours": total_hours / count if count else 0}


# === Demonstrate ===
print("=" * 65)
print(f"{'🏛️  STREAMING CASE SYSTEM':^65}")
print("=" * 65)

db = CaseDatabase(total_records=100_000)
analyzer = CaseAnalyzer(db)

# Preview first 5 records (only 5 in memory!)
print(f"\n📋 First 5 records (of {len(db):,}):")
for case in islice(db, 5):
    print(f"  {case.case_id} | {case.client:<10} | {case.attorney:<10} | "
          f"{case.hours:>3}h | {case.status}")

# Harvey's cases (lazy — processes one at a time)
print(f"\n👤 Harvey's caseload (first 10):")
harvey_cases = analyzer.by_attorney("Harvey")
for case in islice(harvey_cases, 10):
    print(f"  {case.case_id}: {case.client} ({case.hours}h)")

# High-hour cases
print(f"\n⏰ High-hour cases (>150h, first 5):")
big_cases = analyzer.high_hours(150)
for case in islice(big_cases, 5):
    print(f"  {case.case_id}: {case.client} — {case.hours}h")

# Summary stats (streams through entire dataset, ONE record at a time)
print(f"\n📊 Full database summary:")
stats = analyzer.summary(db)
print(f"  Cases: {stats['count']:,}")
print(f"  Total hours: {stats['total_hours']:,}")
print(f"  Avg hours: {stats['avg_hours']:.1f}")

# Open cases summary
print(f"\n📊 Open cases summary:")
open_stats = analyzer.summary(analyzer.by_status("open"))
print(f"  Open cases: {open_stats['count']:,}")
print(f"  Open hours: {open_stats['total_hours']:,}")

# Memory: at NO POINT was the entire dataset in memory!
print(f"\n💾 Memory: streamed {len(db):,} records one at a time")
print(f"{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Iterator Exhaustion — One-Shot Only 🔥

```python
counter = CountUp(1, 4)
print(list(counter))    # [1, 2, 3]
print(list(counter))    # [] ← Empty! Already exhausted!

# ✅ Fix: use a separate iterable that creates fresh iterators
class ReusableCounter:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    
    def __iter__(self):
        return CountUp(self.start, self.stop)    # Fresh iterator each time!

rc = ReusableCounter(1, 4)
print(list(rc))    # [1, 2, 3]
print(list(rc))    # [1, 2, 3] ✅
```

---

### Pitfall 2: Forgetting `StopIteration` 🔥

```python
# ❌ Never raises StopIteration — infinite loop in for!
class BadIterator:
    def __init__(self):
        self.n = 0
    def __iter__(self):
        return self
    def __next__(self):
        self.n += 1
        return self.n    # Never stops!

# for x in BadIterator(): print(x)    # Infinite loop!

# ✅ Always have a termination condition
class GoodIterator:
    def __init__(self, limit):
        self.n = 0
        self.limit = limit
    def __iter__(self):
        return self
    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration    # ✅
        self.n += 1
        return self.n
```

---

### Pitfall 3: Iterator Sharing in Multiple `for` Loops

```python
# Iterators are consumed — sharing causes lost data
my_iter = iter([1, 2, 3, 4, 5])

for x in my_iter:
    if x == 3:
        break    # Stops at 3

# Resuming uses the SAME iterator — starts from 4!
for x in my_iter:
    print(x)    # Prints 4, 5 (not 1, 2, 3, 4, 5!)
```

---

### Pitfall 4: Modifying Data During Iteration

```python
# ❌ Modifying a list while iterating
items = [1, 2, 3, 4, 5]
# for item in items:
#     if item % 2 == 0:
#         items.remove(item)    # Skips elements!

# ✅ Iterate over a copy, or build a new list
items = [x for x in items if x % 2 != 0]
```

---

### Pitfall 5: Returning `self` vs New Iterator

```python
# ❌ __iter__ returns self — can't iterate twice!
class BadIterable:
    def __init__(self):
        self.data = [1, 2, 3]
        self.index = 0
    def __iter__(self):
        return self    # Same object each time!
    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        val = self.data[self.index]
        self.index += 1
        return val

# ✅ __iter__ returns a NEW iterator
class GoodIterable:
    def __init__(self):
        self.data = [1, 2, 3]
    def __iter__(self):
        return iter(self.data)    # Fresh iterator!
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏭 Analogy: Iterable = Warehouse, Iterator = Conveyor Belt

```
LIST (load everything):
  🚛 Truck drives to warehouse.
  📦📦📦📦📦📦📦📦 Loads EVERY box.
  🚛 Drives to office. Unloads.
  Problem: truck too small for 2 million boxes!

ITERATOR (stream one at a time):
  🏭 Warehouse has a CONVEYOR BELT.
  📦 Belt sends one box.
  ✅ Office processes it.
  📦 Belt sends next box.
  ✅ Office processes it.
  ...
  Only ONE box in transit at any time!

DONNA'S RULE:
  "An iterable is the warehouse — it has the goods.
   An iterator is the conveyor belt — it delivers them.
   You don't need the whole warehouse on your desk.
   You need one file at a time."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Range Clone**
```
Build MyRange(start, stop, step) that works like range():
  for x in MyRange(0, 10, 2): print(x)  # 0, 2, 4, 6, 8
Support negative step for counting down.
```

**Exercise 2: Repeat Iterator**
```
Build Repeat(value, times):
  list(Repeat("hello", 3))  →  ["hello", "hello", "hello"]
  list(Repeat(42, 5))       →  [42, 42, 42, 42, 42]
```

**Exercise 3: Cycle Iterator**
```
Build Cycle(iterable) that loops forever:
  c = Cycle([1, 2, 3])
  [next(c) for _ in range(7)]  →  [1, 2, 3, 1, 2, 3, 1]
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Chunked Iterator**
```
Build Chunked(iterable, chunk_size):
  list(Chunked(range(10), 3))  →  [[0,1,2], [3,4,5], [6,7,8], [9]]
```

**Exercise 5: Merge Sorted**
```
Build MergeSorted(iter1, iter2) that merges two sorted
iterators into one sorted output:
  list(MergeSorted([1,3,5], [2,4,6]))  →  [1,2,3,4,5,6]
```

**Exercise 6: Sliding Window**
```
Build Window(iterable, size):
  list(Window([1,2,3,4,5], 3))  →  [(1,2,3), (2,3,4), (3,4,5)]
```

---

### 🔴 Advanced Exercises

**Exercise 7: Lazy CSV Reader**
```
Build a CSVIterator(filename) that:
  - Opens a CSV file
  - Yields one row at a time as a dict
  - Supports filter/map chaining
  - Never loads entire file
```

**Exercise 8: Tree Iterator**
```
Build a tree structure with BFS and DFS iterators:
  for node in tree.bfs(): ...
  for node in tree.dfs(): ...
Both are separate iterator classes.
```

**Exercise 9: Database Paginator**
```
Build PagedIterator(query, page_size=100):
  - Fetches page_size records at a time
  - Yields individual records
  - Transparently fetches next page when needed
  - Tracks total pages fetched
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: No StopIteration
class Counter:
    def __init__(self): self.n = 0
    def __iter__(self): return self
    def __next__(self):
        self.n += 1
        return self.n    # Never stops!

# Bug 2: __iter__ missing
class NotIterable:
    def __next__(self): return 1
# for x in NotIterable():    # TypeError: not iterable!

# Bug 3: Exhausted iterator reuse
data = iter([1, 2, 3])
print(sum(data))    # 6
print(sum(data))    # 0 ← not 6!

# Bug 4: __iter__ doesn't return iterator
class Broken:
    def __iter__(self):
        return [1, 2, 3]    # Returns a list, not an iterator!
    # Actually this one works because list has __iter__...
    # But semantically it should return self or an iterator
```

### Fixed Code ✅

```python
# Fix 1: Add StopIteration
class Counter:
    def __init__(self, limit):
        self.n = 0
        self.limit = limit
    def __iter__(self): return self
    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration
        self.n += 1
        return self.n

# Fix 2: Add __iter__
class NowIterable:
    def __iter__(self):
        return self
    def __next__(self):
        ...

# Fix 3: Use iterable, not iterator
data = [1, 2, 3]    # Iterable — creates fresh iterator each time
print(sum(data))    # 6
print(sum(data))    # 6 ✅

# Fix 4: Return proper iterator
class Fixed:
    def __iter__(self):
        return iter([1, 2, 3])    # ✅ Returns an iterator
```

---

## 8. 🌍 Real-World Application

### Iterators in Python Ecosystem

| Library / Feature | Iterator Usage |
|-------------------|---------------|
| **File objects** | `for line in open("file")` — reads line by line |
| **`csv.reader`** | Yields one row at a time |
| **`json.JSONDecoder`** | Streaming JSON parsing |
| **Django QuerySets** | `.iterator()` for memory-efficient DB reads |
| **SQLAlchemy** | `session.execute().scalars()` streams rows |
| **`itertools`** | Entire module built on iterators |
| **`map`/`filter`/`zip`** | Return iterators (Python 3) |
| **`os.scandir()`** | Iterates directory entries lazily |
| **Pandas** | `read_csv(chunksize=N)` returns chunked iterator |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between an iterable and an iterator?

**Q2.** What two methods make up the iterator protocol?

**Q3.** How does a `for` loop work internally?

**Q4.** Why does an iterator implement `__iter__` that returns `self`?

**Q5.** How do you signal that an iterator is exhausted?

**Q6.** Why should you separate the iterable from the iterator?

**Q7.** What happens if you iterate over an exhausted iterator?

**Q8.** When should you use an iterator class vs a generator?

**Q9.** What is `__getitem__`-based iteration?

**Q10.** How does `next(iterator, default)` work?

---

### 📋 Answer Key

> **Q1.** An iterable has `__iter__()` and can be looped over (lists, tuples, strings). An iterator has both `__iter__()` and `__next__()` and tracks position through the data.

> **Q2.** `__iter__()` returns the iterator object, `__next__()` returns the next value or raises `StopIteration`.

> **Q3.** It calls `iter(obj)` to get an iterator, then repeatedly calls `next(iterator)` until `StopIteration` is raised.

> **Q4.** So iterators can be used directly in `for` loops. `for x in iterator:` calls `iter(iterator)` which returns `self`.

> **Q5.** Raise `StopIteration` in `__next__()` when there are no more items.

> **Q6.** So the collection (iterable) can be iterated multiple times independently. Each `__iter__()` call creates a fresh iterator with its own position.

> **Q7.** It immediately yields nothing — `list(exhausted_iter)` returns `[]`.

> **Q8.** Use iterator classes when you need complex state, reset capability, multiple simultaneous cursors, or extra dunder methods. Use generators for simple linear sequences.

> **Q9.** A legacy protocol where Python calls `__getitem__(0)`, `__getitem__(1)`, etc. until `IndexError`. Still works but `__iter__`/`__next__` is preferred.

> **Q10.** Returns `default` instead of raising `StopIteration` when exhausted: `next(iter([]), "empty")` → `"empty"`.

---

## 🎬 End of Lesson Scene

**Louis:** "Memory usage during the full database scan?"

**Mike:** "Constant. One record at a time. We processed 100,000 cases and at no point did we have more than one in memory."

**Harvey:** "Like a conveyor belt. Not a truck. Next: regex — pattern matching on all that case data."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Iterable vs iterator distinction | ✅ |
| 2 | `__iter__` and `__next__` protocol | ✅ |
| 3 | Separate iterable and iterator classes | ✅ |
| 4 | Infinite iterators | ✅ |
| 5 | Filtering and transforming iterators | ✅ |
| 6 | Built-in function compatibility | ✅ |
| 7 | `__getitem__`-based (legacy) iteration | ✅ |
| 8 | Iterator class vs generator comparison | ✅ |
| 9 | Chainable iterator pipeline | ✅ |
| 10 | Streaming data processing | ✅ |

---

> **Next Lesson →** Level 4, Lesson 8: *Use Regex in Python to Match Patterns and Extract Text*
