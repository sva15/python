# 🏛️ Level 3 – Building | Lesson 5: Handle Exceptions with Try-Except in Python

## *"The Contingency Plan"*

> **O'Reilly Skill Objective:** *Use try-except blocks to handle exceptions gracefully. Handle specific exception types to provide targeted error handling. Use else and finally blocks in try-except structures to manage resources and cleanup.*

> **Previously Learned:** Functions, classes, `@property`, file I/O, JSON, `assert`, debugging, logging, data structures

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

*Monday morning. The billing system processes overnight files from 20 clients. It crashes at client #7.*

**Louis:** "The first 6 clients processed fine. Client 7 had a missing field. The entire script crashed. Clients 8 through 20 were never processed."

**Harvey:** "So one bad file killed the entire pipeline?"

**Mike:** "That's what happens without exception handling. One error stops everything. We need contingency plans — try the operation, and if it fails, handle the failure gracefully without killing the entire process."

**Jessica:** "In this firm, we don't let one bad case shut down the entire practice. Handle it, log it, move on."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Every good lawyer has a contingency plan."

| Concept | Analogy |
|---------|---------|
| `try` | "Attempt this risky operation" |
| `except` | "If it fails, here's Plan B" |
| `else` | "If it succeeds, proceed with this" |
| `finally` | "No matter what happens, do this cleanup" |
| `raise` | "I'm deliberately filing a complaint" |

```python
try:
    # Attempt the risky operation
    result = process(data)
except SomeError as e:
    # Handle the specific failure
    log(f"Failed: {e}")
else:
    # Runs ONLY if try succeeded (no exception)
    save(result)
finally:
    # ALWAYS runs — success or failure
    cleanup()
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Try-Except ⭐

```python
# Without exception handling — program crashes!
# number = int("abc")    # ValueError: invalid literal

# With exception handling — graceful recovery!
try:
    number = int("abc")
except ValueError:
    print("That's not a valid number!")
    number = 0

print(f"Number: {number}")    # Number: 0

# Real example: file reading
try:
    with open("missing_file.txt") as f:
        data = f.read()
except FileNotFoundError:
    print("File not found — using default data")
    data = "default"
```

---

### 3.2 Catching the Exception Object ⭐

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error type: {type(e).__name__}")    # ZeroDivisionError
    print(f"Error message: {e}")                 # division by zero
    result = 0

# The exception object contains:
# - The error message (str(e))
# - The type (type(e))
# - The traceback (e.__traceback__)
```

---

### 3.3 Catching Multiple Exception Types ⭐⭐

```python
def safe_divide(a, b):
    """Handle multiple things that can go wrong."""
    try:
        a = float(a)
        b = float(b)
        result = a / b
    except ValueError as e:
        print(f"  Invalid input: {e}")
        return None
    except ZeroDivisionError:
        print("  Cannot divide by zero!")
        return None
    except TypeError as e:
        print(f"  Type error: {e}")
        return None
    else:
        print(f"  Result: {result}")
        return result

safe_divide("10", "3")     # Result: 3.333...
safe_divide("abc", "3")    # Invalid input
safe_divide("10", "0")     # Cannot divide by zero!
safe_divide(None, "3")     # Type error

# Catching multiple types in one except:
try:
    value = process(data)
except (ValueError, TypeError, KeyError) as e:
    print(f"Data error: {e}")

# Order matters! Most specific first:
try:
    value = risky_operation()
except FileNotFoundError:            # Specific
    print("File not found")
except OSError:                       # Broader (parent class)
    print("OS-level error")
except Exception:                     # Broadest (catch-almost-all)
    print("Something went wrong")
```

---

### 3.4 The `else` Clause — Success Handler ⭐

```python
def load_client_data(filepath):
    """else runs ONLY when try succeeds — keeps success logic separate."""
    try:
        with open(filepath) as f:
            data = f.read()
    except FileNotFoundError:
        print(f"  ❌ File not found: {filepath}")
        return None
    except PermissionError:
        print(f"  ❌ Permission denied: {filepath}")
        return None
    else:
        # Only runs if no exception occurred!
        # This is the "happy path"
        print(f"  ✅ Loaded {len(data)} characters")
        return data

# Why not put it in the try block?
# Because you ONLY want to catch exceptions from the file open,
# not from the processing code. else keeps them separate.
```

