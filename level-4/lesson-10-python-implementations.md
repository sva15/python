# 🏛️ Level 4 – Advancing | Lesson 10: Compare Python Implementations (CPython, PyPy, etc.)

## *"Under the Hood"*

> **O'Reilly Skill Objective:** *Compare Python implementations such as CPython, PyPy, Jython, etc. Understand the GIL and how different implementations handle concurrency.*

> **Previously Learned:** Decorators, closures, inheritance, `__slots__`, iterators, generators, ORM, regex, context managers

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

*The firm's document analysis script processes 50,000 legal files. It takes 45 minutes.*

**Louis:** "45 minutes! Can't we make Python faster?"

**Mike:** "Depends. The Python you're running — CPython — interprets bytecode line by line. But there are other engines: PyPy uses a JIT compiler and could cut that to 10 minutes. For numeric heavy-lifting, Cython compiles to C."

**Harvey:** "So Python isn't just one thing?"

**Mike:** "Python is the LANGUAGE. CPython, PyPy, Jython, Cython — those are the ENGINES. Same fuel, different horsepower."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Python is a specification. The implementation is the machine that runs it."

| Implementation | Written In | Key Feature | Primary Use |
|---------------|:----------:|------------|------------|
| **CPython** | C | Reference implementation, GIL | Default — what you've been using |
| **PyPy** | Python/RPython | JIT compiler, 4-10x faster | CPU-heavy Python code |
| **Cython** | C extension | Compile Python → C | Numeric/scientific computing |
| **Jython** | Java | Runs on JVM | Java integration |
| **IronPython** | C# | Runs on .NET/CLR | .NET integration |
| **MicroPython** | C | Tiny footprint | Embedded/IoT devices |
| **GraalPy** | Java | GraalVM polyglot | Multi-language environments |
| **Stackless** | C | Microthreads (tasklets) | Massively concurrent systems |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 CPython — The Default Engine ⭐⭐

```python
# CPython is what you get when you type 'python' or 'python3'
# It IS Python for 95%+ of developers

import sys
import platform

print(f"Implementation: {platform.python_implementation()}")  # CPython
print(f"Version: {sys.version}")
print(f"Compiler: {platform.python_compiler()}")

# How CPython executes your code:
#
# 1. SOURCE CODE ──→ 2. BYTECODE ──→ 3. EXECUTION
#    (.py file)        (.pyc file)     (CPython VM)
#
# Step 1: Parser reads your .py file
# Step 2: Compiler converts to bytecode (stored in __pycache__)
# Step 3: Virtual machine interprets bytecode ONE INSTRUCTION AT A TIME

# You can inspect bytecode:
import dis

def calculate_billing(hours, rate):
    return hours * rate

dis.dis(calculate_billing)
# LOAD_FAST    0 (hours)
# LOAD_FAST    1 (rate)
# BINARY_MULTIPLY
# RETURN_VALUE
```

**CPython characteristics:**

| Feature | Detail |
|---------|--------|
| **Speed** | Moderate — interprets bytecode |
| **C extensions** | Full support (`numpy`, `pandas`, `PIL`) |
| **GIL** | Global Interpreter Lock (one thread at a time) |
| **Memory** | Reference counting + garbage collection |
| **Ecosystem** | 100% package compatibility (pip/PyPI) |
| **Debugging** | Best tooling (pdb, cProfile, etc.) |

---

### 3.2 The Global Interpreter Lock (GIL) ⭐⭐⭐

