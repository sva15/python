# 🏛️ Level 5 – Innovating | Lesson 7: Optimize Python Code with timeit and cProfile

## *"The Billable Hour"*

> **O'Reilly Skill Objective:** *Optimize Python code using `timeit` and `cProfile`.*

> **Previously Learned:** Decorators, generators, list comprehensions, `__slots__`, threading, multiprocessing, design patterns

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

*The billing report script takes 45 seconds. Louis insists it ran in 5 seconds last month. Nobody knows WHERE the time goes.*

```python
# ❌ Guessing where the bottleneck is
# "I think the sort is slow..."
# "Maybe it's the database?"
# "Let me rewrite everything in C..."

# ✅ MEASURE first, then optimize
import cProfile
cProfile.run('generate_billing_report()')
# 45 seconds total:
#   - 38 seconds in database queries    ← 84% HERE!
#   - 4 seconds in string formatting
#   - 3 seconds in sorting
```

**Harvey:** "Don't guess. Measure."

**Mike:** "Python gives us `timeit` for micro-benchmarks and `cProfile` for program-wide profiling. Measure first, optimize what matters, verify the improvement."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Optimization without measurement is like litigation without evidence."

| Tool | Purpose | Granularity |
|------|---------|-------------|
| **`timeit`** | Time small code snippets | Microsecond-level |
| **`cProfile`** | Profile entire programs | Function-level |
| **`time.perf_counter`** | Manual timing | Custom sections |
| **`line_profiler`** | Profile line by line | Line-level (3rd party) |
| **`py-spy`** | Sampling profiler | No code changes (3rd party) |

**The Optimization Workflow:**
```
1. MEASURE  → Find the bottleneck
2. OPTIMIZE → Fix the bottleneck
3. VERIFY   → Confirm improvement
4. REPEAT   → Until "fast enough"
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `timeit` — Micro-Benchmarking ⭐⭐

```python
import timeit

# === Command line ===
# python -m timeit "sum(range(1000))"
# 100000 loops, best of 5: 12.3 usec per loop

# === In code — timeit.timeit() ===
# Runs the code many times, returns TOTAL time

# Compare: list comprehension vs loop
time_comp = timeit.timeit(
    '[x**2 for x in range(1000)]',
    number=10_000
)

time_loop = timeit.timeit(
    '''
result = []
for x in range(1000):
    result.append(x**2)
    ''',
    number=10_000
)

print(f"List comp:  {time_comp:.4f}s ({time_comp/10_000*1000:.3f}ms each)")
print(f"For loop:   {time_loop:.4f}s ({time_loop/10_000*1000:.3f}ms each)")
print(f"Comp is {time_loop/time_comp:.1f}x faster")

# === timeit.repeat() — multiple trials ===
times = timeit.repeat(
    'sorted(data)',
    setup='import random; data = [random.randint(0, 10000) for _ in range(1000)]',
    number=1000,
    repeat=5
)
print(f"Best of 5: {min(times):.4f}s")    # Use min, not average!

# === With setup code ===
setup = '''
clients = [{"name": f"Client_{i}", "revenue": i * 1000} for i in range(10000)]
'''

time_filter = timeit.timeit(
    '[c for c in clients if c["revenue"] > 5000000]',
    setup=setup,
    number=1000
)
print(f"Filter 10K clients: {time_filter:.4f}s over 1000 iterations")
```

---

### 3.2 Timing with `time.perf_counter` ⭐

```python
import time

# perf_counter = highest resolution timer available
# Use for timing code sections manually

def timed(func):
    """Decorator that times function execution."""
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"  ⏱️ {func.__name__}: {elapsed:.4f}s")
        return result
    return wrapper

@timed
def process_cases(n):
    return [{"id": i, "value": i ** 2} for i in range(n)]

@timed
def sort_cases(cases):
    return sorted(cases, key=lambda c: c["value"], reverse=True)

@timed
def filter_high_value(cases, threshold):
    return [c for c in cases if c["value"] > threshold]

# Pipeline with timing
cases = process_cases(100_000)
sorted_cases = sort_cases(cases)
high_value = filter_high_value(sorted_cases, 50_000)

# Output:
# ⏱️ process_cases: 0.0312s
# ⏱️ sort_cases: 0.0456s      ← Slowest!
# ⏱️ filter_high_value: 0.0089s