**When to use `else`:**

| Pattern | When |
|---------|------|
| `try` only | "Just don't crash" |
| `try` + `except` | "Handle specific failures" |
| `try` + `except` + `else` | "Handle failure AND separate success logic" |
| `try` + `finally` | "Always clean up" |
| `try` + `except` + `else` + `finally` | Full control |

---

### 3.5 The `finally` Clause — Always Runs ⭐⭐

```python
def process_file(filepath):
    """finally ALWAYS runs — even after return, break, or exception."""
    f = None
    try:
        f = open(filepath)
        data = f.read()
        return data
    except FileNotFoundError:
        print("File not found")
        return None
    finally:
        # This runs no matter what:
        # - After successful return
        # - After exception handling
        # - Even after unhandled exceptions!
        if f:
            f.close()
            print("  File closed (finally)")

result = process_file("test.txt")
# "File closed (finally)" — always prints!

# finally with resource management
def database_query(query):
    conn = connect_to_database()
    try:
        result = conn.execute(query)
        return result
    except DatabaseError as e:
        print(f"Query failed: {e}")
        return None
    finally:
        conn.close()    # ALWAYS close the connection!
```

---

### 3.6 The Full Structure — All Four Clauses

```python
import json

def load_client(filepath):
    """
    try:     Attempt the risky operation
    except:  Handle specific failures
    else:    Execute on success only
    finally: Always cleanup
    """
    print(f"\n--- Loading: {filepath} ---")
    
    try:
        with open(filepath, "r") as f:
            raw = f.read()
        data = json.loads(raw)
    
    except FileNotFoundError:
        print(f"  ❌ File not found: {filepath}")
        return None
    
    except json.JSONDecodeError as e:
        print(f"  ❌ Invalid JSON: {e}")
        return None
    
    except PermissionError:
        print(f"  ❌ No permission to read: {filepath}")
        return None
    
    else:
        # Success! Process the data
        print(f"  ✅ Loaded client: {data.get('name', 'Unknown')}")
        return data
    
    finally:
        # Always runs — logging, metrics, etc.
        print(f"  📋 Processing complete for: {filepath}")
```

---

### 3.7 Raising Exceptions — `raise` ⭐⭐

```python
def validate_billing(hours, rate, client_name):
    """Raise exceptions for invalid data."""
    
    if not isinstance(hours, (int, float)):
        raise TypeError(f"Hours must be numeric, got {type(hours).__name__}")
    
    if hours < 0:
        raise ValueError(f"Hours cannot be negative: {hours}")
    
    if hours > 24:
        raise ValueError(f"Cannot bill more than 24 hours per day: {hours}")
    
    if rate <= 0:
        raise ValueError(f"Rate must be positive: {rate}")
    
    if not client_name:
        raise ValueError("Client name cannot be empty")
    
    return hours * rate

# Using it:
try:
    total = validate_billing(85, 450, "Wayne")
    print(f"Total: ${total:,.2f}")
except (TypeError, ValueError) as e:
    print(f"Validation error: {e}")

# Re-raising exceptions
try:
    result = risky_operation()
except ValueError as e:
    print(f"Logging error: {e}")
    raise    # Re-raise the SAME exception (preserves traceback)

# Raising with chained cause
try:
    value = int(user_input)
except ValueError as original:
    raise RuntimeError(f"Invalid config value: '{user_input}'") from original
```

---

### 3.8 Custom Exceptions ⭐

