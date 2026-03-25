# 🏛️ Level 3 – Building | Lesson 8: Use Context Managers in Python

## *"The Retainer Agreement"*

> **O'Reilly Skill Objective:** *Use a context manager to safely manage resources (e.g., file I/O).*

> **Previously Learned:** File I/O, classes, `__init__`, exception handling, `finally`, generators, `yield`, dunder methods

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

*A production outage. The server ran out of file handles.*

**Louis:** "The billing script opens files to process invoices. But when an error occurs mid-processing, the file never closes. After 1,000 errors, we ran out of file handles and the entire server froze."

```python
# The culprit:
f = open("invoices.csv")
data = f.read()
process(data)           # If this raises, f.close() never runs!
f.close()
```

**Harvey:** "Every resource you open — files, connections, locks — must be closed. No exceptions. No excuses."

**Mike:** "Context managers. The `with` statement guarantees cleanup — even when errors occur. It's like a retainer agreement: the terms are clear, the exit is guaranteed."

```python
# The fix:
with open("invoices.csv") as f:
    data = f.read()
    process(data)
# f.close() is called automatically — even if process() crashes!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A context manager is a cleanup contract."

| Without `with` | With `with` |
|:---:|:---:|
| You open the resource | `with` opens it |
| You MUST remember to close | `with` closes automatically |
| Error? Resource leaks! | Error? Still closes! |
| Multiple resources = nested cleanup | `with` handles all of it |

**The `with` statement protocol:**

```python
with expression as variable:
    # Setup: __enter__() is called
    # variable = what __enter__() returns
    use(variable)
    # Teardown: __exit__() is called — ALWAYS
    # Even on exception, return, or break!
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 File I/O — The Classic Context Manager ⭐

```python
# ❌ Without context manager — risky!
f = open("data.txt", "w")
f.write("Hello!")
# If code crashes here, file never closes!
f.close()

# ❌ Better, but verbose
f = open("data.txt", "w")
try:
    f.write("Hello!")
finally:
    f.close()

# ✅ Context manager — clean, safe, Pythonic
with open("data.txt", "w") as f:
    f.write("Hello!")
# f is closed automatically, no matter what!

# Verify:
print(f.closed)    # True
```

---

### 3.2 Multiple Resources with `with` ⭐

```python
# Multiple context managers in one with statement
with open("input.txt", "r") as inp, open("output.txt", "w") as out:
    for line in inp:
        out.write(line.upper())
# BOTH files are closed automatically!

# Python 3.10+ — parenthesized version (multi-line)
with (
    open("input.txt", "r") as inp,
    open("output.txt", "w") as out,
    open("log.txt", "a") as log,
):
    data = inp.read()
    out.write(data.upper())
    log.write(f"Processed {len(data)} chars\n")
```

---

### 3.3 Common Built-in Context Managers ⭐

```python
# 1. Files
with open("data.txt") as f:
    content = f.read()

# 2. Threading locks
import threading
lock = threading.Lock()
with lock:
    # Thread-safe section — lock acquired
    shared_data.append(item)
# Lock released automatically

# 3. Temporary directory
import tempfile
with tempfile.TemporaryDirectory() as tmpdir:
    print(f"Temp dir: {tmpdir}")
    # Use tmpdir...
# Directory and contents are deleted automatically!

# 4. Temporary file
with tempfile.NamedTemporaryFile(mode="w", suffix=".csv", delete=False) as f:
    f.write("name,revenue\n")
    f.write("Wayne,2500000\n")
    print(f"Temp file: {f.name}")

# 5. Database connections
import sqlite3
with sqlite3.connect("firm.db") as conn:
    conn.execute("INSERT INTO clients VALUES (?, ?)", ("Wayne", 2500000))
# Connection committed and closed!

# 6. Suppressing specific exceptions
from contextlib import suppress
with suppress(FileNotFoundError):
    os.remove("temp.txt")    # No error if file doesn't exist!

# 7. Decimal precision
from decimal import Decimal, localcontext
with localcontext() as ctx:
    ctx.prec = 50
    result = Decimal(1) / Decimal(7)
    print(result)    # 50 decimal places
# Precision restored to default outside with block

# 8. Changing directory (custom)
import os
class chdir:
    def __init__(self, path):
        self.path = path
        self.old = os.getcwd()
    def __enter__(self):
        os.chdir(self.path)
        return self
    def __exit__(self, *args):
        os.chdir(self.old)
        return False
```