# === Context manager timer ===
class Timer:
    def __init__(self, label=""):
        self.label = label
    
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    
    def __exit__(self, *args):
        self.elapsed = time.perf_counter() - self.start
        print(f"  ⏱️ {self.label}: {self.elapsed:.4f}s")

with Timer("Sort 100K"):
    sorted(range(100_000), reverse=True)

with Timer("List comp"):
    [x ** 2 for x in range(100_000)]
```

---

### 3.3 `cProfile` — Program-Wide Profiling ⭐⭐

```python
import cProfile
import pstats

# === Basic profiling ===
def generate_report():
    """Simulate a complex report generation."""
    clients = load_clients()
    cases = load_cases()
    billing = calculate_billing(clients, cases)
    formatted = format_report(billing)
    return formatted

def load_clients():
    import time; time.sleep(0.1)
    return [{"name": f"Client_{i}"} for i in range(1000)]

def load_cases():
    import time; time.sleep(0.2)
    return [{"id": i, "hours": i * 10} for i in range(5000)]

def calculate_billing(clients, cases):
    import time; time.sleep(0.05)
    return [{"client": c["name"], "total": sum(range(100))} for c in clients]

def format_report(billing):
    return "\n".join(f"{b['client']}: ${b['total']}" for b in billing)

# Profile it!
cProfile.run('generate_report()')

# Output:
# 5008 function calls in 0.412 seconds
#
# ncalls  tottime  percall  cumtime  percall filename:lineno(function)
#     1    0.000    0.000    0.412    0.412 script.py:4(generate_report)
#     1    0.000    0.000    0.200    0.200 script.py:14(load_cases)       ← Bottleneck!
#     1    0.000    0.000    0.100    0.100 script.py:10(load_clients)
#  1000    0.058    0.000    0.058    0.000 script.py:18(calculate_billing)
#     1    0.004    0.004    0.004    0.004 script.py:22(format_report)

# === Save profile to file ===
cProfile.run('generate_report()', 'report.prof')

# === Analyze saved profile ===
stats = pstats.Stats('report.prof')
stats.sort_stats('cumulative')
stats.print_stats(20)          # Top 20 functions
stats.print_callers()          # Who called whom
stats.print_callees()          # What each function called
```

---

### 3.4 `cProfile` — Programmatic Use ⭐

```python
import cProfile
import pstats
import io

def profile_function(func, *args, **kwargs):
    """Profile any function and return results."""
    profiler = cProfile.Profile()
    profiler.enable()
    
    result = func(*args, **kwargs)
    
    profiler.disable()
    
    # Capture stats as string
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(15)
    
    print(stream.getvalue())
    return result

# Usage
# result = profile_function(generate_report)

# === Profile as context manager ===
class Profiler:
    def __init__(self, top_n=10, sort_by='cumulative'):
        self.top_n = top_n
        self.sort_by = sort_by
    
    def __enter__(self):
        self.profiler = cProfile.Profile()
        self.profiler.enable()
        return self
    
    def __exit__(self, *args):
        self.profiler.disable()
        stream = io.StringIO()
        stats = pstats.Stats(self.profiler, stream=stream)
        stats.sort_stats(self.sort_by)
        stats.print_stats(self.top_n)
        print(stream.getvalue())

# Usage
with Profiler(top_n=5):
    data = [x ** 2 for x in range(100_000)]
    sorted_data = sorted(data, reverse=True)
    filtered = [x for x in sorted_data if x > 1_000_000]
```

---

### 3.5 Reading cProfile Output ⭐⭐

```python
# Understanding cProfile columns:
#
# ncalls    = Number of times the function was called
# tottime   = Time spent IN this function (excluding sub-calls)
# percall   = tottime / ncalls
# cumtime   = Time spent in this function AND all functions it calls
# percall   = cumtime / ncalls
# filename:lineno(function) = Where the function is

# EXAMPLE OUTPUT:
# ncalls  tottime  percall  cumtime  percall filename:lineno(function)
#   5000    3.200    0.001    3.200    0.001 db.py:15(execute_query)
#      1    0.500    0.500    4.100    4.100 report.py:8(generate)
#  10000    0.300    0.000    0.300    0.000 utils.py:3(format_number)
#      1    0.100    0.100    4.500    4.500 main.py:1(main)
#   5000    0.050    0.000    3.250    0.001 report.py:20(get_data)

