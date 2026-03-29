# 🏛️ Level 5 – Innovating | Lesson 8: Monitor Memory Usage in Python with memory-profiler

## *"The Balance Sheet"*

> **O'Reilly Skill Objective:** *Monitor memory usage using memory-profiler.*

> **Previously Learned:** `__slots__`, generators, `sys.getsizeof`, `timeit`, `cProfile`, context managers, decorators

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

*The firm's case analysis server keeps crashing. The process eats 8 GB of RAM and gets killed by the OS.*

```python
# ❌ Loading everything into memory
def analyze_all_cases():
    cases = load_all_cases()         # 2 GB
    documents = load_all_documents() # 3 GB
    billing = load_all_billing()     # 1 GB
    cross_ref = build_cross_ref(cases, documents, billing)  # 2 GB
    # Total: 8 GB → OOM killed!
```

**Mike:** "We can't fix what we can't see. `tracemalloc` ships with Python and traces every allocation. `memory-profiler` shows us line-by-line memory usage. First we measure, then we cut."

```python
# ✅ Stream and process incrementally
def analyze_cases_streaming():
    for case in stream_cases():         # One at a time
        docs = get_docs(case.id)        # Small batch
        result = analyze(case, docs)    # Process and discard
        yield result                    # Don't accumulate!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Speed tells you how fast. Memory tells you how much. You need both."

| Tool | Purpose | Built-in? |
|------|---------|:---------:|
| **`sys.getsizeof()`** | Size of a single object (bytes) | ✅ |
| **`tracemalloc`** | Trace memory allocations | ✅ |
| **`memory-profiler`** | Line-by-line memory usage | `pip install` |
| **`objgraph`** | Visualize object references | `pip install` |
| **`pympler`** | Deep size + tracking | `pip install` |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `sys.getsizeof` — Quick Object Sizes ⭐

```python
import sys

# getsizeof returns the size of a SINGLE object in bytes
# ⚠️ Does NOT include referenced objects!

print(f"int(0):       {sys.getsizeof(0)} bytes")         # 28
print(f"int(1):       {sys.getsizeof(1)} bytes")         # 28
print(f"int(2**30):   {sys.getsizeof(2**30)} bytes")     # 32
print(f"float(3.14):  {sys.getsizeof(3.14)} bytes")      # 24
print(f"bool(True):   {sys.getsizeof(True)} bytes")      # 28
print(f"None:         {sys.getsizeof(None)} bytes")       # 16
print(f"str(''):      {sys.getsizeof('')} bytes")         # 49
print(f"str('hello'): {sys.getsizeof('hello')} bytes")   # 54
print(f"list([]):     {sys.getsizeof([])} bytes")         # 56
print(f"dict({{}}):     {sys.getsizeof({})} bytes")       # 64
print(f"set():        {sys.getsizeof(set())} bytes")      # 216
print(f"tuple(()):    {sys.getsizeof(())} bytes")         # 40

# Compare containers
n = 1000
print(f"\nlist of {n} ints:  {sys.getsizeof(list(range(n))):,} bytes")
print(f"tuple of {n} ints: {sys.getsizeof(tuple(range(n))):,} bytes")
print(f"set of {n} ints:   {sys.getsizeof(set(range(n))):,} bytes")

# ⚠️ getsizeof is SHALLOW — doesn't count nested objects!
data = [[1, 2, 3] for _ in range(1000)]
print(f"\nShallow size of list-of-lists: {sys.getsizeof(data):,} bytes")
# Only counts the outer list, NOT the inner lists!

# === Deep size calculation ===
def deep_getsizeof(obj, seen=None):
    """Recursively calculate total memory of an object and its contents."""
    if seen is None:
        seen = set()
    obj_id = id(obj)
    if obj_id in seen:
        return 0
    seen.add(obj_id)
    
    size = sys.getsizeof(obj)
    
    if isinstance(obj, dict):
        size += sum(deep_getsizeof(k, seen) + deep_getsizeof(v, seen) for k, v in obj.items())
    elif isinstance(obj, (list, tuple, set, frozenset)):
        size += sum(deep_getsizeof(item, seen) for item in obj)
    elif hasattr(obj, '__dict__'):
        size += deep_getsizeof(obj.__dict__, seen)
    
    return size