```python
# The GIL is CPython's most important (and controversial) feature
# It allows ONLY ONE thread to execute Python bytecode at a time

import threading
import time

counter = 0

def increment(n):
    global counter
    for _ in range(n):
        counter += 1    # NOT thread-safe despite GIL!

# Two threads running "simultaneously"
t1 = threading.Thread(target=increment, args=(1_000_000,))
t2 = threading.Thread(target=increment, args=(1_000_000,))

t1.start(); t2.start()
t1.join(); t2.join()

print(f"Expected: 2,000,000")
print(f"Got:      {counter:,}")    # Less than 2M! Race condition!

# WHY THE GIL EXISTS:
# 1. Simplifies CPython's memory management (reference counting)
# 2. Makes C extensions easier to write
# 3. Single-threaded code runs faster with GIL vs fine-grained locks
# 4. Historical: was a reasonable trade-off in 1992

# GIL IMPLICATIONS:
"""
╔═══════════════════════════════════════════════════════╗
║  CPU-BOUND TASKS:                                     ║
║    Threading = NO speedup (GIL blocks parallelism)    ║
║    Multiprocessing = YES speedup (separate processes) ║
║                                                       ║
║  I/O-BOUND TASKS:                                     ║
║    Threading = YES speedup (GIL released during I/O)  ║
║    asyncio = YES speedup (cooperative multitasking)   ║
╚═══════════════════════════════════════════════════════╝
"""

# Demonstrate: GIL blocks CPU parallelism
def cpu_work():
    total = 0
    for i in range(10_000_000):
        total += i
    return total

# Single-threaded
start = time.perf_counter()
cpu_work()
cpu_work()
single = time.perf_counter() - start

# Multi-threaded (NOT faster due to GIL!)
start = time.perf_counter()
t1 = threading.Thread(target=cpu_work)
t2 = threading.Thread(target=cpu_work)
t1.start(); t2.start()
t1.join(); t2.join()
threaded = time.perf_counter() - start

print(f"\nCPU-bound benchmark:")
print(f"  Single-threaded: {single:.3f}s")
print(f"  Multi-threaded:  {threaded:.3f}s")  # ~Same or slower!
print(f"  Speedup: {single/threaded:.2f}x")
```

---

### 3.3 Bypassing the GIL ⭐⭐

```python
import multiprocessing
import time

def cpu_work(n):
    """CPU-bound task."""
    total = 0
    for i in range(n):
        total += i
    return total

# === Multiprocessing — BYPASSES GIL ===
# Each process has its OWN Python interpreter and GIL!

if __name__ == '__main__':
    N = 10_000_000
    
    # Single process
    start = time.perf_counter()
    cpu_work(N)
    cpu_work(N)
    single = time.perf_counter() - start
    
    # Multi-process (true parallelism!)
    start = time.perf_counter()
    with multiprocessing.Pool(2) as pool:
        results = pool.map(cpu_work, [N, N])
    multi = time.perf_counter() - start
    
    print(f"Single process: {single:.3f}s")
    print(f"Multi-process:  {multi:.3f}s")    # ~2x faster!
    print(f"Speedup: {single/multi:.2f}x")

# === Python 3.13+: Free-threaded mode (no GIL!) ===
# python3.13t  — experimental "free-threaded" build
# Run with: python3.13t -X gil=0 script.py
# Or: PYTHON_GIL=0 python3.13t script.py

# === Other GIL bypasses ===
# 1. C extensions: numpy, pandas release GIL during computation
# 2. ctypes/cffi: C code runs without GIL
# 3. asyncio: cooperative multitasking for I/O
# 4. PyPy: has GIL but plans for STM (Software Transactional Memory)
```

---

### 3.4 PyPy — The JIT-Compiled Engine ⭐⭐

```python
# PyPy uses a Just-In-Time (JIT) compiler
# It observes which code is "hot" and compiles it to machine code at runtime

# === Speed comparison ===
# CPython: interprets bytecode → one instruction at a time → moderate
# PyPy: detects hot loops → compiles to machine code → FAST

# Typical speedups: 4-10x for pure Python code

# === Example: same code, different engines ===
# Save as benchmark.py and run with both:
#   python benchmark.py     (CPython)
#   pypy benchmark.py       (PyPy)

import time

def fibonacci(n):
    """CPU-intensive recursive Fibonacci."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

def matrix_sum(size):
    """CPU-intensive nested loop."""
    total = 0
    for i in range(size):
        for j in range(size):
            total += i * j
    return total

# Benchmark
start = time.perf_counter()
fibonacci(35)
elapsed_fib = time.perf_counter() - start

start = time.perf_counter()
matrix_sum(3000)
elapsed_matrix = time.perf_counter() - start

import platform
impl = platform.python_implementation()
print(f"\n{impl} Benchmark:")
print(f"  fibonacci(35):    {elapsed_fib:.3f}s")
print(f"  matrix_sum(3000): {elapsed_matrix:.3f}s")

# Typical results:
# CPython:  fibonacci=3.2s,  matrix=1.8s
# PyPy:     fibonacci=0.4s,  matrix=0.2s  ← 8-9x faster!
```