# HOW TO READ:
# 1. Sort by cumtime to find the OVERALL slowest functions
# 2. Sort by tottime to find WHERE the time is actually spent
# 3. Look at ncalls — is a function called too many times?
# 4. High tottime with low ncalls = slow function
# 5. Low tottime with high ncalls = called too often (loop?)

# Sort options:
# 'cumulative' — total time including sub-calls (find bottleneck chains)
# 'tottime'    — time in function only (find expensive functions)
# 'calls'      — number of calls (find over-called functions)
# 'name'       — alphabetical
```

---

### 3.6 Common Optimization Techniques ⭐⭐

```python
import timeit

# === 1. List comprehension vs for loop ===
# List comp is 20-30% faster (C-level loop)
setup = 'data = range(10000)'
print("List comp:", timeit.timeit('[x**2 for x in data]', setup, number=1000))
print("For loop:", timeit.timeit('''
result = []
for x in data:
    result.append(x**2)
''', setup, number=1000))

# === 2. Generator vs list for large data ===
# Generator: O(1) memory, lazy evaluation
def process_large():
    data = range(1_000_000)
    # ❌ List: creates entire list in memory
    total_list = sum([x ** 2 for x in data])
    # ✅ Generator: processes one at a time
    total_gen = sum(x ** 2 for x in data)

# === 3. dict lookup vs if/elif chain ===
setup = '''
def with_if(key):
    if key == "a": return 1
    elif key == "b": return 2
    elif key == "c": return 3
    elif key == "d": return 4
    else: return 0

lookup = {"a": 1, "b": 2, "c": 3, "d": 4}
def with_dict(key):
    return lookup.get(key, 0)
'''
print("if/elif:", timeit.timeit('with_if("d")', setup, number=100_000))
print("dict:   ", timeit.timeit('with_dict("d")', setup, number=100_000))

# === 4. Local vs global variable access ===
# Local variables are faster (LOAD_FAST vs LOAD_GLOBAL)
import math
def slow_sqrt():
    return [math.sqrt(x) for x in range(10000)]

def fast_sqrt():
    sqrt = math.sqrt    # Local reference!
    return [sqrt(x) for x in range(10000)]

# === 5. String concatenation ===
# ❌ Slow: repeated string concatenation
def slow_concat(n):
    result = ""
    for i in range(n):
        result += f"Line {i}\n"    # Creates new string each time!
    return result

# ✅ Fast: join a list
def fast_concat(n):
    return "\n".join(f"Line {i}" for i in range(n))

# === 6. Set for membership testing ===
setup = 'data = list(range(10000)); data_set = set(data)'
print("list 'in':", timeit.timeit('9999 in data', setup, number=10_000))     # O(n)
print("set 'in': ", timeit.timeit('9999 in data_set', setup, number=10_000)) # O(1)

# === 7. __slots__ for memory-heavy classes ===
# See Level 4 Lesson 6 — 40-50% memory savings

# === 8. functools.lru_cache for repeated calculations ===
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_calculation(n):
    return sum(i ** 2 for i in range(n))

# First call: calculates. Subsequent calls: instant!
```

---

### 3.7 Benchmarking Best Practices ⭐

```python
import timeit
import statistics

def benchmark(label, stmt, setup='pass', number=10_000, repeat=5):
    """Proper benchmarking with statistics."""
    times = timeit.repeat(stmt, setup=setup, number=number, repeat=repeat)
    
    best = min(times) / number * 1_000_000    # Convert to microseconds
    mean = statistics.mean(times) / number * 1_000_000
    stdev = statistics.stdev(times) / number * 1_000_000
    
    print(f"  {label:<30} {best:>8.2f}µs (best)  "
          f"{mean:>8.2f}µs (mean)  ±{stdev:.2f}µs")
    return best

print("=" * 75)
print(f"{'🏛️  OPTIMIZATION BENCHMARK':^75}")
print("=" * 75)
print()

# Run benchmarks
benchmark("List comprehension",  '[x**2 for x in range(100)]')
benchmark("Generator expression", 'sum(x**2 for x in range(100))')
benchmark("Map + lambda",        'list(map(lambda x: x**2, range(100)))')
benchmark("For loop + append",   '''
r = []
for x in range(100):
    r.append(x**2)
''')