```python
# Define your own exception hierarchy

class FirmError(Exception):
    """Base exception for all firm-related errors."""
    pass

class BillingError(FirmError):
    """Errors related to billing."""
    pass

class ClientError(FirmError):
    """Errors related to client operations."""
    pass

class ValidationError(ClientError):
    """Data validation failures."""
    def __init__(self, field, value, message):
        self.field = field
        self.value = value
        self.message = message
        super().__init__(f"Validation error on '{field}': {message} (got: {value!r})")

class InsufficientFundsError(BillingError):
    """Client has insufficient retainer."""
    def __init__(self, client, required, available):
        self.client = client
        self.required = required
        self.available = available
        super().__init__(
            f"{client}: needs ${required:,.2f} but only ${available:,.2f} available"
        )

# Usage:
def process_payment(client, amount, retainer):
    if amount > retainer:
        raise InsufficientFundsError(client, amount, retainer)
    return retainer - amount

try:
    remaining = process_payment("Wayne", 50000, 30000)
except InsufficientFundsError as e:
    print(f"  ❌ {e}")
    print(f"  Shortfall: ${e.required - e.available:,.2f}")
except BillingError:
    print("  Some billing error occurred")
except FirmError:
    print("  Some firm error occurred")
```

**Exception hierarchy:**

```
BaseException
 └── Exception                    ← Catch with: except Exception
      ├── ValueError
      ├── TypeError
      ├── KeyError
      ├── FileNotFoundError
      ├── FirmError               ← Your custom base
      │    ├── BillingError
      │    │    └── InsufficientFundsError
      │    └── ClientError
      │         └── ValidationError
      └── ...
```

---

### 3.9 The Exception Hierarchy — What to Catch

```python
# === Common built-in exceptions ===

# ValueError — right type, wrong value
int("abc")           # ValueError

# TypeError — wrong type for operation
"hello" + 42         # TypeError

# KeyError — dict key doesn't exist
d = {}; d["missing"] # KeyError

# IndexError — list index out of range
lst = [1]; lst[5]    # IndexError

# AttributeError — object doesn't have that attribute
"hello".nonexistent  # AttributeError

# FileNotFoundError — file doesn't exist
open("nope.txt")     # FileNotFoundError

# ZeroDivisionError
10 / 0               # ZeroDivisionError

# ImportError / ModuleNotFoundError
# import nonexistent_module    # ModuleNotFoundError

# PermissionError — no access rights
# open("/etc/shadow", "w")     # PermissionError

# StopIteration — iterator exhausted
next(iter([]))       # StopIteration

# == NEVER catch these (except for very specific reasons): ==
# BaseException      — too broad (includes SystemExit, KeyboardInterrupt)
# KeyboardInterrupt  — user pressed Ctrl+C (let the program stop!)
# SystemExit         — sys.exit() was called
```

---

### 3.10 Practical Patterns

```python
# Pattern 1: EAFP (Easier to Ask Forgiveness than Permission)
# Python's preferred style!

# ❌ LBYL (Look Before You Leap) — Java style
if key in dictionary:
    value = dictionary[key]
else:
    value = default

# ✅ EAFP — Pythonic!
try:
    value = dictionary[key]
except KeyError:
    value = default

# Even better for dicts:
value = dictionary.get(key, default)

# Pattern 2: Retry with Backoff
import time

def retry(func, max_attempts=3, delay=1):
    """Retry a function up to max_attempts times."""
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except Exception as e:
            print(f"  Attempt {attempt}/{max_attempts} failed: {e}")
            if attempt == max_attempts:
                raise    # Re-raise on final attempt
            time.sleep(delay * attempt)   # Exponential backoff

# Pattern 3: Graceful Degradation
def get_client_data(client_id):
    """Try multiple sources — fall back gracefully."""
    try:
        return get_from_cache(client_id)
    except CacheMissError:
        pass
    
    try:
        data = get_from_database(client_id)
        update_cache(client_id, data)
        return data
    except DatabaseError:
        pass
    
    try:
        return get_from_backup(client_id)
    except FileNotFoundError:
        raise ClientError(f"Client {client_id} not found anywhere!")

# Pattern 4: Context Manager for Exception Handling
class ErrorCollector:
    """Collect errors without stopping execution."""
    
    def __init__(self):
        self.errors = []
        self.successes = 0
    
    def attempt(self, label=""):
        """Context manager that catches and stores errors."""
        return self._Attempt(self, label)
    
    class _Attempt:
        def __init__(self, collector, label):
            self.collector = collector
            self.label = label
        
        def __enter__(self):
            return self
        
        def __exit__(self, exc_type, exc_val, exc_tb):
            if exc_type:
                self.collector.errors.append({
                    "label": self.label,
                    "error": str(exc_val),
                    "type": exc_type.__name__,
                })
                return True    # Suppress the exception
            self.collector.successes += 1
            return False
```