---

### 3.4 How `with` Works Under the Hood ⭐⭐

```python
# with open("file.txt") as f:
#     f.read()
#
# Is equivalent to:

manager = open("file.txt")           # Create the context manager
f = manager.__enter__()               # Call __enter__, bind result to f

try:
    f.read()                          # Execute the body
except Exception:
    # Pass exception info to __exit__
    if not manager.__exit__(*sys.exc_info()):
        raise                          # Re-raise if __exit__ returns False
else:
    manager.__exit__(None, None, None) # No exception — still call __exit__

# __enter__() → called at start, return value bound to 'as' variable
# __exit__(exc_type, exc_val, exc_tb) → called at end, always
#   → Return True to suppress the exception
#   → Return False to let it propagate
```

---

### 3.5 `contextlib` Utilities ⭐

```python
from contextlib import suppress, redirect_stdout, redirect_stderr
import io

# suppress — silence specific exceptions
with suppress(FileNotFoundError, PermissionError):
    os.remove("maybe_missing.txt")
# No error, no crash

# redirect_stdout — capture print output
f = io.StringIO()
with redirect_stdout(f):
    print("This goes to the buffer")
    print("Not to the screen")
captured = f.getvalue()
print(f"Captured: {captured!r}")

# redirect_stderr — capture error output
err = io.StringIO()
with redirect_stderr(err):
    import warnings
    warnings.warn("Test warning")
```

---

### 3.6 Nesting and Combining Context Managers

```python
from contextlib import ExitStack

# ExitStack — manage a dynamic number of context managers
def process_files(file_list):
    """Open an arbitrary number of files safely."""
    with ExitStack() as stack:
        files = [
            stack.enter_context(open(f))
            for f in file_list
        ]
        # All files are open
        for f in files:
            print(f"  {f.name}: {len(f.read())} chars")
    # ALL files closed, regardless of errors!

# ExitStack with callbacks
with ExitStack() as stack:
    stack.callback(print, "Cleanup 1: done")
    stack.callback(print, "Cleanup 2: done")
    print("Working...")
# Output:
# Working...
# Cleanup 2: done    ← LIFO order (last registered, first called)
# Cleanup 1: done
```

---

### 3.7 Timing and Profiling Context Manager

```python
import time

class Timer:
    """Measure execution time with a context manager."""
    
    def __init__(self, label="Operation"):
        self.label = label
        self.elapsed = 0
    
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    
    def __exit__(self, *args):
        self.elapsed = time.perf_counter() - self.start
        print(f"  ⏱️ {self.label}: {self.elapsed:.4f}s")
        return False

# Usage
with Timer("Sorting"):
    data = sorted(range(1_000_000), reverse=True)

with Timer("Searching"):
    result = 999_999 in data

# Nested timing
with Timer("Total Pipeline"):
    with Timer("  Load"):
        data = list(range(100_000))
    with Timer("  Process"):
        result = [x ** 2 for x in data]
    with Timer("  Save"):
        _ = str(result)
```

---

### 3.8 Real-World Patterns