print()
benchmark("Dict lookup", 'd.get("key", 0)', setup='d = {"key": 42}')
benchmark("Try/except", '''
try:
    v = d["key"]
except KeyError:
    v = 0
''', setup='d = {"key": 42}')

print()
benchmark("f-string",     'f"{name} has {n} cases"', setup='name="Harvey"; n=15')
benchmark("%-formatting", '"%s has %d cases" % (name, n)', setup='name="Harvey"; n=15')
benchmark("str.format",   '"{} has {} cases".format(name, n)', setup='name="Harvey"; n=15')
benchmark("+ concat",     'name + " has " + str(n) + " cases"', setup='name="Harvey"; n=15')

# RULES:
# 1. Use min(), not mean — min reflects true speed without OS noise
# 2. Run enough iterations (number) for stable results
# 3. Run enough trials (repeat) to catch outliers
# 4. Use setup to separate initialization from measurement
# 5. Compare RELATIVE differences, not absolute numbers
```

---

### 3.8 Profiling I/O and Database Calls ⭐

```python
import cProfile
import time
import functools

# === Profiling reveals I/O bottlenecks ===
# Most real applications are I/O bound, not CPU bound!

def simulate_db_query(query, delay=0.01):
    """Simulate a database query."""
    time.sleep(delay)
    return [{"id": 1, "name": "Wayne"}]

def generate_report():
    results = []
    # N+1 query problem — 100 individual queries!
    for i in range(100):
        data = simulate_db_query(f"SELECT * FROM cases WHERE id={i}")
        results.extend(data)
    return results

# Profile reveals the problem:
# cProfile.run('generate_report()')
# 100 calls to simulate_db_query totaling 1.0s
# FIX: batch query!

def generate_report_optimized():
    # ONE query, not 100!
    results = simulate_db_query("SELECT * FROM cases WHERE id IN (...)", delay=0.05)
    return results

# === Call counting decorator ===
def count_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@count_calls
def db_query(sql):
    time.sleep(0.001)
    return []

# After running:
# print(f"db_query called {db_query.calls} times")
# If calls > expected → you have an N+1 problem!
```

---

### 3.9 Visualizing Profiles with `snakeviz` ⭐

```python
# snakeviz = interactive browser-based profile viewer
# pip install snakeviz

# Step 1: Save profile
import cProfile
cProfile.run('generate_report()', 'output.prof')

# Step 2: Visualize
# python -m snakeviz output.prof
# Opens interactive sunburst diagram in browser!

# === Alternative: gprof2dot ===
# Generates call-graph images
# pip install gprof2dot
# python -m cProfile -o output.prof script.py
# gprof2dot -f pstats output.prof | dot -Tpng -o profile.png

# === py-spy: zero-overhead profiler ===
# pip install py-spy
# py-spy top -- python script.py          # Live view
# py-spy record -o profile.svg -- python script.py  # Flame graph
# No code changes needed!
```

---

### 3.10 Solving the Firm's Problem — Optimization Pipeline

```python
# ============================================
# Pearson Specter Litt — Performance Optimizer
# Find bottlenecks, apply fixes, verify results
# ============================================

import time
import timeit
import cProfile
import pstats
import io
from functools import lru_cache

# === Original (slow) implementation ===
def billing_report_slow(n_clients=500, n_cases=2000):
    """Simulate a slow billing report."""
    # Load clients (slow string building)
    clients = {}
    for i in range(n_clients):
        name = ""
        for c in f"Client_{i:04d}":
            name += c                      # 💀 String concat in loop!
        clients[name] = {"revenue": i * 1000, "tier": "Gold" if i % 3 == 0 else "Silver"}
    
    # Load cases (N+1 pattern)
    cases = []
    for cid in range(n_cases):
        client_name = f"Client_{cid % n_clients:04d}"
        if client_name in list(clients.keys()):    # 💀 list() every iteration!
            cases.append({"id": cid, "client": client_name, "hours": cid * 0.5})
    
    # Calculate billing (redundant recomputation)
    billing = []
    for case in cases:
        rate = calculate_rate_slow(case["client"])  # 💀 Recalculates every time!
        total = case["hours"] * rate
        billing.append({"case": case["id"], "total": total})
    
    # Format report (string concatenation)
    report = ""
    for entry in billing:
        report += f"Case {entry['case']}: ${entry['total']:,.2f}\n"  # 💀 Concat!
    
    return report