---

### 3.11 Solving the Pipeline Problem — Resilient Billing System

```python
# ============================================
# Pearson Specter Litt — Resilient Billing Pipeline
# Exception handling for robust data processing
# ============================================

import json
from datetime import datetime

class BillingError(Exception):
    """Base billing exception."""
    pass

class InvalidDataError(BillingError):
    """Data validation failure."""
    def __init__(self, client, field, reason):
        self.client = client
        self.field = field
        super().__init__(f"{client}: invalid {field} — {reason}")

class ProcessingResult:
    """Track successes, failures, and skips."""
    
    def __init__(self):
        self.successes = []
        self.failures = []
        self.warnings = []
    
    def add_success(self, client, amount):
        self.successes.append({"client": client, "amount": amount})
    
    def add_failure(self, client, error):
        self.failures.append({"client": client, "error": str(error)})
    
    def add_warning(self, client, message):
        self.warnings.append({"client": client, "message": message})
    
    @property
    def total_billed(self):
        return sum(s["amount"] for s in self.successes)
    
    def report(self):
        lines = [
            f"\n{'='*55}",
            f"{'🏛️  BILLING PIPELINE RESULTS':^55}",
            f"{'='*55}",
            f"\n  ✅ Processed: {len(self.successes)}",
            f"  ❌ Failed: {len(self.failures)}",
            f"  ⚠️  Warnings: {len(self.warnings)}",
            f"  💰 Total Billed: ${self.total_billed:,.2f}",
        ]
        
        if self.successes:
            lines.append(f"\n  📊 SUCCESSFUL:")
            for s in sorted(self.successes, key=lambda x: x["amount"], reverse=True):
                lines.append(f"    ✅ {s['client']:<22} ${s['amount']:>12,.2f}")
        
        if self.failures:
            lines.append(f"\n  🚫 FAILURES:")
            for f in self.failures:
                lines.append(f"    ❌ {f['client']:<22} {f['error']}")
        
        if self.warnings:
            lines.append(f"\n  ⚠️  WARNINGS:")
            for w in self.warnings:
                lines.append(f"    ⚠️  {w['client']:<22} {w['message']}")
        
        lines.append(f"\n{'='*55}")
        return "\n".join(lines)


def validate_record(record):
    """Validate a billing record — raises on failure."""
    client = record.get("client", "UNKNOWN")
    
    if "client" not in record:
        raise InvalidDataError(client, "client", "missing field")
    
    if "hours" not in record:
        raise InvalidDataError(client, "hours", "missing field")
    
    hours = record["hours"]
    if not isinstance(hours, (int, float)):
        try:
            hours = float(hours)
        except (ValueError, TypeError):
            raise InvalidDataError(client, "hours", f"not numeric: {hours!r}")
    
    if hours < 0:
        raise InvalidDataError(client, "hours", f"negative: {hours}")
    
    if hours == 0:
        return None    # Skip, not an error
    
    rate = record.get("rate")
    if rate is None:
        raise InvalidDataError(client, "rate", "missing field")
    
    if not isinstance(rate, (int, float)):
        raise InvalidDataError(client, "rate", f"not numeric: {rate!r}")
    
    if rate <= 0:
        raise InvalidDataError(client, "rate", f"non-positive: {rate}")
    
    return {"client": client, "hours": float(hours), "rate": float(rate)}


def process_billing(records):
    """Process all records — never crashes, handles each independently."""
    result = ProcessingResult()
    
    for i, record in enumerate(records, 1):
        try:
            validated = validate_record(record)
            
            if validated is None:
                result.add_warning(
                    record.get("client", f"Record {i}"),
                    "Zero hours — skipped"
                )
                continue
            
            amount = validated["hours"] * validated["rate"]
            result.add_success(validated["client"], amount)
        
        except InvalidDataError as e:
            result.add_failure(e.client, e)
        
        except Exception as e:
            # Catch-all for unexpected errors
            result.add_failure(
                record.get("client", f"Record {i}"),
                f"Unexpected: {type(e).__name__}: {e}"
            )
    
    return result


# === Run the pipeline ===
records = [
    {"client": "Wayne Enterprises", "hours": 85, "rate": 450},
    {"client": "Stark Industries", "hours": "62", "rate": 400},
    {"client": "Oscorp", "hours": 55, "rate": 425},
    {"client": "Dalton Industries", "hours": 0, "rate": 450},       # Zero hours
    {"client": "Queen Industries", "hours": 40, "rate": -100},      # Negative rate
    {"hours": 30, "rate": 350},                                      # Missing client
    {"client": "Palmer Tech", "hours": "N/A", "rate": 375},         # Non-numeric
    {"client": "Luthor Corp", "hours": 25},                          # Missing rate
    {"client": "Rand Corp", "hours": 45, "rate": 400},
]

result = process_billing(records)
print(result.report())

# Key insight: ALL 9 records processed — none crashed the pipeline!
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Bare `except:` — The Silent Bug Factory 🔥

```python
# ❌ Catches EVERYTHING — including Ctrl+C and sys.exit()!
try:
    result = process(data)