```python
# Pattern 1: Transaction — commit or rollback
class Transaction:
    def __init__(self, db):
        self.db = db
    
    def __enter__(self):
        self.db.begin()
        return self.db
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.db.rollback()
            print(f"  ❌ Rolled back: {exc_val}")
        else:
            self.db.commit()
            print("  ✅ Committed")
        return False    # Don't suppress exceptions

# Pattern 2: Temporary state change
class TemporaryAttribute:
    """Temporarily change an attribute, restore on exit."""
    
    def __init__(self, obj, attr, value):
        self.obj = obj
        self.attr = attr
        self.value = value
    
    def __enter__(self):
        self.original = getattr(self.obj, self.attr)
        setattr(self.obj, self.attr, self.value)
        return self.obj
    
    def __exit__(self, *args):
        setattr(self.obj, self.attr, self.original)
        return False

class Config:
    debug = False

config = Config()
print(config.debug)          # False
with TemporaryAttribute(config, "debug", True):
    print(config.debug)      # True (temporarily)
print(config.debug)          # False (restored)

# Pattern 3: Resource pool
class ConnectionPool:
    def __init__(self, size=5):
        self.pool = [f"conn_{i}" for i in range(size)]
        self.in_use = []
    
    def __enter__(self):
        if not self.pool:
            raise RuntimeError("No connections available!")
        conn = self.pool.pop()
        self.in_use.append(conn)
        return conn
    
    def __exit__(self, *args):
        conn = self.in_use.pop()
        self.pool.append(conn)    # Return to pool
        return False

pool = ConnectionPool(3)
with pool as conn1:
    with pool as conn2:
        print(f"  Using: {conn1}, {conn2}")
        print(f"  Available: {len(pool.pool)}")
    print(f"  After releasing conn2: {len(pool.pool)} available")
print(f"  After releasing conn1: {len(pool.pool)} available")
```

---

### 3.9 Solving the Server Problem — Resource-Safe Billing

```python
# ============================================
# Pearson Specter Litt — Resource-Safe Billing
# Context managers for guaranteed cleanup
# ============================================

import time
import os
import json
from contextlib import ExitStack, suppress
from datetime import datetime

class Timer:
    def __init__(self, label=""):
        self.label = label
        self.elapsed = 0
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    def __exit__(self, *args):
        self.elapsed = time.perf_counter() - self.start
        return False

class AuditLogger:
    """Log all operations to an audit file."""
    
    def __init__(self, filepath):
        self.filepath = filepath
        self.entries = []
    
    def __enter__(self):
        self.start = datetime.now()
        self.entries.append(f"[{self.start:%H:%M:%S}] Audit session started")
        return self
    
    def log(self, message):
        ts = datetime.now().strftime("%H:%M:%S")
        self.entries.append(f"[{ts}] {message}")
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.entries.append(f"[ERROR] {exc_type.__name__}: {exc_val}")
        
        self.entries.append(f"[{datetime.now():%H:%M:%S}] Session ended")
        
        with open(self.filepath, "a") as f:
            f.write("\n".join(self.entries) + "\n\n")
        
        return False    # Don't suppress exceptions

class DataProcessor:
    """Process data files with guaranteed cleanup."""
    
    def __init__(self, input_path, output_path):
        self.input_path = input_path
        self.output_path = output_path
        self.records_processed = 0
        self.errors = []
    
    def __enter__(self):
        self._input = open(self.input_path, "r")
        self._output = open(self.output_path, "w")
        return self
    
    def process_line(self, line):
        """Process a single line — may raise."""
        parts = line.strip().split(",")
        if len(parts) < 3:
            raise ValueError(f"Incomplete record: {line.strip()!r}")
        
        name, hours, rate = parts[0], float(parts[1]), float(parts[2])
        amount = hours * rate
        self._output.write(f"{name},{amount:.2f}\n")
        self.records_processed += 1
        return name, amount
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self._input.close()
        self._output.close()
        
        if exc_type:
            # Clean up partial output on error
            with suppress(FileNotFoundError):
                os.remove(self.output_path)
        
        return False


# === Create test data ===
test_input = "billing_input.csv"
with open(test_input, "w") as f:
    f.write("Wayne,85,450\n")
    f.write("Stark,62,400\n")
    f.write("Oscorp,55,425\n")
    f.write("Bad Record\n")           # Will cause error
    f.write("Dalton,70,450\n")

# === Run with full resource management ===
print("=" * 55)
print(f"{'🏛️  RESOURCE-SAFE BILLING':^55}")
print("=" * 55)

with Timer("Total Processing") as timer:
    with AuditLogger("audit.log") as audit:
        audit.log("Starting billing pipeline")
        
        total = 0
        successes = 0
        failures = 0
        
        with open(test_input, "r") as f:
            for i, line in enumerate(f, 1):
                try:
                    # Process each line independently
                    parts = line.strip().split(",")
                    if len(parts) != 3:
                        raise ValueError(f"Expected 3 fields, got {len(parts)}")
                    
                    name = parts[0]
                    amount = float(parts[1]) * float(parts[2])
                    total += amount
                    successes += 1
                    audit.log(f"✅ {name}: ${amount:,.2f}")
                    
                except (ValueError, IndexError) as e:
                    failures += 1
                    audit.log(f"❌ Line {i}: {e}")
        
        audit.log(f"Complete: {successes} ok, {failures} failed, ${total:,.2f} total")

print(f"\n  ✅ Processed: {successes}")
print(f"  ❌ Errors: {failures}")
print(f"  💰 Total: ${total:,.2f}")
print(f"  ⏱️  Time: {timer.elapsed:.4f}s")

# All resources guaranteed closed:
# ✅ Input file closed
# ✅ Audit log written and closed
# ✅ Timer completed

# Cleanup test files
with suppress(FileNotFoundError):
    os.remove(test_input)
    os.remove("audit.log")

print(f"\n{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Using the Resource After `with` Exits

```python
with open("data.txt", "w") as f:
    f.write("Hello")