**When to use PyPy:**

| Scenario | CPython | PyPy |
|----------|:-------:|:----:|
| Pure Python computation | ⚠️ Slow | ✅ Fast |
| C extensions (numpy, pandas) | ✅ Full support | ⚠️ Limited |
| Web servers (pure Python) | OK | ✅ Faster |
| Short-lived scripts | ✅ Fast startup | ⚠️ Slow startup (JIT warmup) |
| Long-running services | OK | ✅ JIT pays off over time |
| Memory-heavy workloads | OK | ✅ Often uses less memory |

---

### 3.5 Cython — Compiling Python to C ⭐

```python
# Cython compiles Python-like code to C, then to native machine code
# Used heavily in numpy, pandas, scikit-learn, lxml

# === Pure Python (slow) ===
def calculate_billing_python(attorneys, hours_list, rates_list):
    total = 0.0
    for i in range(len(attorneys)):
        total += hours_list[i] * rates_list[i]
    return total

# === Cython equivalent (billing.pyx) ===
"""
# billing.pyx — Cython file
def calculate_billing_cython(list attorneys, list hours_list, list rates_list):
    cdef double total = 0.0        # C type declaration
    cdef int i
    cdef int n = len(attorneys)
    cdef double hours, rate
    
    for i in range(n):
        hours = hours_list[i]
        rate = rates_list[i]
        total += hours * rate
    
    return total
"""

# Compile: cython billing.pyx → billing.c → billing.so
# Or use setup.py with Extension

# Speed: 10-100x faster for numeric code!
# Because: no Python overhead, direct C operations, no GIL for typed code
```

---

### 3.6 Jython, IronPython, and Other Implementations ⭐

```python
# === Jython: Python on the JVM ===
"""
- Written in Java
- Runs on Java Virtual Machine
- Access to ALL Java libraries
- No GIL (uses JVM threading)
- ⚠️ Currently Python 2.7 only
- Use case: Java ecosystem integration

# Example Jython code:
from java.util import ArrayList
jlist = ArrayList()
jlist.add("Harvey")
jlist.add("Mike")
"""

# === IronPython: Python on .NET ===
"""
- Written in C#
- Runs on .NET CLR
- Access to .NET libraries
- No GIL (uses CLR threading)
- ⚠️ Currently Python 3.4 compatible
- Use case: .NET/Windows integration

# Example IronPython code:
import clr
clr.AddReference("System.Windows.Forms")
from System.Windows.Forms import MessageBox
MessageBox.Show("Hello from Python!")
"""

# === MicroPython: Python for microcontrollers ===
"""
- Extremely small footprint (~256KB Flash, ~16KB RAM)
- Runs on: Raspberry Pi Pico, ESP32, STM32, etc.
- Subset of Python 3 standard library
- Hardware access: GPIO, I2C, SPI, UART
- Use case: IoT, embedded systems, robotics

# Example MicroPython code:
from machine import Pin
import time

led = Pin(25, Pin.OUT)
while True:
    led.toggle()
    time.sleep(0.5)
"""

# === GraalPy: Python on GraalVM ===
"""
- Part of the GraalVM polyglot ecosystem
- Can interop with Java, JavaScript, Ruby, R, LLVM
- JIT compiled for performance
- Growing Python 3.x compatibility
- Use case: polyglot applications, enterprise
"""

# === Stackless Python ===
"""
- Fork of CPython with microthreads (tasklets)
- Thousands of lightweight threads without OS overhead
- Used by: EVE Online (game server), many game engines
- Use case: massively concurrent applications
"""
```