except:             # Bare except — catches BaseException!
    pass            # Silently swallows ALL errors!

# ❌ Also too broad
try:
    result = process(data)
except Exception:   # Catches all "regular" exceptions
    pass            # Still swallows errors silently!

# ✅ Catch specific exceptions
try:
    result = process(data)
except (ValueError, KeyError) as e:
    print(f"Data error: {e}")
    result = default_value
```

---

### Pitfall 2: Catching Too Early — Hiding Bugs

```python
# ❌ Wrapping too much code in try
try:
    name = get_name()
    age = get_age()
    email = get_email()
    result = process(name, age, email)
    save(result)
except ValueError:
    print("Something went wrong")    # Which line? We have no idea!

# ✅ Wrap only the risky part
name = get_name()
try:
    age = int(age_string)
except ValueError:
    print(f"Invalid age: {age_string}")
    age = 0
```

---

### Pitfall 3: `except Exception as e: pass` — The Worst Pattern

```python
# ❌ NEVER DO THIS — errors become invisible!
def process(data):
    try:
        return complex_operation(data)
    except Exception:
        pass    # Bug? What bug? I don't see any bug!

# ✅ At minimum: log the error
def process(data):
    try:
        return complex_operation(data)
    except Exception as e:
        logging.error(f"process() failed: {e}", exc_info=True)
        return None    # Explicit fallback
```

---

### Pitfall 4: Exception in `finally` Replaces Original

```python
# ❌ Exception in finally replaces the original!
try:
    result = 1 / 0          # ZeroDivisionError
finally:
    cleanup()                # If THIS raises, ZeroDivisionError is LOST!

# ✅ Protect finally from exceptions too
try:
    result = 1 / 0
finally:
    try:
        cleanup()
    except Exception:
        logging.error("Cleanup failed", exc_info=True)
```

---

### Pitfall 5: Using Exceptions for Flow Control

```python
# ❌ Don't use exceptions as if/else replacements
def find_item(lst, target):
    try:
        return lst.index(target)
    except ValueError:
        return -1

# ✅ Use normal control flow when possible
def find_item(lst, target):
    if target in lst:
        return lst.index(target)
    return -1

# However, EAFP IS Pythonic for dict lookups, type conversions, etc.
# Use your judgment — exceptions are fine for truly exceptional cases.
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🛡️ Analogy: Exception Handling = Legal Contingency Planning

```
TRIAL PREPARATION:
  try:
      Present the case (main logic)
  
  except EvidenceRejected:
      Switch to alternative evidence (specific handler)
  
  except WitnessHostile:
      Redirect questioning (different strategy)
  
  except Exception:
      Request recess (catch-all fallback)
  
  else:
      Jury deliberates (only if nothing went wrong)
  
  finally:
      File court transcripts (ALWAYS done — cleanup)

DONNA'S RULES:
  1. "Plan for specific disasters, not just 'something bad'"
  2. "Never hide evidence (don't suppress exceptions silently)"
  3. "Always file the transcripts (always cleanup in finally)"
  4. "A good contingency plan doesn't mean expecting failure —
      it means being prepared for it"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Safe Calculator**
```
Build a calculator that never crashes:
  - Division by zero → "Cannot divide by zero"
  - Invalid numbers → "Invalid input"
  - Unsupported operation → "Unknown operation"