def calculate_rate_slow(client_name):
    """Simulate expensive rate lookup."""
    time.sleep(0.0001)    # Simulates DB call
    return 450.0

# === Optimized implementation ===
def billing_report_fast(n_clients=500, n_cases=2000):
    """Optimized billing report."""
    # Use f-string directly (no char-by-char concat)
    clients = {
        f"Client_{i:04d}": {"revenue": i * 1000, "tier": "Gold" if i % 3 == 0 else "Silver"}
        for i in range(n_clients)
    }
    
    # Direct dict membership (no list() conversion)
    cases = [
        {"id": cid, "client": f"Client_{cid % n_clients:04d}", "hours": cid * 0.5}
        for cid in range(n_cases)
        if f"Client_{cid % n_clients:04d}" in clients    # ✅ O(1) dict lookup
    ]
    
    # Cache rate lookups
    rate_cache = {}
    billing = []
    for case in cases:
        client = case["client"]
        if client not in rate_cache:
            rate_cache[client] = calculate_rate_fast(client)
        total = case["hours"] * rate_cache[client]
        billing.append(f"Case {case['id']}: ${total:,.2f}")
    
    # Join instead of concatenation
    return "\n".join(billing)

def calculate_rate_fast(client_name):
    return 450.0    # Cached/batched in real implementation


# === Benchmark ===
print("=" * 65)
print(f"{'🏛️  OPTIMIZATION RESULTS':^65}")
print("=" * 65)

# Profile the slow version
print("\n📊 PROFILING SLOW VERSION:")
profiler = cProfile.Profile()
profiler.enable()
start = time.perf_counter()
result_slow = billing_report_slow(100, 500)
slow_time = time.perf_counter() - start
profiler.disable()

stream = io.StringIO()
stats = pstats.Stats(profiler, stream=stream)
stats.sort_stats('tottime')
stats.print_stats(8)
print(stream.getvalue())

# Time the fast version
start = time.perf_counter()
result_fast = billing_report_fast(100, 500)
fast_time = time.perf_counter() - start

print(f"\n⏱️ TIMING COMPARISON:")
print(f"  Slow version: {slow_time:.4f}s")
print(f"  Fast version: {fast_time:.4f}s")
print(f"  Speedup: {slow_time/fast_time:.1f}x")

# verify same output length
print(f"\n✅ VERIFICATION:")
print(f"  Slow output lines: {len(result_slow.strip().splitlines())}")
print(f"  Fast output lines: {len(result_fast.strip().splitlines())}")

# Optimization summary
print(f"\n🔧 OPTIMIZATIONS APPLIED:")
print(f"  1. Replaced char-by-char string concat with f-strings")
print(f"  2. Removed list(dict.keys()) → direct 'in dict'")
print(f"  3. Cached rate lookups (eliminated redundant calls)")
print(f"  4. Replaced string += with '\\n'.join()")
print(f"  5. Used dict comprehension instead of loop")

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Optimizing Without Profiling 🔥

```python
# ❌ Rewriting a function that takes 0.001% of total time
# "I made this sort 50% faster!"
# But it was only 0.01s of a 45s program...

# ✅ Profile first → find the REAL bottleneck
# cProfile.run('my_program()')
# If 84% of time is in DB queries → optimize queries, not Python!
```

---

### Pitfall 2: `timeit` Measures the Wrong Thing

```python
# ❌ Including setup in the measurement
time = timeit.timeit('''
import random
data = [random.randint(0, 1000) for _ in range(10000)]
sorted(data)
''', number=100)

# ✅ Separate setup from measurement
time = timeit.timeit(
    'sorted(data)',
    setup='import random; data = [random.randint(0, 1000) for _ in range(10000)]',
    number=100
)
```

---

### Pitfall 3: Profiling Overhead Skews Results

```python
# ❌ cProfile adds overhead (10-20% slower)
# Use it to find RELATIVE bottlenecks, not absolute timing

# ✅ For accurate timing, use timeit after finding the bottleneck
# Step 1: cProfile → identify slow function
# Step 2: timeit → accurately measure that function
# Step 3: Optimize → apply fix
# Step 4: timeit → verify improvement
```

---

### Pitfall 4: Premature Optimization

```python
# ❌ "Let me optimize this before it's even working"
# Donald Knuth: "Premature optimization is the root of all evil"

# ✅ Workflow:
# 1. Make it WORK correctly
# 2. Write tests to verify correctness
# 3. Profile to find bottlenecks
# 4. Optimize the MEASURED bottleneck
# 5. Verify tests still pass
# 6. Verify improvement with benchmarks
```