data = [{"name": f"Client_{i}", "revenue": i * 1000} for i in range(100)]
print(f"Deep size of 100 client dicts: {deep_getsizeof(data):,} bytes")
```

---

### 3.2 `tracemalloc` — Built-in Memory Tracing ⭐⭐

```python
import tracemalloc

# tracemalloc traces ALL memory allocations
# Built into Python — no pip install needed!

# === Basic usage ===
tracemalloc.start()

# --- Code to measure ---
clients = [{"name": f"Client_{i}", "revenue": i * 1000} for i in range(10_000)]
cases = [{"id": i, "hours": i * 10} for i in range(50_000)]
report = "\n".join(f"Case {c['id']}: {c['hours']}h" for c in cases)
# --- End measured code ---

snapshot = tracemalloc.take_snapshot()
stats = snapshot.statistics('lineno')    # Group by line number

print("🏛️ TOP 10 MEMORY ALLOCATIONS:")
for stat in stats[:10]:
    print(f"  {stat}")

current, peak = tracemalloc.get_traced_memory()
print(f"\nCurrent memory: {current / 1024 / 1024:.1f} MB")
print(f"Peak memory:    {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```

---

### 3.3 `tracemalloc` — Comparing Snapshots ⭐⭐

```python
import tracemalloc

tracemalloc.start()

# Snapshot BEFORE
snapshot1 = tracemalloc.take_snapshot()

# === Some operations ===
big_list = [x ** 2 for x in range(100_000)]
big_dict = {f"key_{i}": i for i in range(50_000)}

# Snapshot AFTER
snapshot2 = tracemalloc.take_snapshot()

# Compare — what CHANGED?
stats = snapshot2.compare_to(snapshot1, 'lineno')

print("🔍 MEMORY CHANGES (top 5):")
for stat in stats[:5]:
    print(f"  {stat}")

# Output shows:
# +5.2 MiB  script.py:8  — big_list allocation
# +3.1 MiB  script.py:9  — big_dict allocation

# === Filter by filename ===
stats = snapshot2.compare_to(snapshot1, 'filename')
for stat in stats[:5]:
    print(f"  {stat}")

# === Traceback for specific allocations ===
tracemalloc.start(25)    # Store 25 frames of traceback

snapshot = tracemalloc.take_snapshot()
stats = snapshot.statistics('traceback')

# Print traceback of biggest allocation
if stats:
    print(f"\n📍 Biggest allocation traceback:")
    for line in stats[0].traceback.format():
        print(f"  {line}")

tracemalloc.stop()
```

---

### 3.4 `memory-profiler` — Line-by-Line Memory Usage ⭐⭐

```python
# pip install memory-profiler

# === Usage: @profile decorator ===
# Add @profile to functions, then run with:
#   python -m memory_profiler script.py

from memory_profiler import profile

@profile
def process_billing():
    """Generate billing report."""
    clients = [{"name": f"Client_{i}", "revenue": i * 1000}
               for i in range(100_000)]                       # +15.3 MiB
    
    high_value = [c for c in clients if c["revenue"] > 50_000_000]  # +0.1 MiB
    
    report_lines = [f"{c['name']}: ${c['revenue']:,}"
                    for c in clients]                          # +8.2 MiB
    
    report = "\n".join(report_lines)                           # +7.8 MiB
    
    del clients          # Free memory!                        # -15.3 MiB
    del report_lines     # Free memory!                        # -8.2 MiB
    
    return report

# Run: python -m memory_profiler script.py
# Output:
# Line #    Mem usage    Increment  Occurrences   Line Contents
# ============================================================
#     6     45.2 MiB     45.2 MiB           1   @profile
#     7                                         def process_billing():
#     8     60.5 MiB     15.3 MiB           1       clients = [...]
#     9     60.6 MiB      0.1 MiB           1       high_value = [...]
#    10     68.8 MiB      8.2 MiB           1       report_lines = [...]
#    11     76.6 MiB      7.8 MiB           1       report = "\n".join(...)
#    12     61.3 MiB    -15.3 MiB           1       del clients
#    13     53.1 MiB     -8.2 MiB           1       del report_lines

# === Programmatic usage ===
from memory_profiler import memory_usage

def my_function():
    data = [list(range(1000)) for _ in range(10_000)]
    return len(data)

# Track memory over time
mem_usage = memory_usage((my_function,), interval=0.1)
print(f"Peak memory: {max(mem_usage):.1f} MiB")
print(f"Memory range: {min(mem_usage):.1f} - {max(mem_usage):.1f} MiB")
```

---

### 3.5 Memory Comparison: Data Structures ⭐

```python
import sys
from collections import namedtuple
from dataclasses import dataclass

# Compare memory usage of different structures for the SAME data

N = 100_000

# 1. Dictionary
dict_clients = [{"name": f"C_{i}", "revenue": i, "active": True} for i in range(N)]

# 2. Named tuple
ClientTuple = namedtuple("Client", ["name", "revenue", "active"])
tuple_clients = [ClientTuple(f"C_{i}", i, True) for i in range(N)]

# 3. Regular class
class ClientClass:
    def __init__(self, name, revenue, active):
        self.name = name
        self.revenue = revenue
        self.active = active

class_clients = [ClientClass(f"C_{i}", i, True) for i in range(N)]

# 4. __slots__ class
class ClientSlots:
    __slots__ = ('name', 'revenue', 'active')
    def __init__(self, name, revenue, active):
        self.name = name
        self.revenue = revenue
        self.active = active

slots_clients = [ClientSlots(f"C_{i}", i, True) for i in range(N)]

# 5. Dataclass
@dataclass
class ClientDC:
    name: str
    revenue: int
    active: bool

dc_clients = [ClientDC(f"C_{i}", i, True) for i in range(N)]

# 6. Dataclass with slots
@dataclass(slots=True)
class ClientDCSlots:
    name: str
    revenue: int
    active: bool

dcs_clients = [ClientDCSlots(f"C_{i}", i, True) for i in range(N)]

# Measure
import tracemalloc

def measure_structure(label, factory):
    tracemalloc.start()
    data = factory()
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    return label, peak / 1024 / 1024  # MB

results = [
    measure_structure("dict", lambda: [{"name": f"C_{i}", "revenue": i, "active": True} for i in range(N)]),
    measure_structure("namedtuple", lambda: [ClientTuple(f"C_{i}", i, True) for i in range(N)]),
    measure_structure("class", lambda: [ClientClass(f"C_{i}", i, True) for i in range(N)]),
    measure_structure("__slots__", lambda: [ClientSlots(f"C_{i}", i, True) for i in range(N)]),
    measure_structure("dataclass", lambda: [ClientDC(f"C_{i}", i, True) for i in range(N)]),
    measure_structure("dc(slots)", lambda: [ClientDCSlots(f"C_{i}", i, True) for i in range(N)]),
]

print(f"\n{'Structure':<15} {'Memory (MB)':>12}")
print(f"{'─'*15} {'─'*12}")
for label, mb in sorted(results, key=lambda x: x[1]):
    bar = '█' * int(mb * 2)
    print(f"{label:<15} {mb:>10.1f} MB  {bar}")

# Typical results (100K records):
# __slots__         6.2 MB  ████████████
# dc(slots)         6.2 MB  ████████████
# namedtuple        8.5 MB  █████████████████
# class            14.8 MB  █████████████████████████████
# dataclass        14.8 MB  █████████████████████████████
# dict             22.1 MB  ████████████████████████████████████████████
```

---

### 3.6 Generators vs Lists — Memory Impact ⭐

```python
import tracemalloc

# === List: ALL data in memory at once ===
tracemalloc.start()
data_list = [x ** 2 for x in range(1_000_000)]
total_list = sum(data_list)
list_current, list_peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

# === Generator: ONE item at a time ===
tracemalloc.start()
data_gen = (x ** 2 for x in range(1_000_000))
total_gen = sum(data_gen)
gen_current, gen_peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

print(f"List:      peak = {list_peak / 1024 / 1024:.1f} MB")    # ~8.0 MB
print(f"Generator: peak = {gen_peak / 1024 / 1024:.3f} MB")     # ~0.001 MB
print(f"Same result: {total_list == total_gen}")                 # True

# Generator uses ~8000x LESS memory for the same result!

# === Streaming file processing ===
def process_file_list(filepath):
    """❌ Loads entire file into memory."""
    with open(filepath) as f:
        lines = f.readlines()    # ALL lines in memory!
    return [process_line(line) for line in lines]

def process_file_generator(filepath):
    """✅ Processes one line at a time."""
    with open(filepath) as f:
        for line in f:           # One line in memory!
            yield process_line(line)
```

---

### 3.7 Finding Memory Leaks ⭐⭐

```python
import tracemalloc
import gc

# === Common leak: growing cache without bounds ===
class LeakyCache:
    """❌ Cache that never evicts — memory grows forever!"""
    _cache = {}
    
    @classmethod
    def get(cls, key, factory):
        if key not in cls._cache:
            cls._cache[key] = factory()
        return cls._cache[key]

# After 100K unique keys: cache holds 100K objects forever!

# === Fixed: bounded cache ===
from functools import lru_cache
from collections import OrderedDict

class BoundedCache:
    """✅ Cache with maximum size."""
    def __init__(self, max_size=1000):
        self.max_size = max_size
        self._cache = OrderedDict()
    
    def get(self, key, factory):
        if key in self._cache:
            self._cache.move_to_end(key)
            return self._cache[key]
        
        value = factory()
        self._cache[key] = value
        
        if len(self._cache) > self.max_size:
            self._cache.popitem(last=False)    # Remove oldest
        
        return value

# === Detecting leaks with tracemalloc ===
def detect_leak():
    tracemalloc.start()
    
    snapshots = []
    cache = {}
    
    for i in range(5):
        # Simulate work that leaks
        for j in range(10_000):
            cache[f"key_{i}_{j}"] = [0] * 100    # Growing!
        
        snapshot = tracemalloc.take_snapshot()
        snapshots.append(snapshot)
    
    # Compare first and last snapshot
    stats = snapshots[-1].compare_to(snapshots[0], 'lineno')
    
    print("🔍 MEMORY GROWTH:")
    for stat in stats[:5]:
        print(f"  {stat}")
    
    tracemalloc.stop()

# === Circular reference detection ===
import gc

gc.set_debug(gc.DEBUG_SAVEALL)
gc.collect()

class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

a = Node(1)
b = Node(2)
a.next = b
b.next = a    # Circular!
del a, b

collected = gc.collect()
print(f"Garbage collected: {collected} objects")
print(f"Unreachable: {len(gc.garbage)}")
```

---

### 3.8 Memory Optimization Techniques ⭐⭐

```python
# === 1. Use __slots__ for data classes ===
class RecordSlots:
    __slots__ = ('name', 'value')
    def __init__(self, name, value):
        self.name = name
        self.value = value
# 40-50% less memory per instance!

# === 2. Use generators for pipelines ===
def pipeline(filepath):
    lines = (line.strip() for line in open(filepath))
    records = (parse(line) for line in lines)
    filtered = (r for r in records if r['value'] > 100)
    return sum(r['value'] for r in filtered)
# Processes millions of records in constant memory!

# === 3. Delete large objects when done ===
big_data = load_huge_dataset()
results = process(big_data)
del big_data    # Free immediately
import gc; gc.collect()    # Force garbage collection

# === 4. Use array.array for numeric data ===
import array
# List of ints: ~28 bytes per int + 8 bytes pointer = ~36 bytes
# array of ints: 4-8 bytes per int
int_list = list(range(100_000))
int_array = array.array('i', range(100_000))
import sys
print(f"List:  {sys.getsizeof(int_list):>10,} bytes")    # ~800,000
print(f"Array: {sys.getsizeof(int_array):>10,} bytes")   # ~400,000

# === 5. Use memoryview for zero-copy slicing ===
data = bytearray(1_000_000)
view = memoryview(data)
chunk = view[1000:2000]    # No copy! Shares memory with data.

# === 6. Weak references for caches ===
import weakref

class ExpensiveObject:
    def __init__(self, data):
        self.data = data

cache = weakref.WeakValueDictionary()
obj = ExpensiveObject("big data")
cache["key"] = obj
# When 'obj' is deleted elsewhere, cache entry auto-removes!

# === 7. Interning strings ===
import sys
# Python interns short strings automatically
# For longer strings:
s = sys.intern("long_repeated_string_used_many_times")
# All references to the same interned string share ONE object
```

---

### 3.9 Building a Memory Monitor ⭐

```python
import tracemalloc
import time
import functools

class MemoryMonitor:
    """Context manager and decorator for memory monitoring."""
    
    def __init__(self, label=""):
        self.label = label
        self.peak = 0
        self.current = 0
    
    def __enter__(self):
        tracemalloc.start()
        return self
    
    def __exit__(self, *args):
        self.current, self.peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        if self.label:
            print(f"  📊 [{self.label}] "
                  f"Current: {self.current / 1024 / 1024:.2f} MB | "
                  f"Peak: {self.peak / 1024 / 1024:.2f} MB")
    
    @staticmethod
    def profile(func):
        """Decorator version."""
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            tracemalloc.start()
            result = func(*args, **kwargs)
            current, peak = tracemalloc.get_traced_memory()
            tracemalloc.stop()
            print(f"  📊 [{func.__name__}] "
                  f"Peak: {peak / 1024 / 1024:.2f} MB")
            return result
        return wrapper

# As context manager
with MemoryMonitor("Load clients"):
    clients = [{"name": f"C_{i}"} for i in range(100_000)]

# As decorator
@MemoryMonitor.profile
def build_report(n):
    data = [{"id": i, "value": i ** 2} for i in range(n)]
    return "\n".join(f"{d['id']}: {d['value']}" for d in data)

report = build_report(50_000)
```

---

### 3.10 Solving the Firm's Problem — Memory-Efficient Pipeline

```python
# ============================================
# Pearson Specter Litt — Memory-Efficient Pipeline
# Process 500K records in bounded memory
# ============================================

import tracemalloc
import sys
import time
from dataclasses import dataclass

@dataclass(slots=True)
class CaseRecord:
    case_id: str
    client: str
    hours: float
    rate: float
    status: str

def generate_records(n):
    """Simulate streaming from database — yields one at a time."""
    clients = ["Wayne", "Stark", "Oscorp", "Queen", "Palmer"]
    statuses = ["open", "discovery", "trial", "settled", "closed"]
    
    for i in range(n):
        yield CaseRecord(
            case_id=f"CASE-{i:06d}",
            client=clients[i % len(clients)],
            hours=round((i % 100) * 0.5, 1),
            rate=450.0 if i % 3 == 0 else 300.0,
            status=statuses[i % len(statuses)],
        )

def process_streaming(n_records):
    """✅ Memory-efficient: streaming pipeline."""
    totals_by_client = {}
    count = 0
    
    for record in generate_records(n_records):
        billing = record.hours * record.rate
        
        if record.client not in totals_by_client:
            totals_by_client[record.client] = {"amount": 0.0, "hours": 0.0, "cases": 0}
        
        totals_by_client[record.client]["amount"] += billing
        totals_by_client[record.client]["hours"] += record.hours
        totals_by_client[record.client]["cases"] += 1
        count += 1
    
    return totals_by_client, count

def process_batch(n_records):
    """❌ Memory-heavy: loads everything into lists."""
    all_records = list(generate_records(n_records))
    
    all_billing = [
        {"client": r.client, "amount": r.hours * r.rate, "hours": r.hours}
        for r in all_records
    ]
    
    totals_by_client = {}
    for entry in all_billing:
        if entry["client"] not in totals_by_client:
            totals_by_client[entry["client"]] = {"amount": 0.0, "hours": 0.0, "cases": 0}
        totals_by_client[entry["client"]]["amount"] += entry["amount"]
        totals_by_client[entry["client"]]["hours"] += entry["hours"]
        totals_by_client[entry["client"]]["cases"] += 1
    
    return totals_by_client, len(all_records)

# === Benchmark ===
N = 200_000

print("=" * 65)
print(f"{'🏛️  MEMORY EFFICIENCY COMPARISON':^65}")
print(f"{'Processing ' + f'{N:,}' + ' records':^65}")
print("=" * 65)

# Batch approach
tracemalloc.start()
start = time.perf_counter()
batch_result, batch_count = process_batch(N)
batch_time = time.perf_counter() - start
batch_current, batch_peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

# Streaming approach
tracemalloc.start()
start = time.perf_counter()
stream_result, stream_count = process_streaming(N)
stream_time = time.perf_counter() - start
stream_current, stream_peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

# Results
print(f"\n{'Metric':<25} {'Batch':>15} {'Streaming':>15} {'Savings':>10}")
print(f"{'─'*25} {'─'*15} {'─'*15} {'─'*10}")
print(f"{'Peak memory':<25} {batch_peak/1024/1024:>13.1f}MB {stream_peak/1024/1024:>13.1f}MB "
      f"{(1-stream_peak/batch_peak)*100:>8.0f}%")
print(f"{'Time':<25} {batch_time:>14.3f}s {stream_time:>14.3f}s")
print(f"{'Records processed':<25} {batch_count:>15,} {stream_count:>15,}")
print(f"{'Same result':<25} {str(batch_result == stream_result):>15}")

# Client billing summary
print(f"\n📊 BILLING SUMMARY:")
print(f"  {'Client':<12} {'Cases':>8} {'Hours':>10} {'Billed':>15}")
print(f"  {'─'*12} {'─'*8} {'─'*10} {'─'*15}")
for client, data in sorted(stream_result.items()):
    print(f"  {client:<12} {data['cases']:>8,} {data['hours']:>10,.1f} ${data['amount']:>13,.2f}")

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `sys.getsizeof` Is Shallow 🔥

```python
import sys

# ❌ Only measures the container, not contents!
data = [[1, 2, 3] for _ in range(1000)]
print(sys.getsizeof(data))    # ~8,000 bytes (just the outer list!)
# Actual total: ~120,000 bytes (includes inner lists + ints)

# ✅ Use deep_getsizeof (Section 3.1) or tracemalloc for true size
```

---

### Pitfall 2: `del` Doesn't Guarantee Immediate Deallocation

```python
# ❌ Expecting instant memory release
big = [0] * 10_000_000
del big    # Removes reference, but memory may not free immediately!

# ✅ Force garbage collection if needed
import gc
del big
gc.collect()    # Forces cycle detection and collection
```

---

### Pitfall 3: Hidden References Preventing Garbage Collection

```python
# ❌ Variable still referenced in traceback/exception
try:
    big_data = [0] * 10_000_000
    process(big_data)
except Exception as e:
    log_error(e)    # Exception holds reference to frame → big_data stays!

# ✅ Delete explicitly before exception handling
try:
    big_data = [0] * 10_000_000
    result = process(big_data)
    del big_data    # Free before any exception
except Exception:
    pass
```

---

### Pitfall 4: String Concatenation Memory Explosion

```python
# ❌ Each += creates a NEW string
result = ""
for i in range(100_000):
    result += f"Line {i}\n"    # O(n²) memory + time!

# ✅ Collect in list, join once
lines = [f"Line {i}" for i in range(100_000)]
result = "\n".join(lines)    # O(n) memory + time
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 💰 Analogy: Memory = The Firm's Budget

```
RAM = THE FIRM'S CASH ON HAND:
  Every object you create = salary expense.
  Limited budget. When it runs out, everything stops (OOM).

TRACEMALLOC = THE ACCOUNTANT:
  Tracks every dollar spent (every byte allocated).
  Shows you WHERE the money goes.

MEMORY-PROFILER = THE AUDIT:
  Line by line, shows what each line costs.

GENERATORS = CONTRACT WORKERS:
  Hired one at a time, paid and released.
  Never have 1000 contractors on payroll at once.

LISTS = FULL-TIME EMPLOYEES:
  ALL on payroll simultaneously.
  Great for small teams. Bankrupting for 100,000.

DONNA'S RULE:
  "Track your spending. You can't budget
   what you don't measure. And don't hire
   100,000 employees when you only need
   one at a time."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Object Size Explorer**
```
Use sys.getsizeof to measure 10+ different data types.
Compare: int, float, str, list, tuple, dict, set, None, bool.
Format as a sorted table (smallest to largest).
```

**Exercise 2: tracemalloc Basics**
```
Use tracemalloc to measure peak memory when creating:
  - 100K integers in a list
  - 100K integers in a tuple
  - 100K integers in a set
Compare peak memory for each.
```

**Exercise 3: Generator vs List**
```
Process 1 million numbers: square, filter evens, sum.
Compare memory: list comprehension vs generator.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Memory Leak Detector**
```
Build a class that grows unboundedly (simulated leak).
Use tracemalloc snapshots to detect the growth.
Report: what's growing, how fast, where.
```

**Exercise 5: Data Structure Benchmark**
```
Store 500K records in 6 structures (dict, namedtuple,
class, slots, dataclass, dataclass+slots).
Compare memory with tracemalloc. Report winner.
```

**Exercise 6: Streaming CSV Processor**
```
Process a large CSV without loading it all:
  - Count rows, sum a column, find max
  - Use generators throughout
  - Track memory with tracemalloc
  - Prove peak memory stays constant regardless of file size
```

---

### 🔴 Advanced Exercises

**Exercise 7: Memory Budget Manager**
```
Build a MemoryBudget context manager:
  with MemoryBudget(max_mb=100):
      process_data()    # Raises if memory exceeds 100MB!
```

**Exercise 8: Object Graph Analyzer**
```
Build a tool that walks object references:
  analyze(my_dict)
  → Total size: 5.2 MB
  → 10,000 strings (3.1 MB)
  → 50,000 ints (1.4 MB)
  → 200 lists (0.7 MB)
```

**Exercise 9: Memory-Optimized Cache**
```
Build an LRU cache with memory (not count) limit:
  cache = MemoryLRUCache(max_mb=50)
  cache.set("key", big_object)    # Evicts if total > 50MB
Track actual memory, not just item count.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Unbounded cache
cache = {}
def get_data(key):
    if key not in cache:
        cache[key] = expensive_load(key)
    return cache[key]    # Cache grows forever!

# Bug 2: Loading entire file into list
with open("huge.csv") as f:
    lines = f.readlines()    # 5 GB file → 5 GB in memory!

# Bug 3: Shallow size measurement
data = [{"name": f"C_{i}"} for i in range(100_000)]
print(f"Size: {sys.getsizeof(data)} bytes")    # Only list shell!

# Bug 4: Large intermediate results
raw = load_raw_data()           # 2 GB
parsed = parse_all(raw)         # + 2 GB
filtered = filter_all(parsed)   # + 1 GB
# 5 GB in memory!
```

### Fixed Code ✅

```python
# Fix 1: Bounded cache
from functools import lru_cache
@lru_cache(maxsize=1000)
def get_data(key):
    return expensive_load(key)

# Fix 2: Stream line by line
with open("huge.csv") as f:
    for line in f:      # One line at a time!
        process(line)

# Fix 3: Deep size or tracemalloc
tracemalloc.start()
data = [{"name": f"C_{i}"} for i in range(100_000)]
_, peak = tracemalloc.get_traced_memory()
print(f"True size: {peak / 1024 / 1024:.1f} MB")

# Fix 4: Pipeline — delete intermediates
raw = load_raw_data()       # 2 GB
parsed = parse_all(raw)
del raw                     # Free 2 GB!
filtered = filter_all(parsed)
del parsed                  # Free 2 GB!
# Only 1 GB in memory
```

---

## 8. 🌍 Real-World Application

### Memory Profiling in Production

| Scenario | Tool | Solution |
|----------|------|----------|
| Server memory growth | tracemalloc snapshots | Found unbounded cache |
| Data pipeline OOM | generators | Streaming instead of batch |
| 1M record processing | `__slots__` | 50% memory reduction |
| Large file processing | line-by-line iteration | Constant memory |
| API response caching | `lru_cache(maxsize)` | Bounded memory |
| Image processing | `memoryview` | Zero-copy slicing |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does `sys.getsizeof()` measure?

**Q2.** What is `tracemalloc`?

**Q3.** What does `memory-profiler` show?

**Q4.** Why do generators use less memory than lists?

**Q5.** What is a memory leak in Python?

**Q6.** How does `__slots__` reduce memory?

**Q7.** What does `tracemalloc.compare_to()` do?

**Q8.** Why is `del` not always sufficient?

**Q9.** Name three memory optimization techniques.

**Q10.** What should you do BEFORE optimizing memory?

---

### 📋 Answer Key

> **Q1.** The shallow size of a single object in bytes. Does NOT include the sizes of objects it references.

> **Q2.** Python's built-in module for tracing memory allocations. Tracks where memory was allocated and how much.

> **Q3.** Line-by-line memory usage — shows how much memory each line of your function uses (increment and total).

> **Q4.** Generators produce one item at a time and discard it after use. Lists hold ALL items simultaneously.

> **Q5.** Objects that accumulate but are never freed — typically unbounded caches, circular references, or hidden references that prevent garbage collection.

> **Q6.** Eliminates the per-instance `__dict__` (a dictionary). Stores attributes in a fixed-size C struct instead.

> **Q7.** Compares two snapshots and shows what CHANGED — which allocations grew, shrank, or appeared.

> **Q8.** `del` removes the reference, but objects may not be freed if circular references exist, or if other references remain. Use `gc.collect()` to force cycle collection.

> **Q9.** `__slots__`, generators/streaming, `lru_cache(maxsize=N)`, `del` + `gc.collect()`, `array.array` for numerics, `memoryview` for zero-copy, string interning.

> **Q10.** MEASURE! Use `tracemalloc` or `memory-profiler` to find WHERE the memory goes. Don't optimize blindly.

---

## 🎬 End of Lesson Scene

**Louis:** "The server is still crashing with 8 GB!"

**Mike:** "Profiled it. The case list holds 500K records at 22 MB per 100K — that's 110 MB. But the cross-reference builds a 6 GB intermediate dict. Switching to streaming + `__slots__` cuts peak memory to 400 MB."

**Harvey:** "From 8 GB to 400 MB?"

**Mike:** "Measure first, cut what matters. Same result, 95% less memory."

**Harvey:** "Last lesson: packaging. Ship it to the world."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `sys.getsizeof` (shallow vs deep) | ✅ |
| 2 | `tracemalloc` (trace allocations) | ✅ |
| 3 | `tracemalloc` snapshots and comparison | ✅ |
| 4 | `memory-profiler` (line-by-line) | ✅ |
| 5 | Data structure memory comparison | ✅ |
| 6 | Generators vs lists for memory | ✅ |
| 7 | Finding memory leaks | ✅ |
| 8 | Memory optimization techniques | ✅ |
| 9 | Building a memory monitor | ✅ |
| 10 | Memory-efficient pipeline | ✅ |

---

> **Next Lesson →** Level 5, Lesson 9: *Package and Publish a Python Library*