safe_calc("10", "+", "5")   → 15.0
safe_calc("10", "/", "0")   → "Cannot divide by zero"
safe_calc("abc", "+", "5")  → "Invalid input: abc"
```

**Exercise 2: File Reader with Fallback**
```
Write safe_read(path, default=None):
  - Returns file contents if file exists
  - Returns default if file not found
  - Prints warning for permission errors
  - Always logs which file was attempted
```

**Exercise 3: List Accessor**
```
Write safe_get(lst, index, default=None):
  - Returns lst[index] if valid
  - Returns default for IndexError
  - Returns default for TypeError (invalid index type)
  
Test with: lists, negative indices, slices, string indices
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Custom Exception Hierarchy**
```
Build a web app exception hierarchy:
  AppError
  ├── AuthError
  │    ├── InvalidCredentialsError
  │    └── ExpiredTokenError
  ├── DatabaseError  
  │    ├── ConnectionError
  │    └── QueryError
  └── ValidationError
       ├── MissingFieldError
       └── InvalidFormatError

Each with meaningful attributes and messages.
```

**Exercise 5: Retry Decorator**
```
Build @retry(max_attempts=3, backoff=1.0, exceptions=(Exception,)):

@retry(max_attempts=3, backoff=2.0, exceptions=(ConnectionError,))
def fetch_data(url):
    return requests.get(url)

Should: retry on failure, exponential backoff, log attempts.
```

**Exercise 6: Batch Processor**
```
Build a batch_process(items, func) that:
  - Calls func on each item
  - Collects successes and failures separately
  - Never stops on individual failures
  - Returns BatchResult with .successes, .failures, .report()
```

---

### 🔴 Advanced Exercises

**Exercise 7: Circuit Breaker**
```
Build a CircuitBreaker pattern:
  - Closed (normal) → failures increment counter
  - Open (tripped) → immediately raises without calling function
  - Half-Open (testing) → allows one call to test recovery
  - Auto-reset after cooldown period

@circuit_breaker(threshold=5, cooldown=30)
def flaky_service():
    ...
```

**Exercise 8: Exception Middleware**
```
Build an exception handling middleware:

pipeline = Pipeline()
pipeline.add_handler(ValueError, lambda e: default_value)
pipeline.add_handler(ConnectionError, lambda e: retry())
pipeline.add_handler(Exception, lambda e: log_and_alert(e))

result = pipeline.execute(risky_function, args)
```

**Exercise 9: Structured Error Reporting**
```
Build an error reporter that:
  - Collects all exceptions during execution
  - Includes traceback, timestamp, context
  - Groups by exception type
  - Generates HTML/JSON report
  - Sends email alert for critical errors
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Bare except
try:
    data = load_data()
except:                     # Catches Ctrl+C too!
    data = {}

# Bug 2: Wrong exception order
try:
    open("file.txt")
except OSError:             # Catches FileNotFoundError first!
    print("OS error")
except FileNotFoundError:   # Never reached!
    print("File not found")

# Bug 3: Exception variable scope
try:
    x = 1 / 0
except ZeroDivisionError as e:
    error = e
# print(e)                  # NameError: 'e' is not defined!
#                            # (e is deleted after except block in Python 3)

# Bug 4: finally changes return value
def get_value():
    try:
        return 42
    finally:
        return 0    # ⚠️ Overrides the try's return!

print(get_value())           # 0, not 42!

# Bug 5: Modifying data before potential exception
items = [1, 2, 3]
try:
    items.append(4)          # Modifies before possible error
    result = items[10]       # IndexError
except IndexError:
    print(items)             # [1, 2, 3, 4] — already modified!
```

### Fixed Code ✅