---

### Pitfall 5: Micro-Optimizing When Architecture Is the Problem

```python
# ❌ Saving 5µs per loop when doing 10,000 unnecessary API calls
# Micro-optimization: list comp vs loop saves 0.001s
# Architecture fix: batch API calls saves 30s

# ✅ Priority:
# 1. Fix the algorithm (O(n²) → O(n log n))
# 2. Fix the I/O pattern (N+1 queries → batch)
# 3. Fix data structures (list → set for lookups)
# 4. THEN micro-optimize if still needed
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### ⏱️ Analogy: The Firm's Efficiency Audit

```
TIMEIT = STOPWATCH:
  Time a specific task precisely.
  "How long does Harvey's dictation take?"

CPROFILE = FULL EFFICIENCY AUDIT:
  Track EVERYTHING everyone did all day.
  "Harvey: 2h court, 3h documents, 1h meetings."
  "Mike: 30min research, 5.5h waiting for court filings!"
  ← Found the bottleneck: 5.5h waiting!

OPTIMIZATION WORKFLOW = MANAGEMENT CONSULTING:
  Don't hire more paralegals until you measure
  WHERE the time goes. Maybe the copier is slow.

DONNA'S RULE:
  "You can't fix what you can't measure.
   Measure first. Optimize the biggest waste.
   Don't reorganize the supply closet when
   the real problem is a slow database."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: timeit Comparison**
```
Compare 5 ways to create a list of squares (1-10000):
  - List comprehension
  - For loop + append
  - map() + lambda
  - Generator into list()
  - numpy (if available)
Report results in a formatted table.
```

**Exercise 2: Manual Timer Decorator**
```
Build a @timed decorator that:
  - Reports function name and execution time
  - Returns the original result unchanged
  - Works with any function signature
```

**Exercise 3: Profile a Script**
```
Write a function that creates 10,000 client records,
sorts them, filters them, and formats a report.
Profile with cProfile and identify the slowest step.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Before/After Benchmark**
```
Write "slow" and "fast" versions of the same operation:
  - String building (concat vs join)
  - Lookup (list scan vs set/dict)
  - N+1 pattern (individual vs batch)
Benchmark both, report speedup factor.
```

**Exercise 5: Profile Context Manager**
```
Build a reusable Profiler context manager:
  with Profiler("My section") as p:
      expensive_operation()
  print(p.elapsed)
  print(p.stats)    # Top 10 functions
```

**Exercise 6: Optimization Report Generator**
```
Build a tool that profiles a function and generates a report:
  report = profile_and_report(my_function, *args)
  - Total time
  - Top 5 bottleneck functions
  - Call count anomalies
  - Recommendations
```

---

### 🔴 Advanced Exercises

**Exercise 7: Comparative Benchmark Suite**
```
Build a benchmark framework:
  suite = BenchmarkSuite()
  suite.add("sorted()", lambda: sorted(data))
  suite.add("list.sort()", lambda: data.copy().sort())
  suite.run(iterations=10000)
  suite.report()    # Table with stats, winner highlighted
```

**Exercise 8: Continuous Performance Monitor**
```
Build a decorator that logs performance over time:
  @monitor_performance(threshold_ms=100)
  def process_cases(): ...
  # Logs warning when function exceeds threshold
  # Tracks historical performance trend
```

**Exercise 9: Automated Optimization Advisor**
```
Build a tool that profiles code and suggests fixes:
  advisor = OptimizationAdvisor()
  advisor.analyze(my_function)
  # Suggestions:
  #   - "Function X called 10000 times — consider caching"
  #   - "90% of time in I/O — consider async"
  #   - "List used for lookups — consider set/dict"
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Measuring wrong thing in timeit
time = timeit.timeit('import json; json.loads(data)', number=1000)
# Re-imports json 1000 times!

# Bug 2: Using time.time() instead of perf_counter
start = time.time()    # Low resolution, affected by system clock changes!
result = process()
elapsed = time.time() - start

# Bug 3: Not realizing cProfile overhead
# "My function takes 2 seconds!"
# (Actually 1.7s — cProfile added 0.3s overhead)