---

### 3.7 CPython Internals — Reference Counting and Memory ⭐

```python
import sys

# === Reference counting ===
# CPython tracks how many references point to each object
# When count hits 0 → object is immediately deallocated

a = [1, 2, 3]
print(sys.getrefcount(a))    # 2 (a + the arg to getrefcount)

b = a                         # Another reference
print(sys.getrefcount(a))    # 3

c = [a, a, a]                 # Three more references
print(sys.getrefcount(a))    # 6

del b                         # Remove one reference
print(sys.getrefcount(a))    # 5

# === Garbage collector ===
# Handles CIRCULAR references that refcount can't catch
import gc

class Node:
    def __init__(self, name):
        self.name = name
        self.partner = None

a = Node("Harvey")
b = Node("Mike")
a.partner = b    # a → b
b.partner = a    # b → a (circular!)

del a, b          # refcount > 0 due to cycle!
gc.collect()      # Garbage collector detects and cleans cycles

# GC stats
print(f"GC collections: {gc.get_stats()}")
print(f"GC thresholds: {gc.get_threshold()}")

# === Memory optimization ===
# Python interns small integers and short strings
a = 256
b = 256
print(a is b)     # True — same object! (integers -5 to 256)

x = 257
y = 257
print(x is y)     # May be True or False (depends on context)

# String interning
s1 = "hello"
s2 = "hello"
print(s1 is s2)    # True — interned!

s3 = "hello world!"
s4 = "hello world!"
print(s3 is s4)    # False (but may be True in some contexts)
# Use sys.intern() to force interning
```

---

### 3.8 Python 3.13+ and the Future (No-GIL, JIT) ⭐⭐

```python
# === Python 3.11: Faster CPython (10-60% speedup) ===
# - Specialized adaptive interpreter
# - Zero-cost exception handling
# - Inlined function calls

# === Python 3.12: Even faster + per-interpreter GIL ===
# - Sub-interpreters with separate GILs
# - Better error messages
# - f-string improvements

# === Python 3.13: Free-threaded Python (PEP 703) ===
"""
EXPERIMENTAL: Build Python without the GIL!

Install: python3.13t (the 't' means free-threaded)
Run: python3.13t -X gil=0 script.py

import threading, time

def cpu_work(n):
    total = 0
    for i in range(n):
        total += i
    return total

# With free-threaded Python, threads achieve TRUE parallelism!
threads = [threading.Thread(target=cpu_work, args=(10**7,)) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
# Actually ~4x faster on 4 cores!
"""

# === Python 3.13: Experimental JIT compiler ===
"""
CPython now has an experimental copy-and-patch JIT!
Run: python3.13 -X jit script.py

Not as fast as PyPy's JIT, but a step toward a faster CPython.
"""

# === Future roadmap ===
"""
Python's speed evolution:
  3.10  →  Baseline
  3.11  →  10-60% faster (adaptive interpreter)
  3.12  →  Further optimizations, per-interpreter GIL
  3.13  →  Experimental: no-GIL + JIT
  3.14+ →  GIL removal stabilization, JIT maturation
  
GOAL: Python "fast enough" that you rarely need C extensions.
"""
```

---

### 3.9 Choosing the Right Implementation

```python
"""
DECISION GUIDE:

┌─────────────────────────────────┐
│ What are you building?          │
├─────────────────────────────────┤
│ General app / web / data?       │ → CPython (default)
│ CPU-heavy pure Python?          │ → PyPy (4-10x speedup)
│ Numeric/scientific computing?   │ → CPython + NumPy/Cython
│ Need Java libraries?            │ → Jython
│ Need .NET libraries?            │ → IronPython
│ Embedded / IoT?                 │ → MicroPython
│ Polyglot (Java+Python+JS)?      │ → GraalPy
│ Maximum concurrency?            │ → Stackless / free-threaded CPython
│ Need ALL pip packages?          │ → CPython (only 100% compatible)
└─────────────────────────────────┘

FOR 99% OF DEVELOPERS:
  Use CPython. It's the default, has the best ecosystem,
  and is "fast enough" for most applications.

  IF your bottleneck is pure Python computation:
    → Try PyPy first (drop-in replacement for many apps)
    → Profile before optimizing (the bottleneck may not be Python)
"""
```