# ❌ File is closed after the with block!
# f.write("More")    # ValueError: I/O operation on closed file!

# The variable f still exists, but the resource is closed
print(f.closed)    # True
print(f.name)      # "data.txt" — metadata still accessible
```

---

### Pitfall 2: Suppressing Exceptions Accidentally

```python
class BadManager:
    def __enter__(self):
        return self
    def __exit__(self, *args):
        return True    # ⚠️ Suppresses ALL exceptions!

with BadManager():
    raise ValueError("This error vanishes!")
# No error raised! Bug is hidden!

# ✅ Always return False unless you intentionally suppress
class GoodManager:
    def __exit__(self, *args):
        return False    # ✅ Let exceptions propagate
```

---

### Pitfall 3: `__enter__` Returns the Wrong Thing

```python
class FileProcessor:
    def __init__(self, path):
        self.path = path
    
    def __enter__(self):
        self.file = open(self.path)
        return self.file    # ✅ Return the file, not self!
    
    def __exit__(self, *args):
        self.file.close()
        return False

# ❌ Common mistake: forgetting return in __enter__
class BadProcessor:
    def __enter__(self):
        self.file = open("data.txt")
        # Forgot return! 'as f' will be None!
    
    def __exit__(self, *args):
        self.file.close()
        return False

with BadProcessor() as f:
    print(f)    # None!  ← forgot to return
```

---

### Pitfall 4: Not Handling `__exit__` Errors

```python
# ❌ If __exit__ raises, the original exception is lost!
class RiskyCleanup:
    def __enter__(self):
        return self
    def __exit__(self, *args):
        raise RuntimeError("Cleanup failed!")  # ← replaces original error!

# ✅ Protect cleanup code
class SafeCleanup:
    def __exit__(self, *args):
        try:
            self.cleanup()
        except Exception:
            pass    # Log but don't mask the original error
        return False
```

---

### Pitfall 5: Nested Context Managers — Resource Leak on Inner Failure

```python
# ❌ If second open() fails, first file is NOT closed!
f1 = open("file1.txt")
f2 = open("nonexistent.txt")    # FileNotFoundError — f1 leaks!

# ✅ Use multiple with or ExitStack
with open("file1.txt") as f1:
    with open("file2.txt") as f2:
        pass    # If file2 fails, file1 is still cleaned up!

# ✅ Or ExitStack for dynamic resources
from contextlib import ExitStack
with ExitStack() as stack:
    f1 = stack.enter_context(open("file1.txt"))
    f2 = stack.enter_context(open("file2.txt"))  # If fails, f1 closes!
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📜 Analogy: Context Manager = Retainer Agreement