# Bug 4: Comparing apples to oranges
timeit.timeit('sorted(data)', number=1000)     # Sorts random data
timeit.timeit('data.sort()', number=1000)       # Sorts already-sorted data after first run!
```

### Fixed Code ✅

```python
# Fix 1: Use setup parameter
time = timeit.timeit('json.loads(data)',
    setup='import json; data = \'{"key": "value"}\'',
    number=1000)

# Fix 2: Use perf_counter
start = time.perf_counter()    # High resolution, monotonic
result = process()
elapsed = time.perf_counter() - start

# Fix 3: Use timeit for accurate absolute timing after cProfile finds bottleneck

# Fix 4: Reset data each iteration
timeit.timeit('sorted(data)',    # sorted() doesn't modify original
    setup='import random; data = [random.random() for _ in range(1000)]',
    number=1000)
```

---

## 8. 🌍 Real-World Application

### Where Profiling Matters

| Scenario | Tool | Finding |
|----------|------|---------|
| Slow API endpoint | cProfile | N+1 DB queries |
| Memory issue | memory_profiler (next lesson!) | Unbounded cache |
| Algorithm comparison | timeit | O(n²) vs O(n log n) |
| Production monitoring | py-spy | Hot path identification |
| CI regression testing | timeit + assert | Performance gate |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is `timeit` used for?

**Q2.** What is `cProfile` used for?

**Q3.** What do `tottime` and `cumtime` mean in cProfile output?

**Q4.** Why should you use `min()` on `timeit.repeat()` results?

**Q5.** What is the difference between `time.time()` and `time.perf_counter()`?

**Q6.** What does "premature optimization" mean?

**Q7.** Name three common Python optimization techniques.

**Q8.** Why shouldn't you rely on cProfile for precise timing?

**Q9.** What is the correct optimization workflow?

**Q10.** What should you fix BEFORE micro-optimizing?

---

### 📋 Answer Key

> **Q1.** Micro-benchmarking — precisely timing small code snippets. Runs code many times to get statistically reliable measurements.

> **Q2.** Profiling entire programs — shows which functions take the most time and how often they're called.

> **Q3.** `tottime` = time spent IN the function only. `cumtime` = time in the function PLUS all sub-calls.

> **Q4.** `min()` reflects the best achievable time, free from OS scheduling noise. Higher values include interference from other processes.

> **Q5.** `perf_counter` is monotonic (can't go backward), high resolution, and best for benchmarks. `time.time()` can be affected by system clock adjustments.

> **Q6.** Optimizing code before knowing if it's actually a bottleneck. "The root of all evil" — wastes effort and often makes code harder to read.

> **Q7.** List comprehensions over loops, `str.join()` over concatenation, set/dict for membership tests, `@lru_cache`, generators for large data, local variable references.

> **Q8.** cProfile adds 10-20% overhead. Use it to find RELATIVE bottlenecks, then use `timeit` for accurate absolute timing.

> **Q9.** Measure → Find bottleneck → Optimize → Verify improvement → Repeat.

> **Q10.** Algorithm complexity (O(n²) → O(n log n)), I/O patterns (N+1 → batch), data structures (list → set for lookups). Architecture before micro-optimization.

---

## 🎬 End of Lesson Scene

**Louis:** "The billing report is still at 45 seconds!"

**Mike:** "Profiled it. 84% of the time — 38 seconds — is in database queries. Not the sort, not the formatting. There are 5,000 individual queries where one batch query would work."

**Louis:** "So the sort I rewrote three times..."

**Mike:** "Saved 0.1 seconds. The query optimization saved 36 seconds. Always profile first."

**Harvey:** "Next: memory. Speed isn't the only bottleneck."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `timeit` — micro-benchmarking | ✅ |
| 2 | `time.perf_counter` — manual timing | ✅ |
| 3 | `cProfile` — program-wide profiling | ✅ |
| 4 | Reading cProfile output columns | ✅ |
| 5 | Programmatic profiling (Profile class) | ✅ |
| 6 | Common optimization techniques | ✅ |
| 7 | Benchmarking best practices | ✅ |
| 8 | Profiling I/O and database calls | ✅ |
| 9 | Visualization tools (snakeviz, py-spy) | ✅ |
| 10 | Optimization workflow and pipeline | ✅ |

---

> **Next Lesson →** Level 5, Lesson 8: *Monitor Memory Usage in Python with memory-profiler*