---

### 3.10 Benchmarking: Choosing Based on Evidence

```python
# ============================================
# Pearson Specter Litt — Implementation Comparison
# Measure, don't guess!
# ============================================

import time
import platform
import sys

def benchmark(label, func, *args, iterations=5):
    """Run a function multiple times, report average time."""
    times = []
    for _ in range(iterations):
        start = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - start
        times.append(elapsed)
    
    avg = sum(times) / len(times)
    return {"label": label, "avg": avg, "min": min(times), "max": max(times)}

# === Benchmark functions ===
def fibonacci(n):
    if n < 2: return n
    return fibonacci(n - 1) + fibonacci(n - 2)

def string_concat(n):
    result = ""
    for i in range(n):
        result += f"Attorney #{i}: billed ${i * 450:,}\n"
    return result

def list_operations(n):
    data = list(range(n))
    filtered = [x for x in data if x % 2 == 0]
    mapped = [x ** 2 for x in filtered]
    total = sum(mapped)
    return total

def dict_operations(n):
    clients = {f"Client_{i}": i * 1000 for i in range(n)}
    high_value = {k: v for k, v in clients.items() if v > n * 500}
    total = sum(high_value.values())
    return total

def class_creation(n):
    class Record:
        __slots__ = ('name', 'value')
        def __init__(self, name, value):
            self.name = name
            self.value = value
    records = [Record(f"item_{i}", i) for i in range(n)]
    return len(records)


# === Run benchmarks ===
impl = platform.python_implementation()
version = platform.python_version()

print("=" * 65)
print(f"{'🏛️  PYTHON IMPLEMENTATION BENCHMARK':^65}")
print(f"{'Engine: ' + impl + ' ' + version:^65}")
print("=" * 65)

benchmarks = [
    benchmark("fibonacci(30)", fibonacci, 30),
    benchmark("string_concat(10K)", string_concat, 10_000, iterations=3),
    benchmark("list_operations(100K)", list_operations, 100_000),
    benchmark("dict_operations(50K)", dict_operations, 50_000),
    benchmark("class_creation(100K)", class_creation, 100_000),
]

print(f"\n  {'Benchmark':<30} {'Avg':>10} {'Min':>10} {'Max':>10}")
print(f"  {'─'*30} {'─'*10} {'─'*10} {'─'*10}")

for b in benchmarks:
    print(f"  {b['label']:<30} {b['avg']:>9.4f}s {b['min']:>9.4f}s {b['max']:>9.4f}s")

total = sum(b['avg'] for b in benchmarks)
print(f"\n  {'TOTAL':<30} {total:>9.4f}s")

# Memory info
print(f"\n📊 SYSTEM INFO:")
print(f"  Implementation: {impl}")
print(f"  Version: {version}")
print(f"  Compiler: {platform.python_compiler()}")
print(f"  Platform: {platform.platform()}")
print(f"  Int max str digits: {sys.get_int_max_str_digits()}")

# Concurrency model
print(f"\n🔒 GIL STATUS:")
if hasattr(sys, '_is_gil_enabled'):
    print(f"  GIL enabled: {sys._is_gil_enabled()}")
else:
    print(f"  GIL: present (standard CPython)")
    print(f"  For CPU parallelism: use multiprocessing")
    print(f"  For I/O parallelism: use threading or asyncio")

print(f"\n{'='*65}")

# === Profiling tips ===
"""
BEFORE choosing a different implementation, PROFILE:

1. cProfile — function-level profiling
   python -m cProfile -s cumtime script.py

2. line_profiler — line-level profiling
   @profile
   def my_function(): ...
   kernprof -l -v script.py

3. memory_profiler — memory usage
   @profile
   def my_function(): ...
   python -m memory_profiler script.py

4. py-spy — sampling profiler (no code changes!)
   py-spy top -- python script.py
   py-spy record -o profile.svg -- python script.py

5. timeit — micro-benchmarks
   python -m timeit "sum(range(1000))"

RULE: "Profile first, optimize second."
Do NOT switch implementations based on gut feeling!
"""
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Assuming Threading = Parallelism 🔥

```python
import threading