```
WITHOUT CONTEXT MANAGER (handshake deal):
  "I'll open this file, do some work, and close it when I'm done"
  → What if you forget? What if an error occurs?
  → Resource leak. No cleanup. Chaos.

WITH CONTEXT MANAGER (signed retainer):
  📜 RETAINER AGREEMENT
  
  Section 1 (__enter__):
    "Upon opening this document, the following resources
     shall be allocated..."
  
  Section 2 (body):
    "The client may use said resources for..."
  
  Section 3 (__exit__):
    "Upon completion OR termination for any reason,
     including errors, exceptions, or early returns,
     all resources SHALL be released.
     THIS CLAUSE IS NON-NEGOTIABLE."

DONNA'S RULE:
  "If it opens, it must close. If it acquires, it must release.
   The 'with' statement is the ONLY guarantee."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Safe File Copy**
```
Write safe_copy(src, dst) using context managers:
  - Opens both files with one with statement
  - Copies content from src to dst
  - Both files guaranteed closed even on error
```

**Exercise 2: Word Counter**
```
Write count_words(filepath) using with:
  - Returns a dict of word → count
  - Handles FileNotFoundError gracefully
  - File always closed
```

**Exercise 3: JSON Safe Load/Save**
```
Write json_load(path, default=None) and json_save(path, data):
  - Both use context managers
  - json_load returns default if file missing or invalid
  - json_save writes atomically (temp file + rename)
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Timer Decorator + Context Manager**
```
Build a Timer that works BOTH ways:

@Timer("my_func")
def my_func(): ...

with Timer("block"):
    ...

Should log elapsed time in both cases.
```

**Exercise 5: Temporary Environment Variables**
```
Build temp_env(**kwargs) context manager:

with temp_env(DATABASE_URL="test://...", DEBUG="1"):
    print(os.environ["DEBUG"])    # "1"
print(os.environ.get("DEBUG"))   # None (restored)
```

**Exercise 6: Resource Pool**
```
Build a generic ResourcePool:
  - Takes a factory function and pool size
  - with pool.acquire() as resource: ...
  - Resources returned to pool on exit
  - Blocks or raises if pool exhausted
```

---

### 🔴 Advanced Exercises

**Exercise 7: Multi-Phase Transaction**
```
Build a transaction with savepoints:

with Transaction(db) as txn:
    txn.execute("INSERT ...")
    with txn.savepoint():
        txn.execute("INSERT ...")    # May fail
    # Savepoint rolled back, outer continues
    txn.execute("INSERT ...")        # Still runs
# Commit if no unhandled exception
```

**Exercise 8: Progress Bar Context Manager**
```
Build a progress context manager:

with ProgressBar(total=100, label="Processing") as pb:
    for i in range(100):
        process(items[i])
        pb.update(1)    # Displays: Processing [████████░░] 80%
```

**Exercise 9: Managed Pipeline**
```
Build a pipeline where each stage is a context manager:

with Pipeline() as pipe:
    pipe.add_stage(FileReader("input.csv"))
    pipe.add_stage(Parser())
    pipe.add_stage(Transformer())
    pipe.add_stage(FileWriter("output.json"))
    pipe.execute()
# All stages cleaned up in reverse order
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: File never closes on error
f = open("data.txt")
data = f.read()
process(data)      # Raises! f.close() never reached!
f.close()

# Bug 2: Missing return in __enter__
class Manager:
    def __enter__(self):
        self.resource = acquire()
        # Forgot return!

with Manager() as r:
    print(r)    # None!

# Bug 3: __exit__ suppresses all exceptions
class BadManager:
    def __enter__(self): return self
    def __exit__(self, *a): return True   # Swallows everything!

with BadManager():
    1/0    # ZeroDivisionError silently suppressed!

# Bug 4: Resource used after with block
with open("x.txt", "w") as f:
    f.write("data")
f.write("more")    # ValueError: I/O operation on closed file!

# Bug 5: Nested opens without context manager
f1 = open("a.txt")
f2 = open("nonexistent.txt")   # f1 leaks if this fails!
```

### Fixed Code ✅

