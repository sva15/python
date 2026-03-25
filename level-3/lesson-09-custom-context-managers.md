# 🏛️ Level 3 – Building | Lesson 9: Create Custom Context Managers in Python

## *"The Bespoke Contract"*

> **O'Reilly Skill Objective:** *Create custom context managers with `__enter__` and `__exit__` for reusable resource management.*

> **Previously Learned:** `with` statement, `__enter__`/`__exit__` protocol, generators, `yield`, decorators (basic), exception handling, `contextlib`

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

*The firm keeps building the same try/finally pattern everywhere.*

**Mike:** "Every database query has the same boilerplate — connect, acquire cursor, execute, commit or rollback, close cursor, close connection. Dozens of files, same 15 lines every time."

```python
# Repeated everywhere:
conn = connect()
cursor = conn.cursor()
try:
    cursor.execute(query)
    conn.commit()
except Exception:
    conn.rollback()
    raise
finally:
    cursor.close()
    conn.close()
```

**Harvey:** "If you're writing the same cleanup code in 50 places, you need a custom context manager. Write the pattern ONCE, reuse it everywhere."

**Mike:** "Two approaches — class-based with `__enter__/__exit__`, or generator-based with `@contextmanager`. Let me show both."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Two ways to build custom context managers."

| Approach | Best For | Complexity |
|----------|----------|:---:|
| **Class-based** (`__enter__`/`__exit__`) | Stateful resources, reusable objects | Medium |
| **Generator-based** (`@contextmanager`) | Simple one-shot patterns, less boilerplate | Low |