# ❌ Threads DON'T speed up CPU-bound work in CPython!
def crunch_numbers():
    return sum(i * i for i in range(10_000_000))

# This is NOT faster than single-threaded:
threads = [threading.Thread(target=crunch_numbers) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()

# ✅ Use multiprocessing for CPU parallelism
import multiprocessing
with multiprocessing.Pool(4) as pool:
    results = pool.map(crunch_numbers, range(4))    # Actually parallel!
```

---

### Pitfall 2: Switching to PyPy When the Bottleneck Is I/O

```python
# ❌ PyPy won't help if your code is slow because of:
# - Database queries
# - Network requests
# - File I/O
# - Sleep/wait

# ✅ Profile first!
# If 90% of time is in database calls, PyPy gives ~0% improvement
# Fix the queries instead: indexes, caching, connection pooling
```

---

### Pitfall 3: PyPy + C Extensions Incompatibility

```python
# ❌ Some C extensions don't work with PyPy
# import numpy     # ⚠️ May work via cpyext but slower
# import pandas    # ⚠️ Limited support
# import lxml      # ⚠️ Compatibility issues

# ✅ Check compatibility before switching:
# https://pypy.org/compat.html

# Pure Python packages work fine:
# requests, flask, django, sqlalchemy → all work with PyPy
```

---

### Pitfall 4: `is` vs `==` and Integer Interning

```python
# CPython interns small integers (-5 to 256)
a = 100
b = 100
print(a is b)    # True in CPython — SAME object!

a = 1000
b = 1000
print(a is b)    # False in CPython! (may differ in PyPy)

# ⚠️ NEVER use 'is' for value comparison!
# ✅ Always use '==' for comparing values
print(a == b)    # True ✅
```

---

### Pitfall 5: Relying on CPython-Specific Behavior

```python
# ❌ __del__ timing depends on implementation
class Resource:
    def __del__(self):
        print("Resource freed")    # When is this called?

r = Resource()
del r    # CPython: freed immediately (refcount → 0)
         # PyPy: freed later (GC cycle)
         # Jython: freed whenever JVM's GC runs

# ✅ Use context managers for deterministic cleanup
class Resource:
    def __enter__(self):
        return self
    def __exit__(self, *args):
        self.cleanup()
    def cleanup(self):
        print("Resource freed")    # Deterministic!

with Resource() as r:
    pass    # cleanup() called here, guaranteed ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏎️ Analogy: Implementations = Car Engines

```
PYTHON (the language) = THE ROAD MAP
  "Get from client intake to case resolution."
  The map is the same for everyone.

CPython = RELIABLE SEDAN (Toyota Camry)
  ✅ Gets you everywhere
  ✅ Every gas station supports it (pip packages)
  ✅ Well-maintained, reliable
  ⚠️ Not the fastest

PyPy = SPORTS CAR (Porsche)
  ✅ Much faster on open highway (CPU-bound)
  ⚠️ Not every gas station works (C extensions)
  ⚠️ Takes a moment to warm up (JIT)

Cython = CUSTOM ENGINE SWAP
  ✅ Blazing fast for specific tasks
  ⚠️ Requires mechanical knowledge (C types)
  ⚠️ More maintenance

MicroPython = MOTORCYCLE
  ✅ Fits in tiny spaces (embedded)
  ⚠️ Can't carry as much (limited stdlib)

DONNA'S RULE:
  "Use the sedan. It goes everywhere.
   If you PROVE the sedan is too slow — and only then —
   consider the sports car.
   But CHECK if a better route (algorithm) is the real fix."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Identify Your Engine**
```
Write a script that prints:
  - Implementation name
  - Version
  - Compiler
  - Platform
  - GIL status (if detectable)
Run it with CPython. If available, run with PyPy.
```

**Exercise 2: Bytecode Inspector**
```
Write 5 different functions (arithmetic, loop, list comp,
dictionary, class creation).
Use dis.dis() to inspect the bytecode of each.
Which generates the most bytecode?
```

**Exercise 3: Reference Counter**
```
Create objects and watch sys.getrefcount():
  - Create an object → refcount
  - Add references → refcount increases
  - Delete references → refcount decreases
  - Demonstrate circular reference
```

---

### 🟡 Intermediate Exercises

**Exercise 4: GIL Demonstrator**
```
Write a CPU-bound function and an I/O-bound function.
Time each with:
  a) Single-threaded
  b) Multi-threaded (threading)
  c) Multi-process (multiprocessing)