```python
# Fix 1: Use with
with open("data.txt") as f:
    data = f.read()
    process(data)

# Fix 2: Return the resource
def __enter__(self):
    self.resource = acquire()
    return self.resource    # ✅ or return self

# Fix 3: Return False
def __exit__(self, *a):
    return False    # ✅ Let exceptions propagate

# Fix 4: Keep work inside with block
with open("x.txt", "w") as f:
    f.write("data")
    f.write("more")    # ✅ Inside the block

# Fix 5: Nested with statements
with open("a.txt") as f1:
    with open("b.txt") as f2:
        pass    # ✅ Both cleaned up properly
```

---

## 8. 🌍 Real-World Application

### Context Managers in Production

| Resource | Context Manager |
|----------|----------------|
| **Files** | `open()` |
| **DB connections** | `sqlite3.connect()`, `psycopg2.connect()` |
| **HTTP sessions** | `requests.Session()` |
| **Thread locks** | `threading.Lock()` |
| **Temp files/dirs** | `tempfile.NamedTemporaryFile()` |
| **Network sockets** | `socket.socket()` |
| **Subprocess** | `subprocess.Popen()` |
| **Decimal context** | `decimal.localcontext()` |
| **Mock patching** | `unittest.mock.patch()` |

---

## 9. ✅ Knowledge Check

---

**Q1.** What problem does the `with` statement solve?

**Q2.** What two methods must a context manager implement?

**Q3.** What does `__enter__` return?

**Q4.** When does `__exit__` run?

**Q5.** What do `__exit__`'s three parameters represent?

**Q6.** What should `__exit__` return to let exceptions propagate?

**Q7.** What is `ExitStack` used for?

**Q8.** What happens if you use a file object after the `with` block exits?

**Q9.** How do you suppress a specific exception with `contextlib`?

**Q10.** Can you use multiple context managers in one `with` statement?

---

### 📋 Answer Key

> **Q1.** Guaranteed resource cleanup — even when exceptions occur. No more leaked files, connections, or locks.

> **Q2.** `__enter__(self)` (setup) and `__exit__(self, exc_type, exc_val, exc_tb)` (cleanup).

> **Q3.** The value bound to the `as` variable. Often `self` or the managed resource.

> **Q4.** Always — after the `with` block completes normally, after an exception, after a `return`, or after a `break`.

> **Q5.** `exc_type`: exception class (or None). `exc_val`: exception instance. `exc_tb`: traceback object.

> **Q6.** `False` (or any falsy value). Returning `True` suppresses the exception.

> **Q7.** Managing a dynamic number of context managers — opens N resources and guarantees all are closed.

> **Q8.** `ValueError: I/O operation on closed file` — the file is closed but the variable still exists.

> **Q9.** `from contextlib import suppress; with suppress(FileNotFoundError): ...`

> **Q10.** Yes — `with open("a") as f1, open("b") as f2:` or parenthesized form in Python 3.10+.

---

## 🎬 End of Lesson Scene

**Louis:** "Zero leaked file handles. Every file, every connection, every lock — guaranteed closed."

**Mike:** "The `with` statement. `__enter__` sets it up, `__exit__` tears it down. Works even when exceptions fire, even on early returns. It's the retainer agreement of Python — the cleanup clause is non-negotiable."

**Harvey:** "Next: building your own context managers from scratch."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `with` statement syntax and purpose | ✅ |
| 2 | `__enter__` and `__exit__` protocol | ✅ |
| 3 | File I/O with context managers | ✅ |
| 4 | Multiple resources in one `with` | ✅ |
| 5 | Built-in context managers (files, locks, temp) | ✅ |
| 6 | Exception handling in `__exit__` | ✅ |
| 7 | `contextlib.suppress` and `redirect_stdout` | ✅ |
| 8 | `ExitStack` for dynamic resources | ✅ |
| 9 | Timer, Transaction, Pool patterns | ✅ |
| 10 | Resource safety guarantees | ✅ |

---

> **Next Lesson →** Level 3, Lesson 9: *Create Custom Context Managers in Python*