```python
# Fix 1: Catch specific exceptions
try:
    data = load_data()
except (FileNotFoundError, json.JSONDecodeError) as e:
    print(f"Using default: {e}")
    data = {}

# Fix 2: Most specific first
try:
    open("file.txt")
except FileNotFoundError:   # ✅ Specific first
    print("File not found")
except OSError:              # ✅ Broader second
    print("OS error")

# Fix 3: Save to a different variable
try:
    x = 1 / 0
except ZeroDivisionError as e:
    saved_error = e            # ✅ Save before it's deleted
print(saved_error)

# Fix 4: Don't return in finally
def get_value():
    try:
        return 42
    finally:
        print("Cleanup")      # ✅ Cleanup, but don't return

# Fix 5: Validate before modifying
items = [1, 2, 3]
try:
    if 10 < len(items):       # ✅ Check first
        result = items[10]
    items.append(4)            # ✅ Modify after validation
except IndexError:
    pass
```

---

## 8. 🌍 Real-World Application

### Exception Handling in Production

| Application | Pattern | What It Does |
|-------------|---------|-------------|
| **Web servers** | Global handler | Catch-all returns 500 page instead of crash |
| **APIs** | HTTP error codes | Exceptions map to 400/404/500 responses |
| **ETL pipelines** | Batch processing | Record failures without stopping |
| **CLI tools** | User-friendly errors | Show message, not traceback |
| **Libraries** | Custom exceptions | Clear error types for consumers |
| **Databases** | Transaction rollback | Undo partial changes on error |

---

## 9. ✅ Knowledge Check

---

**Q1.** What do `try`, `except`, `else`, and `finally` do?

**Q2.** Why should you catch specific exceptions instead of `except Exception`?

**Q3.** What is the difference between `raise` and `raise ... from ...`?

**Q4.** When does the `else` block run?

**Q5.** When does the `finally` block run?

**Q6.** How do you define a custom exception?

**Q7.** What is EAFP and why is it preferred in Python?

**Q8.** Why is `except: pass` the worst pattern?

**Q9.** What happens to the exception variable `e` after the except block?

**Q10.** In what order should you list except clauses?

---

### 📋 Answer Key

> **Q1.** `try`: attempt risky code. `except`: handle specific failures. `else`: runs only on success. `finally`: always runs (cleanup).

> **Q2.** Broad catches hide bugs. Specific catches let you handle each error appropriately and let unexpected errors propagate.

> **Q3.** `raise` creates/re-raises an exception. `raise X from Y` chains exceptions, showing the original cause in the traceback.

> **Q4.** Only when the `try` block completes without any exception.

> **Q5.** Always — after success, after handled exception, even after unhandled exception or `return`.

> **Q6.** `class MyError(Exception): pass` — inherit from `Exception` or a custom base exception.

> **Q7.** "Easier to Ask Forgiveness than Permission" — try the operation and handle failure, rather than checking preconditions. Preferred because it's faster and cleaner for common patterns.

> **Q8.** Bare `except` catches everything (including `SystemExit`, `KeyboardInterrupt`). `pass` silently swallows all errors, making bugs invisible.

> **Q9.** Python deletes `e` after the except block exits (Python 3) to avoid reference cycles with traceback objects.

> **Q10.** Most specific first, most general last. `FileNotFoundError` before `OSError` before `Exception`.

---

## 🎬 End of Lesson Scene

**Jessica:** "How many clients processed?"

**Mike:** "All 9. Three had errors — missing fields, non-numeric hours, negative rate. Each error was caught, logged, and reported. The other 6 billed successfully. Total: $138,000."

**Harvey:** "And the pipeline didn't crash?"

**Mike:** "Not once. Each record processed independently. Failures are collected, not fatal. That's the contingency plan."

**Harvey:** "Next: lambda functions. Anonymous functions for quick, inline operations."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `try`/`except` basics | ✅ |
| 2 | Catching specific exception types | ✅ |
| 3 | Multiple `except` clauses (order matters!) | ✅ |
| 4 | `else` clause (success handler) | ✅ |
| 5 | `finally` clause (always runs) | ✅ |
| 6 | `raise` and re-raising exceptions | ✅ |
| 7 | Custom exceptions with hierarchy | ✅ |
| 8 | EAFP pattern | ✅ |
| 9 | Exception chaining (`from`) | ✅ |
| 10 | Resilient pipeline pattern | ✅ |

---

> **Next Lesson →** Level 3, Lesson 6: *Use Lambda Functions in Python*