Show: threading helps I/O but NOT CPU.
```

**Exercise 5: Benchmark Suite**
```
Build a benchmark that tests 5 operations:
  - Fibonacci (recursive)
  - String manipulation
  - List comprehension vs loop
  - Dictionary operations
  - Generator vs list
Format results as a table. Design for cross-implementation comparison.
```

**Exercise 6: Memory Profiler**
```
Compare memory usage of:
  - list vs tuple
  - dict vs namedtuple
  - Regular class vs __slots__
  - List vs generator
Use sys.getsizeof and tracemalloc.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Thread vs Process Pool**
```
Build a task dispatcher:
  - Auto-detect if task is CPU or I/O bound
  - Routes CPU tasks to ProcessPoolExecutor
  - Routes I/O tasks to ThreadPoolExecutor
  - Reports timing for each approach
```

**Exercise 8: Cross-Implementation Test Suite**
```
Build tests that validate behavior across implementations:
  - Integer interning boundaries
  - __del__ timing
  - Thread safety
  - Memory layout differences
  - Garbage collection behavior
```

**Exercise 9: Performance Report Generator**
```
Build a script that:
  1. Runs a benchmark suite
  2. Detects the implementation
  3. Generates a formatted report with:
     - System info
     - Benchmark results
     - Bottleneck identification
     - Recommendations (PyPy? multiprocessing? Cython?)
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Using 'is' for comparison
x = 1000
y = 1000
if x is y:    # May be False! Implementation-specific!
    print("Equal")

# Bug 2: Expecting threading speedup for CPU work
import threading
def heavy():
    return sum(range(10**7))

threads = [threading.Thread(target=heavy) for _ in range(4)]
# NOT faster than single-threaded!

# Bug 3: Relying on __del__ for cleanup
class DBConn:
    def __del__(self):
        self.close()    # May never be called in PyPy!

# Bug 4: Assuming sys.getrefcount works in PyPy
import sys
x = [1, 2, 3]
sys.getrefcount(x)    # Meaningless in PyPy (no refcounting)!
```

### Fixed Code ✅

```python
# Fix 1: Use == for value comparison
if x == y:    # ✅ Always correct
    print("Equal")

# Fix 2: Use multiprocessing for CPU work
import multiprocessing
with multiprocessing.Pool(4) as pool:
    results = pool.starmap(heavy, [()] * 4)    # ✅ True parallelism

# Fix 3: Use context manager
class DBConn:
    def __enter__(self): return self
    def __exit__(self, *args): self.close()    # ✅ Deterministic

# Fix 4: Check implementation first
import platform
if platform.python_implementation() == "CPython":
    print(f"Refcount: {sys.getrefcount(x)}")
else:
    print("Refcount not available (non-CPython)")
```

---

## 8. 🌍 Real-World Application

### Who Uses What?