```python
# Class-based
class MyManager:
    def __enter__(self): ...
    def __exit__(self, exc_type, exc_val, exc_tb): ...

# Generator-based
from contextlib import contextmanager

@contextmanager
def my_manager():
    # setup
    yield resource
    # cleanup
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Class-Based Context Manager — The Full Protocol ⭐⭐

```python
class DatabaseConnection:
    """A class-based context manager for database operations."""
    
    def __init__(self, db_name, read_only=False):
        self.db_name = db_name
        self.read_only = read_only
        self.connection = None
        self.cursor = None
    
    def __enter__(self):
        """Set up: connect to database, return cursor."""
        print(f"  🔌 Connecting to {self.db_name}...")
        self.connection = self._connect()
        self.cursor = self.connection.cursor()
        return self.cursor    # This is what 'as' binds to
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Tear down: commit or rollback, then close."""
        if exc_type:
            # An exception occurred
            print(f"  ❌ Error: {exc_val} — rolling back")
            self.connection.rollback()
        else:
            if not self.read_only:
                print(f"  ✅ Committing changes")
                self.connection.commit()
        
        # Always close resources
        self.cursor.close()
        self.connection.close()
        print(f"  🔌 Disconnected from {self.db_name}")
        
        return False    # Don't suppress exceptions
    
    def _connect(self):
        """Simulate database connection."""
        class MockConnection:
            def cursor(self): return MockCursor()
            def commit(self): pass
            def rollback(self): pass
            def close(self): pass
        return MockConnection()

class MockCursor:
    def execute(self, q): print(f"  📝 Executing: {q}")
    def close(self): pass

# Usage — clean and safe!
with DatabaseConnection("firm_db") as cursor:
    cursor.execute("INSERT INTO clients VALUES ('Wayne', 2500000)")
    cursor.execute("INSERT INTO clients VALUES ('Stark', 800000)")
# Automatically: commit + close cursor + close connection
```

---

### 3.2 `__exit__` Parameters Explained ⭐

```python
class DetailedExit:
    """Shows what __exit__ receives."""
    
    def __enter__(self):
        print("  Entering...")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """
        exc_type: The exception CLASS (e.g., ValueError)
                  None if no exception
        exc_val:  The exception INSTANCE (e.g., ValueError("bad"))
                  None if no exception
        exc_tb:   The traceback object
                  None if no exception
        """
        if exc_type:
            print(f"  Exception caught!")
            print(f"    Type: {exc_type.__name__}")
            print(f"    Value: {exc_val}")
            print(f"    Traceback: {exc_tb}")
        else:
            print("  Clean exit — no exceptions")
        
        # Return True → suppress exception (swallow it)
        # Return False → let exception propagate (re-raise it)
        return False

# No exception
with DetailedExit():
    print("  Working normally...")
# Clean exit — no exceptions

# With exception
try:
    with DetailedExit():
        raise ValueError("Bad data!")
except ValueError:
    print("  Exception propagated (not suppressed)")
```

---

### 3.3 `@contextmanager` — Generator-Based ⭐⭐⭐

```python
from contextlib import contextmanager

@contextmanager
def database_connection(db_name):
    """Generator-based context manager — much simpler!"""
    print(f"  🔌 Connecting to {db_name}...")
    conn = {"name": db_name}    # Simulate connection
    
    try:
        yield conn                # ← The 'as' variable
    except Exception as e:
        print(f"  ❌ Error: {e} — rolling back")
        raise                     # Re-raise after handling
    else:
        print(f"  ✅ Committed")
    finally:
        print(f"  🔌 Disconnected from {db_name}")

# Usage — identical to class-based!
with database_connection("firm_db") as conn:
    print(f"  Using {conn['name']}")
# Output:
# 🔌 Connecting to firm_db...
# Using firm_db
# ✅ Committed
# 🔌 Disconnected from firm_db
```

**How `@contextmanager` maps to the protocol:**

```python
@contextmanager
def my_manager():
    # ──── __enter__ ────
    setup_code()
    resource = create_resource()
    
    try:
        yield resource       # ← value bound to 'as'
    #                        # ──── body of with block runs here ────
    except Exception:
        # ──── __exit__ with exception ────
        handle_error()
        raise
    else:
        # ──── __exit__ without exception ────
        on_success()
    finally:
        # ──── always runs ────
        cleanup(resource)
```

---

### 3.4 Practical Custom Managers — Timer, Logger, Indent

```python
from contextlib import contextmanager
import time

# === Timer (generator-based) ===
@contextmanager
def timer(label="Operation"):
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    print(f"  ⏱️ {label}: {elapsed:.4f}s")

with timer("Sorting"):
    data = sorted(range(500_000), reverse=True)

# === Indent printer ===
@contextmanager
def indent(level=1, char="  "):
    """Indent all prints within the block."""
    import builtins
    original_print = builtins.print
    prefix = char * level
    
    def indented_print(*args, **kwargs):
        original_print(prefix, *args, **kwargs)
    
    builtins.print = indented_print
    try:
        yield
    finally:
        builtins.print = original_print

print("Normal")
with indent(2):
    print("Indented twice")
    with indent(1):
        print("Indented three times total")
print("Normal again")

# === Change directory ===
import os

@contextmanager
def working_directory(path):
    """Temporarily change working directory."""
    old = os.getcwd()
    os.makedirs(path, exist_ok=True)
    os.chdir(path)
    try:
        yield path
    finally:
        os.chdir(old)

# === Temporary file with auto-cleanup ===
@contextmanager
def temp_file(suffix=".tmp", content=None):
    """Create a temp file, delete on exit."""
    import tempfile
    f = tempfile.NamedTemporaryFile(
        mode="w", suffix=suffix, delete=False
    )
    try:
        if content:
            f.write(content)
            f.flush()
        yield f.name
    finally:
        f.close()
        os.unlink(f.name)
```

---

### 3.5 Error-Handling Context Managers

```python
from contextlib import contextmanager

# === Retry manager ===
@contextmanager
def retry_on_error(max_attempts=3, exceptions=(Exception,)):
    """Retry the block up to max_attempts times."""
    for attempt in range(1, max_attempts + 1):
        try:
            yield attempt
            return    # Success — exit
        except exceptions as e:
            if attempt == max_attempts:
                print(f"  ❌ All {max_attempts} attempts failed")
                raise
            print(f"  ⚠️ Attempt {attempt} failed: {e}. Retrying...")

# === Error collector ===
@contextmanager
def collect_errors():
    """Collect errors instead of raising them."""
    errors = []
    
    class ErrorCollector:
        def attempt(self, func, *args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                errors.append(e)
                return None
        
        @property
        def error_list(self):
            return errors
    
    collector = ErrorCollector()
    yield collector
    
    if errors:
        print(f"  ⚠️ {len(errors)} errors collected:")
        for e in errors:
            print(f"    • {type(e).__name__}: {e}")

# Usage:
with collect_errors() as ec:
    ec.attempt(int, "abc")       # ValueError — collected
    ec.attempt(int, "42")        # ✅ Returns 42
    ec.attempt(lambda: 1/0)      # ZeroDivisionError — collected

print(f"Errors: {len(ec.error_list)}")  # 2
```

---

### 3.6 Reentrant and Reusable Context Managers

```python
from contextlib import contextmanager

# === Reusable: can be used multiple times ===
class ReusableTimer:
    """Can be used in multiple with blocks."""
    
    def __init__(self, label=""):
        self.label = label
        self.timings = []
    
    def __enter__(self):
        self._start = time.perf_counter()
        return self
    
    def __exit__(self, *args):
        elapsed = time.perf_counter() - self._start
        self.timings.append(elapsed)
        print(f"  ⏱️ {self.label}: {elapsed:.4f}s")
        return False

t = ReusableTimer("Sort")
with t:
    sorted(range(100_000))
with t:
    sorted(range(200_000))
print(f"Average: {sum(t.timings)/len(t.timings):.4f}s")

# === NOT reusable: @contextmanager generators are one-shot ===
@contextmanager
def one_shot():
    print("Setup")
    yield
    print("Cleanup")

cm = one_shot()
with cm:
    pass    # ✅ Works
# with cm:    # ❌ RuntimeError: generator didn't yield!
#     pass
```

---

### 3.7 Asynchronous Context Managers (Preview)

```python
# async context managers use __aenter__ and __aexit__

class AsyncDBConnection:
    async def __aenter__(self):
        self.conn = await connect_async()
        return self.conn
    
    async def __aexit__(self, *args):
        await self.conn.close()
        return False

# Usage:
# async with AsyncDBConnection() as conn:
#     await conn.execute(query)

# Generator-based async:
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_db(url):
    conn = await connect(url)
    try:
        yield conn
    finally:
        await conn.close()
```

---

### 3.8 Class-Based vs `@contextmanager` — Decision Guide

```python
"""
WHEN TO USE CLASS-BASED:
  ✅ Need to reuse the same manager multiple times
  ✅ Need to store state across uses (.timings, .count)
  ✅ Complex setup/teardown with multiple attributes
  ✅ Need to be used as a decorator AND context manager
  ✅ Need custom exception suppression logic

WHEN TO USE @contextmanager:
  ✅ Simple, one-shot resource management
  ✅ Quick prototyping
  ✅ Less boilerplate needed
  ✅ The resource lifecycle is straightforward
  ✅ You're already comfortable with generators
"""

# Example: Same functionality, both approaches

# Class-based (15 lines)
class Transact:
    def __init__(self, conn):
        self.conn = conn
    def __enter__(self):
        self.conn.begin()
        return self.conn
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.conn.rollback()
        else:
            self.conn.commit()
        return False

# Generator-based (8 lines)
@contextmanager
def transact(conn):
    conn.begin()
    try:
        yield conn
    except Exception:
        conn.rollback()
        raise
    else:
        conn.commit()
```

---

### 3.9 Solving Mike's Problem — Universal DB Manager

```python
# ============================================
# Pearson Specter Litt — Custom Context Managers
# Both class-based and generator-based approaches
# ============================================

from contextlib import contextmanager
import time
from datetime import datetime

# === Class-based: Reusable audit session ===

class AuditSession:
    """Track operations with automatic logging — reusable."""
    
    _sessions = 0
    
    def __init__(self, operator, log_file=None):
        self.operator = operator
        self.log_file = log_file
        self.operations = []
        self.session_id = None
    
    def __enter__(self):
        AuditSession._sessions += 1
        self.session_id = f"AUDIT-{AuditSession._sessions:04d}"
        self.start = datetime.now()
        self._log(f"Session started by {self.operator}")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        elapsed = (datetime.now() - self.start).total_seconds()
        
        if exc_type:
            self._log(f"❌ Session FAILED: {exc_type.__name__}: {exc_val}")
        else:
            self._log(f"✅ Session completed ({len(self.operations)} ops, {elapsed:.2f}s)")
        
        if self.log_file:
            with open(self.log_file, "a") as f:
                for entry in self.operations:
                    f.write(entry + "\n")
        
        return False    # Don't suppress exceptions
    
    def log(self, message):
        """Log an operation during this session."""
        self._log(message)
    
    def _log(self, message):
        ts = datetime.now().strftime("%H:%M:%S")
        entry = f"[{self.session_id}] [{ts}] {message}"
        self.operations.append(entry)
        print(f"  {entry}")


# === Generator-based: Quick utilities ===

@contextmanager
def performance_monitor(label, warn_threshold=1.0):
    """Monitor execution time with warning threshold."""
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    status = "⚠️ SLOW" if elapsed > warn_threshold else "✅"
    print(f"  {status} [{label}] {elapsed:.4f}s")

@contextmanager
def error_boundary(label, default=None):
    """Catch and log errors without crashing."""
    try:
        yield
    except Exception as e:
        print(f"  ❌ [{label}] {type(e).__name__}: {e}")
        if default is not None:
            print(f"  ↩️ Using default: {default}")

@contextmanager
def batch_processor(items, label="Batch"):
    """Process items in a batch with summary."""
    results = {"success": 0, "failed": 0, "total": len(items)}
    
    class Batch:
        def process(self, func):
            for item in items:
                try:
                    func(item)
                    results["success"] += 1
                except Exception:
                    results["failed"] += 1
    
    batch = Batch()
    yield batch
    
    print(f"  📊 [{label}] {results['success']}/{results['total']} succeeded, "
          f"{results['failed']} failed")


# === Demonstrate ===
print("=" * 60)
print(f"{'🏛️  CUSTOM CONTEXT MANAGERS':^60}")
print("=" * 60)

# Class-based — reusable audit session
print("\n📋 AUDIT SESSION:")
with AuditSession("Harvey Specter") as audit:
    audit.log("Created client: Wayne Enterprises")
    audit.log("Filed case: Wayne v. Stark")
    audit.log("Billed: $38,250.00")

# Generator-based — performance monitoring
print("\n⏱️ PERFORMANCE MONITOR:")
with performance_monitor("Data sort", warn_threshold=0.01):
    sorted(range(100_000))

with performance_monitor("Quick lookup", warn_threshold=0.001):
    x = 42 in range(100)

# Generator-based — error boundary
print("\n🛡️ ERROR BOUNDARIES:")
with error_boundary("Parse config"):
    config = {"port": int("abc")}    # ValueError — caught!

with error_boundary("Load data", default=[]):
    data = open("nonexistent.json")  # FileNotFoundError — caught!

# Generator-based — batch processing
print("\n📦 BATCH PROCESSING:")
items = [10, 20, "bad", 40, None, 60]
with batch_processor(items, "Revenue calc") as batch:
    batch.process(lambda x: float(x) * 100)

# Nested: audit + performance
print("\n🔄 NESTED MANAGERS:")
with AuditSession("Mike Ross") as audit:
    with performance_monitor("Client processing"):
        for name in ["Wayne", "Stark", "Oscorp"]:
            with error_boundary(f"Process {name}"):
                audit.log(f"Processed client: {name}")
                if name == "Stark":
                    raise ValueError("Missing billing data")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `@contextmanager` — Forgetting try/finally 🔥

```python
@contextmanager
def bad_manager():
    resource = acquire()
    yield resource
    release(resource)           # ❌ If body raises, this never runs!

@contextmanager
def good_manager():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)       # ✅ Always runs!
```

---

### Pitfall 2: `@contextmanager` — Yielding More Than Once

```python
@contextmanager
def bad():
    yield 1
    yield 2    # ❌ RuntimeError! Must yield exactly once!

@contextmanager
def good():
    yield 1    # ✅ Exactly one yield
```

---

### Pitfall 3: `@contextmanager` Generators Are NOT Reusable

```python
@contextmanager
def once_only():
    print("Setup")
    yield
    print("Cleanup")

cm = once_only()
with cm:
    pass    # ✅ First time works

# with cm:    # ❌ RuntimeError: generator didn't yield
#     pass    # Cannot reuse!

# ✅ Call the function again for a new generator
with once_only():
    pass    # ✅ Fresh instance
```

---

### Pitfall 4: Suppressing Exceptions in `@contextmanager`

```python
# ❌ Catching the exception without re-raising silently suppresses it
@contextmanager
def silent_suppressor():
    try:
        yield
    except Exception:
        pass    # ❌ All exceptions vanish!

# ✅ Re-raise or handle explicitly
@contextmanager
def proper_handler():
    try:
        yield
    except ValueError as e:
        print(f"Handled: {e}")
        # Don't re-raise — intentionally suppress ValueError only
    except Exception:
        raise    # ✅ Re-raise everything else
```

---

### Pitfall 5: Forgetting `return False` in Class-Based `__exit__`

```python
class Manager:
    def __enter__(self):
        return self
    def __exit__(self, *args):
        cleanup()
        # No explicit return → returns None → falsy → exceptions propagate
        # This is actually correct! But be explicit:
        return False    # ✅ Clear intention
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📝 Analogy: Custom Context Managers = Bespoke Contracts

```
BUILT-IN CONTEXT MANAGERS:
  📋 Standard form contracts (open(), Lock(), etc.)
  "One size fits most"

CUSTOM CLASS-BASED:
  📜 A full bespoke contract drafted by a partner
  - Multiple clauses (setup, resource tracking, audit, cleanup)
  - Reusable template — use it for many clients
  - Stores history and state between uses

CUSTOM @contextmanager:
  📌 A quick contract written on a legal pad
  - Setup → yield → cleanup
  - Simple, fast to write
  - One-time use, not reusable as an object

DONNA'S RULE:
  "The boilerplate you write ten times should be
   written once — as a context manager.
   Class-based for complex reusable contracts.
   @contextmanager for quick one-pagers."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Styled Output**
```
Build @contextmanager styled_output(color, bold=False):
  
with styled_output("green", bold=True):
    print("Success!")    # Prints in green bold (ANSI codes)
# Resets formatting automatically
```

**Exercise 2: Atomic File Writer**
```
Build @contextmanager atomic_write(filepath):
  - Writes to a temp file
  - On success: renames temp → target (atomic)
  - On error: deletes temp file, target unchanged
```

**Exercise 3: Logging Context**
```
Build @contextmanager log_block(label, logger):
  - Logs "Starting: {label}" on enter
  - Logs "Completed: {label}" on normal exit
  - Logs "Failed: {label}: {error}" on exception
  - Always logs elapsed time
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Rate Limiter**
```
Build a class-based RateLimiter(max_calls, period):

limiter = RateLimiter(max_calls=5, period=60)
with limiter:
    api_call()    # Blocks if rate exceeded
```

**Exercise 5: Dual-Use Timer**
```
Build a Timer that works as both context manager AND decorator:

with Timer("block"):
    ...

@Timer("function")
def my_func():
    ...
```

**Exercise 6: Nested Transaction**
```
Build a Transaction with savepoint support:

with Transaction(db) as txn:
    txn.execute("INSERT ...")
    with txn.savepoint("sp1"):
        txn.execute("INSERT ...")    # Fails
    # sp1 rolled back, outer continues
```

---

### 🔴 Advanced Exercises

**Exercise 7: Generic Resource Pool**
```
Build a ResourcePool where:
  pool = ResourcePool(factory=create_connection, size=10)
  
  with pool.acquire() as conn:
      conn.execute(query)
  # Connection returned to pool, not destroyed
```

**Exercise 8: Pipeline Context Manager**
```
Build a Pipeline where each stage is a context manager:

with Pipeline("ETL") as pipe:
    with pipe.stage("Extract") as src:
        data = src.read("input.csv")
    with pipe.stage("Transform"):
        data = transform(data)
    with pipe.stage("Load") as dst:
        dst.write("output.json", data)
# Full pipeline audit: timings, errors, data flow
```

**Exercise 9: Distributed Lock**
```
Build a DistributedLock context manager:
  - Acquires lock with timeout
  - Auto-extends lock while held
  - Releases on exit (even if crashed)
  - Supports reentrant locking
  - Logs all lock operations
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Missing yield in @contextmanager
@contextmanager
def bad1():
    setup()
    # Forgot yield! — generates: RuntimeError: generator didn't yield

# Bug 2: No try/finally — cleanup doesn't run on error
@contextmanager
def bad2():
    conn = connect()
    yield conn
    conn.close()      # ❌ Never runs if body raises!

# Bug 3: Multiple yields
@contextmanager
def bad3():
    yield "first"
    yield "second"    # ❌ RuntimeError!

# Bug 4: __enter__ returns None
class Bad4:
    def __enter__(self):
        self.data = load()
        # Forgot return!
    def __exit__(self, *a):
        return False

with Bad4() as d:
    print(d)    # None!

# Bug 5: Reusing a @contextmanager instance
cm = one_shot_manager()
with cm: pass    # ✅
with cm: pass    # ❌ RuntimeError!
```

### Fixed Code ✅

```python
# Fix 1: Add yield
@contextmanager
def good1():
    setup()
    yield           # ✅

# Fix 2: Use try/finally
@contextmanager
def good2():
    conn = connect()
    try:
        yield conn
    finally:
        conn.close()   # ✅ Always runs

# Fix 3: One yield only
@contextmanager
def good3():
    yield "only one"   # ✅

# Fix 4: Return something
class Good4:
    def __enter__(self):
        self.data = load()
        return self.data   # ✅

# Fix 5: Create new instance each time
with one_shot_manager(): pass   # ✅
with one_shot_manager(): pass   # ✅ Fresh
```

---

## 8. 🌍 Real-World Application

### Custom Context Managers in Production

| Pattern | Use Case |
|---------|----------|
| **Database transaction** | Auto commit/rollback |
| **Feature flag** | Temporary enable/disable features |
| **Profiling** | Measure code block performance |
| **Mocking** | Replace dependencies during testing |
| **Caching** | Temporary cache with auto-eviction |
| **Rate limiting** | Throttle API calls |
| **Audit logging** | Track who did what, when |
| **Environment override** | Temp env var changes |

---

## 9. ✅ Knowledge Check

---

**Q1.** What two methods define a class-based context manager?

**Q2.** What does `@contextmanager` turn a generator into?

**Q3.** How many times must a `@contextmanager` generator yield?

**Q4.** Why is `try/finally` essential inside `@contextmanager`?

**Q5.** Can a `@contextmanager` instance be reused in multiple `with` blocks?

**Q6.** What should `__exit__` return to suppress an exception?

**Q7.** When should you use class-based over `@contextmanager`?

**Q8.** What is the `as` variable bound to?

**Q9.** What happens if `__enter__` has no return statement?

**Q10.** How do you build a context manager that also works as a decorator?

---

### 📋 Answer Key

> **Q1.** `__enter__(self)` (setup, returns resource) and `__exit__(self, exc_type, exc_val, exc_tb)` (cleanup).

> **Q2.** A context manager — `yield` separates `__enter__` (before) from `__exit__` (after). The yielded value is bound to `as`.

> **Q3.** Exactly once. Zero yields → `RuntimeError: generator didn't yield`. Two+ yields → `RuntimeError: generator didn't stop`.

> **Q4.** Without it, cleanup code after `yield` doesn't run if the body raises an exception.

> **Q5.** No — generators are single-use. Call the function again for a fresh instance.

> **Q6.** `True`. Returning `False` (or not returning) lets the exception propagate.

> **Q7.** When you need reusability, stored state across uses, complex multi-attribute setup, or a combination decorator/context manager.

> **Q8.** Whatever `__enter__` returns (class-based) or whatever is yielded (`@contextmanager`).

> **Q9.** The `as` variable is `None` — a very common bug.

> **Q10.** Implement both `__enter__`/`__exit__` and `__call__` (wrapping the function in a `with self:` block).

---

## 🎬 End of Lesson Scene

**Mike:** "Two approaches. Class-based for reusable, stateful managers — audit sessions that track operations across multiple uses. Generator-based for quick patterns — timers, error boundaries, atomic writes."

**Harvey:** "The boilerplate problem?"

**Mike:** "Solved. The 15-line database pattern is now a 1-line `with database(...)` call. Written once, used in 50 places. Cleanup is guaranteed."

**Harvey:** "Next: Python's logging module. professional-grade logging for production systems."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Class-based context managers (`__enter__`/`__exit__`) | ✅ |
| 2 | `__exit__` parameters (exc_type, exc_val, exc_tb) | ✅ |
| 3 | `@contextmanager` decorator (generator-based) | ✅ |
| 4 | try/yield/finally pattern | ✅ |
| 5 | Exception handling in both approaches | ✅ |
| 6 | Reusable (class) vs one-shot (`@contextmanager`) | ✅ |
| 7 | Timer, error boundary, batch processor patterns | ✅ |
| 8 | Class vs `@contextmanager` decision guide | ✅ |
| 9 | Nested context managers | ✅ |
| 10 | Async context managers (preview) | ✅ |

---

> **Next Lesson →** Level 3, Lesson 10: *Use Python's logging Module*