| Company/Project | Implementation | Why |
|----------------|:--------------:|-----|
| **Instagram** | CPython + Cython | Web app + performance-critical paths |
| **Dropbox** | CPython → PyPy | Switched for server performance |
| **Netflix** | CPython | Data pipelines, recommendation engine |
| **EVE Online** | Stackless Python | Thousands of concurrent game entities |
| **Raspberry Pi** | MicroPython | Hardware control and education |
| **NumPy/SciPy** | CPython + Cython | Scientific computing |
| **Django** | CPython (PyPy compatible) | Web framework |
| **Trading firms** | CPython + Cython/C | Microsecond latency |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is CPython?

**Q2.** What is the GIL?

**Q3.** Why doesn't threading speed up CPU-bound Python code?

**Q4.** How do you achieve true CPU parallelism in CPython?

**Q5.** What is PyPy and when should you use it?

**Q6.** What is Cython?

**Q7.** What is reference counting?

**Q8.** What does `dis.dis()` show?

**Q9.** What is Python 3.13's free-threaded mode?

**Q10.** What should you ALWAYS do before switching implementations?

---

### 📋 Answer Key

> **Q1.** The default, reference implementation of Python, written in C. It compiles Python source to bytecode and executes it on a virtual machine.

> **Q2.** The Global Interpreter Lock — a mutex that allows only one thread to execute Python bytecode at a time in CPython. Simplifies memory management but limits CPU parallelism.

> **Q3.** The GIL allows only one thread to run Python bytecode at a time. Multiple threads take turns rather than running simultaneously on multiple cores.

> **Q4.** Use the `multiprocessing` module — each process has its own interpreter and GIL. Or use C extensions that release the GIL.

> **Q5.** A Python implementation with a JIT compiler. 4-10x faster than CPython for pure Python code. Use when CPU-bound and your dependencies are PyPy-compatible.

> **Q6.** A compiler that translates Python-like code (with C type annotations) to C, then compiles to native machine code. Used in numpy, pandas, scikit-learn.

> **Q7.** CPython's primary memory management strategy. Each object tracks how many references point to it. When the count reaches zero, the object is immediately freed.

> **Q8.** The Python bytecode (intermediate representation) of a function. Useful for understanding what CPython's compiler produces.

> **Q9.** An experimental CPython build (`python3.13t`) that removes the GIL, allowing true multi-threaded parallelism for Python code.

> **Q10.** PROFILE! Use cProfile, line_profiler, or py-spy to identify the actual bottleneck. The problem may be algorithmic, I/O-bound, or in a specific library — not the Python implementation.

---

## 🎬 End of Lesson Scene

**Louis:** "So the answer to 'Python is slow' is…"

**Mike:** "Which Python? CPython interprets bytecode, PyPy JIT-compiles it, Cython compiles to C. The language is the same — the engine determines the speed. And most 'slow Python' problems aren't about the engine — they're about the algorithm, the database query, or the I/O pattern."

**Harvey:** "Profile first. Optimize second. Switch engines last."

**Mike:** "Exactly. And with Python 3.13 removing the GIL experimentally, the speed story is changing fast."

**Harvey:** "Level 4 complete. We've gone from decorators to database ORMs to the engine itself. Next level: the architecture that makes it all production-ready."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | CPython: reference implementation, bytecode VM | ✅ |
| 2 | The GIL: what it is, why it exists, implications | ✅ |
| 3 | Bypassing the GIL: multiprocessing, free-threaded 3.13 | ✅ |
| 4 | PyPy: JIT compiler, 4-10x speedup | ✅ |
| 5 | Cython: Python → C compilation | ✅ |
| 6 | Jython, IronPython, MicroPython, GraalPy, Stackless | ✅ |
| 7 | CPython internals: reference counting, GC, interning | ✅ |
| 8 | Python 3.13+: no-GIL, JIT, future roadmap | ✅ |
| 9 | Choosing the right implementation | ✅ |
| 10 | Profiling before optimizing | ✅ |

---

> **🎉 Level 4 Complete!** You've mastered advanced Python patterns from decorators and closures through inheritance hierarchies, memory optimization, regex, ORMs, and the Python engine itself.
>
> **Next Level →** Level 5: *Innovating — Production Patterns, Async, Metaclasses, and Beyond*
