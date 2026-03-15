# 🏛️ Level 2 – Applying | Lesson 13: Debug Python Code with Print Statements and Tools

## *"The Closing Argument"*

> **O'Reilly Skill Objective:** *Debug Python code using print statements, logging, and the built-in debugger (pdb).*

> **Previously Learned:** All Level 1 concepts + functions, classes, comprehensions, generators, data structures, attributes, class/static methods, mutability, serialization, standard library, virtual environments

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

*Late Friday evening. The annual billing report is due in 2 hours. The script produces wrong numbers.*

**Harvey:** "The report says Wayne Enterprises was billed $0. They're our biggest client."

**Mike:** "The code runs without errors. No exceptions, no crashes. It just… gives the wrong answer."

**Harvey:** "Then find the bug. You have 2 hours."

**Mike:** "These are the hardest bugs — silent failures. The code WORKS, it just doesn't work CORRECTLY. You can't fix what you can't see. That's what debugging is: making invisible logic visible."

**Harvey:** "So make it visible."

**Mike:** "Three tools. Print statements to see values. Logging for persistent trails. The debugger (pdb) to freeze time and inspect everything. Let me show you."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Debugging is detective work."

| Stage | Detective Analogy | Python Tool |
|-------|------------------|-------------|
| **Observe** | "What happened at the crime scene?" | Read the error message / traceback |
| **Hypothesize** | "I think the suspect went left" | Form a theory about the bug |
| **Investigate** | "Check the security cameras" | Print statements, logging, debugger |
| **Verify** | "Does the evidence match?" | Run and confirm the fix |

**The debugging hierarchy:**

| Level | Tool | When to Use |
|-------|------|-------------|
| 1️⃣ | **Read the error** | Always start here |
| 2️⃣ | **Print statements** | Quick inspection of values |
| 3️⃣ | **f-string debugging** | Formatted variable inspection |
| 4️⃣ | **Logging** | Persistent, configurable output |
| 5️⃣ | **`pdb` debugger** | Step-by-step execution, breakpoints |
| 6️⃣ | **IDE debugger** | Visual breakpoints, watch variables |
| 7️⃣ | **`assert` statements** | Catch assumptions early |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Reading Error Messages ⭐⭐⭐

**Mike:** "90% of bugs can be found just by READING the error message carefully."

```python
# The error:
# Traceback (most recent call last):
#   File "billing.py", line 42, in calculate_total
#     total = hours * rate
#             ~~~~~ ^ ~~~~
# TypeError: can't multiply sequence by non-int of type 'str'

# Translation:
# File: billing.py
# Line: 42
# Function: calculate_total
# Problem: hours or rate is a STRING, not a number
# Fix: Convert to number: total = float(hours) * float(rate)
```

**Common error types and what they mean:**

| Error | Meaning | Common Fix |
|-------|---------|------------|
| `SyntaxError` | Python can't parse your code | Fix typo, missing colon/bracket |
| `NameError` | Variable doesn't exist | Check spelling, scope, imports |
| `TypeError` | Wrong type for an operation | Convert types, check function args |
| `ValueError` | Right type, wrong value | Validate input before using |
| `IndexError` | List index out of range | Check list length, off-by-one |
| `KeyError` | Dict key doesn't exist | Use `.get()` or check `in` |
| `AttributeError` | Object doesn't have that attribute | Check type, spelling, imports |
| `ImportError` | Module not found | Install package, check name |
| `FileNotFoundError` | File path wrong | Check path, use `os.path.exists()` |
| `IndentationError` | Mixed tabs/spaces | Use consistent indentation |

---

### 3.2 Print Debugging — Level 1 ⭐

```python
def calculate_billing(cases):
    """Bug: returns wrong total for some clients."""
    total = 0
    for case in cases:
        # STEP 1: See what we're working with
        print(f"Processing: {case}")
        
        hours = case.get("hours", 0)
        rate = case.get("rate", 0)
        
        # STEP 2: Check intermediate values
        print(f"  hours={hours} (type: {type(hours).__name__})")
        print(f"  rate={rate} (type: {type(rate).__name__})")
        
        amount = hours * rate
        
        # STEP 3: Check calculation
        print(f"  amount={amount}")
        
        total += amount
    
    print(f"TOTAL: {total}")
    return total

# Test
cases = [
    {"client": "Wayne", "hours": 85, "rate": 450},
    {"client": "Stark", "hours": "62", "rate": 400},   # 🐛 hours is a STRING!
]
calculate_billing(cases)
# Processing: {'client': 'Wayne', 'hours': 85, 'rate': 450}
#   hours=85 (type: int)
#   rate=450 (type: int)
#   amount=38250
# Processing: {'client': 'Stark', 'hours': '62', 'rate': 400}
#   hours=62 (type: str)        ← FOUND IT! String, not int
#   rate=400 (type: int)
#   amount=626262...62          ← String repeated 400 times!
```

---

### 3.3 Strategic Print Patterns ⭐

```python
# Pattern 1: Marker prints — "Did execution reach here?"
def process_case(case):
    print(">>> ENTERING process_case")
    
    if case["type"] == "Merger":
        print(">>> Taking MERGER path")
        result = handle_merger(case)
    else:
        print(">>> Taking DEFAULT path")
        result = handle_default(case)
    
    print(f">>> EXITING process_case, result={result}")
    return result

# Pattern 2: Variable dump — "What are the values?"
def debug_dump(**kwargs):
    """Print all variables with their names and types."""
    print("=" * 40)
    for name, value in kwargs.items():
        print(f"  {name} = {value!r} ({type(value).__name__})")
    print("=" * 40)

# Usage:
debug_dump(hours=hours, rate=rate, client=client_name, total=total)

# Pattern 3: Loop iteration tracking
for i, item in enumerate(items):
    if i < 3 or i == len(items) - 1:    # First 3 + last
        print(f"  [{i}/{len(items)}] {item}")

# Pattern 4: Conditional prints (toggle on/off)
DEBUG = True

def dprint(*args, **kwargs):
    if DEBUG:
        print("[DEBUG]", *args, **kwargs)

dprint(f"Value is {x}")   # Only prints when DEBUG = True
```

---

### 3.4 f-string Debugging (Python 3.8+) ⭐

```python
# The = sign in f-strings automatically shows "expression=value"
name = "Harvey"
rate = 450
cases = ["Wayne", "Stark"]

# Without =
print(f"name: {name}, rate: {rate}")    # name: Harvey, rate: 450

# With = (self-documenting!)
print(f"{name=}")                        # name='Harvey'
print(f"{rate=}")                        # rate=450
print(f"{len(cases)=}")                  # len(cases)=2
print(f"{cases[0]=}")                    # cases[0]='Wayne'
print(f"{rate * 85 =}")                  # rate * 85 =38250
print(f"{type(rate)=}")                  # type(rate)=<class 'int'>

# Combine with formatting
print(f"{rate=:,.2f}")                   # rate=450.00
print(f"{rate * 2000=:>15,.2f}")         # rate * 2000=    900,000.00
```

**Mike:** "`f'{var=}'` is the single most useful debugging trick in Python. It shows the variable name AND its value in one expression."

---

### 3.5 `logging` Module — Professional Debugging ⭐⭐

```python
import logging

# Basic setup
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logger = logging.getLogger("billing")

def calculate_billing(cases):
    logger.info(f"Processing {len(cases)} cases")
    total = 0
    
    for case in cases:
        logger.debug(f"Case: {case['client']}, hours={case['hours']}, rate={case['rate']}")
        
        try:
            hours = float(case["hours"])
            rate = float(case["rate"])
        except (ValueError, TypeError) as e:
            logger.error(f"Invalid data for {case['client']}: {e}")
            continue
        
        amount = hours * rate
        total += amount
        logger.debug(f"  Amount: ${amount:,.2f} | Running total: ${total:,.2f}")
    
    logger.info(f"Total billing: ${total:,.2f}")
    return total

# Output:
# 2026-03-15 07:35:00 [INFO] billing: Processing 3 cases
# 2026-03-15 07:35:00 [DEBUG] billing: Case: Wayne, hours=85, rate=450
# 2026-03-15 07:35:00 [DEBUG] billing:   Amount: $38,250.00 | Running total: $38,250.00
# 2026-03-15 07:35:00 [ERROR] billing: Invalid data for Stark: could not convert...
```

**Logging levels:**

| Level | When to Use | Example |
|-------|-------------|---------|
| `DEBUG` | Detailed diagnostic info | `logger.debug(f"x={x}")` |
| `INFO` | General operational events | `logger.info("Server started")` |
| `WARNING` | Something unexpected but not broken | `logger.warning("Disk 90% full")` |
| `ERROR` | Something failed | `logger.error("Connection lost")` |
| `CRITICAL` | System is about to crash | `logger.critical("Data corrupted!")` |

```python
# Logging to a file (persists after program ends)
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler("billing.log"),      # Write to file
        logging.StreamHandler()                    # Also print to console
    ]
)
```

**Print vs Logging:**

| Feature | `print()` | `logging` |
|---------|-----------|-----------|
| Turn off easily | ❌ Must delete/comment | ✅ Change level |
| Severity levels | ❌ | ✅ DEBUG → CRITICAL |
| Write to file | ❌ | ✅ FileHandler |
| Timestamps | ❌ | ✅ Automatic |
| Production safe | ❌ | ✅ |
| Format control | ❌ | ✅ Formatters |
| Multiple outputs | ❌ | ✅ Handlers |

---

### 3.6 `assert` Statements — Catch Assumptions ⭐

```python
def calculate_billing(hours, rate, client_name):
    """Calculate billing with built-in sanity checks."""
    
    # Assert your assumptions
    assert isinstance(hours, (int, float)), f"hours must be numeric, got {type(hours)}"
    assert isinstance(rate, (int, float)), f"rate must be numeric, got {type(rate)}"
    assert hours >= 0, f"hours cannot be negative: {hours}"
    assert rate > 0, f"rate must be positive: {rate}"
    assert client_name, "client_name cannot be empty"
    
    total = hours * rate
    
    # Assert postconditions
    assert total >= 0, f"total should never be negative: {total}"
    
    return total

# Good call
calculate_billing(85, 450, "Wayne")   # ✅ Returns 38250

# Bad call
calculate_billing("85", 450, "Wayne")
# AssertionError: hours must be numeric, got <class 'str'>

# ⚠️ Asserts can be disabled! (python -O flag)
# Never use assert for input validation in production!
# Use assert for development/testing checks only
```

---

### 3.7 `pdb` — The Python Debugger ⭐⭐

```python
# pdb lets you PAUSE execution and inspect everything interactively

def calculate_total(cases):
    total = 0
    for i, case in enumerate(cases):
        hours = case.get("hours", 0)
        rate = case.get("rate", 0)
        
        # Insert a breakpoint — execution PAUSES here
        breakpoint()   # Python 3.7+ (same as: import pdb; pdb.set_trace())
        
        amount = hours * rate
        total += amount
    return total

# When execution hits breakpoint(), you get an interactive prompt:
# > billing.py(10)calculate_total()
# -> amount = hours * rate
# (Pdb)
```

**Essential pdb commands:**

| Command | Short | What It Does |
|---------|:-----:|-------------|
| `help` | `h` | Show all commands |
| `next` | `n` | Execute next line (step OVER functions) |
| `step` | `s` | Execute next line (step INTO functions) |
| `continue` | `c` | Run until next breakpoint |
| `print expr` | `p expr` | Print an expression |
| `pp expr` | | Pretty-print an expression |
| `list` | `l` | Show code around current line |
| `where` | `w` | Show call stack (how you got here) |
| `up` | `u` | Go up one frame in call stack |
| `down` | `d` | Go down one frame in call stack |
| `break N` | `b N` | Set breakpoint at line N |
| `clear N` | `cl N` | Clear breakpoint |
| `quit` | `q` | Exit debugger |
| `locals()` | | Show all local variables |
| `!statement` | | Execute Python statement |

```python
# Example pdb session:
# (Pdb) p hours
# '62'
# (Pdb) p type(hours)
# <class 'str'>           ← FOUND IT! Should be int!
# (Pdb) p rate
# 400
# (Pdb) p hours * rate
# '62626262...62'          ← String repetition, not multiplication!
# (Pdb) p float(hours) * rate
# 24800.0                  ← CORRECT answer
# (Pdb) c                  ← Continue execution
```

---

### 3.8 Conditional Breakpoints

```python
# Only break when a condition is true
def process_cases(cases):
    for case in cases:
        hours = case.get("hours", 0)
        rate = case.get("rate", 0)
        
        # Only pause for Wayne's case
        if case["client"] == "Wayne":
            breakpoint()
        
        # Or: only pause when something looks wrong
        if not isinstance(hours, (int, float)):
            breakpoint()   # Pause only on suspicious data
        
        amount = hours * rate

# In pdb itself:
# (Pdb) b 10, case["client"] == "Wayne"    # Conditional breakpoint
```

---

### 3.9 `traceback` Module — Better Error Reports

```python
import traceback

def risky_operation():
    try:
        result = calculate_billing(cases)
    except Exception as e:
        # Print the full traceback without crashing
        print("An error occurred:")
        traceback.print_exc()
        
        # Or capture it as a string
        error_details = traceback.format_exc()
        log_error(error_details)
        
        # Or get structured info
        tb = traceback.extract_tb(e.__traceback__)
        for frame in tb:
            print(f"  File: {frame.filename}, Line: {frame.lineno}")
            print(f"  Function: {frame.name}")
            print(f"  Code: {frame.line}")
```

---

### 3.10 Debugging Strategies — The Systematic Approach

```python
"""
THE 6-STEP DEBUGGING METHOD (Mike's approach):

1. REPRODUCE — Can you make the bug happen consistently?
   → Write a minimal test case that triggers the bug

2. ISOLATE — Where EXACTLY does it go wrong?
   → Binary search: add prints at start/middle/end
   → Narrow down which function, which line

3. IDENTIFY — What is the wrong value? What should it be?
   → Print the actual vs expected values
   → Check types, not just values

4. HYPOTHESIZE — Why is it wrong?
   → Type mismatch? Off-by-one? Wrong variable?
   → Reference sharing? Mutation? Scope issue?

5. FIX — Change the minimal amount of code
   → Fix the root cause, not the symptom

6. VERIFY — Did the fix work? Did it break anything else?
   → Run the original failing case
   → Run all existing tests
"""

# Example: Binary search debugging
def find_bug(data):
    # Step 1: Is the input correct?
    print(f"INPUT: {data[:3]}... ({len(data)} items)")   # ← Check here
    
    processed = transform(data)
    # Step 2: Is the transform correct?
    print(f"AFTER TRANSFORM: {processed[:3]}...")         # ← Check here
    
    filtered = filter_items(processed)
    # Step 3: Is the filter correct?
    print(f"AFTER FILTER: {filtered[:3]}...")             # ← Check here
    
    result = aggregate(filtered)
    # Step 4: Is the aggregation correct?
    print(f"RESULT: {result}")                            # ← Check here
    
    return result

# If the bug appears between Step 2 and Step 3,
# the problem is in filter_items()!
```

---

### 3.11 Solving Harvey's Problem — The Complete Debugging Session

```python
# ============================================
# THE BUG: Wayne Enterprises billed $0
# ============================================

import logging

# Set up logging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)8s] %(message)s",
    datefmt="%H:%M:%S"
)
logger = logging.getLogger("billing_debug")

class BillingDebugger:
    """Systematic debugging of the billing system."""
    
    def __init__(self):
        self.issues_found = []
    
    def investigate(self, cases):
        """Run the billing calculation with full debugging."""
        logger.info(f"{'='*50}")
        logger.info(f"BILLING INVESTIGATION — {len(cases)} cases")
        logger.info(f"{'='*50}")
        
        total = 0
        client_totals = {}
        
        for i, case in enumerate(cases):
            logger.debug(f"\n--- Case {i+1}/{len(cases)} ---")
            logger.debug(f"Raw data: {case}")
            
            # Check 1: Required fields present?
            required = ["client", "hours", "rate"]
            missing = [f for f in required if f not in case]
            if missing:
                issue = f"Missing fields {missing} in case {i+1}"
                logger.error(issue)
                self.issues_found.append(issue)
                continue
            
            client = case["client"]
            hours = case["hours"]
            rate = case["rate"]
            
            # Check 2: Correct types?
            logger.debug(f"  client={client!r} (type: {type(client).__name__})")
            logger.debug(f"  hours={hours!r} (type: {type(hours).__name__})")
            logger.debug(f"  rate={rate!r} (type: {type(rate).__name__})")
            
            if not isinstance(hours, (int, float)):
                issue = f"Case {i+1} ({client}): hours is {type(hours).__name__}, not numeric"
                logger.warning(issue)
                self.issues_found.append(issue)
                try:
                    hours = float(hours)
                    logger.info(f"  Auto-fixed: hours converted to {hours}")
                except (ValueError, TypeError):
                    logger.error(f"  Cannot convert hours={hours!r}")
                    continue
            
            if not isinstance(rate, (int, float)):
                issue = f"Case {i+1} ({client}): rate is {type(rate).__name__}, not numeric"
                logger.warning(issue)
                self.issues_found.append(issue)
                try:
                    rate = float(rate)
                    logger.info(f"  Auto-fixed: rate converted to {rate}")
                except (ValueError, TypeError):
                    logger.error(f"  Cannot convert rate={rate!r}")
                    continue
            
            # Check 3: Reasonable values?
            if hours < 0:
                issue = f"Case {i+1} ({client}): Negative hours ({hours})"
                logger.error(issue)
                self.issues_found.append(issue)
                continue
            
            if hours == 0:
                issue = f"Case {i+1} ({client}): Zero hours — intentional?"
                logger.warning(issue)
                self.issues_found.append(issue)
            
            if rate <= 0:
                issue = f"Case {i+1} ({client}): Invalid rate ({rate})"
                logger.error(issue)
                self.issues_found.append(issue)
                continue
            
            # Calculate
            amount = hours * rate
            total += amount
            client_totals[client] = client_totals.get(client, 0) + amount
            
            logger.debug(f"  ✅ {client}: {hours}h × ${rate}/hr = ${amount:,.2f}")
        
        # Final report
        logger.info(f"\n{'='*50}")
        logger.info(f"INVESTIGATION REPORT")
        logger.info(f"{'='*50}")
        logger.info(f"Total billing: ${total:,.2f}")
        
        logger.info(f"\nClient breakdown:")
        for client, amount in sorted(client_totals.items(),
                                      key=lambda x: x[1], reverse=True):
            bar = "█" * int(amount / max(client_totals.values()) * 20)
            logger.info(f"  {client:<22} ${amount:>12,.2f} {bar}")
        
        if self.issues_found:
            logger.warning(f"\n⚠️  {len(self.issues_found)} ISSUES FOUND:")
            for issue in self.issues_found:
                logger.warning(f"  • {issue}")
        else:
            logger.info(f"\n✅ No issues found!")
        
        return total, client_totals

# === THE BUGGY DATA ===
cases = [
    {"client": "Wayne Enterprises", "hours": 85, "rate": 450},
    {"client": "Stark Industries", "hours": "62", "rate": 400},      # 🐛 string hours
    {"client": "Oscorp", "hours": 55, "rate": 425},
    {"client": "Dalton Industries", "hours": 0, "rate": 450},         # 🐛 zero hours
    {"client": "Queen Industries", "hours": 40, "rate": -100},        # 🐛 negative rate
    {"hours": 30, "rate": 350},                                        # 🐛 missing client
    {"client": "Palmer Tech", "hours": "N/A", "rate": 375},           # 🐛 non-numeric hours
]

debugger = BillingDebugger()
total, breakdown = debugger.investigate(cases)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Leaving Debug Prints in Production Code 🔥

```python
# ❌ Scattered print statements everywhere
def calculate(hours, rate):
    print(f"DEBUG: hours={hours}, rate={rate}")    # Forgot to remove!
    total = hours * rate
    print(f"DEBUG: total={total}")                  # Still there!
    return total

# ✅ Use logging — can be turned off without removing code
import logging
logger = logging.getLogger(__name__)

def calculate(hours, rate):
    logger.debug(f"hours={hours}, rate={rate}")    # Only shows at DEBUG level
    total = hours * rate
    logger.debug(f"total={total}")
    return total

# In production: set level to WARNING — debug messages disappear
logging.basicConfig(level=logging.WARNING)
```

---

### Pitfall 2: Print Doesn't Show Type Issues

```python
# ❌ print() can be misleading
x = "42"
print(x)         # 42 — looks like an integer!
print(42)        # 42 — looks identical!

# ✅ Use repr() or f-string = to see the truth
print(repr(x))    # '42' — clearly a string!
print(f"{x=}")     # x='42' — shows quotes
print(f"{type(x)=}")  # type(x)=<class 'str'>
```

---

### Pitfall 3: Debugging Mutated Data

```python
# ❌ The bug happens BEFORE the print
data = [3, 1, 2]
sorted_data = data.sort()      # sort() returns None and mutates data!
print(f"sorted: {sorted_data}")  # sorted: None — but data IS sorted

# ✅ Print BEFORE and AFTER mutations
print(f"Before: {data}")
data.sort()
print(f"After: {data}")

# Even better: use sorted() which doesn't mutate
sorted_data = sorted(data)
```

---

### Pitfall 4: Breakpoint Left in Code

```python
# ❌ Committed to git with breakpoint still in code
def process(data):
    breakpoint()          # 💀 Production users hit this!
    return transform(data)

# ✅ Use environment variable to disable
# PYTHONBREAKPOINT=0 python main.py    ← Skips all breakpoints

# ✅ Or use conditional breakpoints
import os
if os.environ.get("DEBUG"):
    breakpoint()
```

---

### Pitfall 5: Silent Failures in Try/Except

```python
# ❌ Hiding errors — the WORST debugging pitfall
try:
    result = process(data)
except Exception:
    pass    # Silently swallows ALL errors!

# ✅ At minimum, log the error
try:
    result = process(data)
except Exception as e:
    logger.error(f"process() failed: {e}", exc_info=True)
    result = default_value
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🔍 Analogy: Debugging = Crime Scene Investigation

```
CRIME (BUG):
  "Wayne's bill is $0"
  
EVIDENCE GATHERING:
  📷 Print statements  = Security camera footage (snapshots)
  📋 Logging           = Witness statements (permanent record)  
  🔬 pdb debugger      = Forensic analysis (freeze & examine)
  🚨 assert            = Alarm system (catch impossible states)
  
INVESTIGATION:
  1. REPRODUCE  = "Can we recreate the crime?"
  2. ISOLATE    = "Which room did it happen in?"
  3. IDENTIFY   = "What was taken vs what should be there?"
  4. HYPOTHESIZE = "Who had motive and opportunity?"
  5. FIX        = "Arrest the suspect"
  6. VERIFY     = "Did this solve the case? Any accomplices?"
  
DONNA'S RULE:
  "Never hide evidence. A bare 'except: pass' is like
   shredding the security tapes."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Print Debugging**
```
This function has a bug. Use print statements to find it:

def average(numbers):
    total = 0
    for num in numbers:
        total += num
    return total / len(numbers) - 1    # Returns wrong answer

# Expected: average([10, 20, 30]) → 20.0
# Actual:   average([10, 20, 30]) → 19.0
```

**Exercise 2: f-string = Debugging**
```
Use f"{var=}" to debug this function:

def find_discount(price, member_years):
    if member_years > 10:
        rate = 0.20
    elif member_years > 5:
        rate = 0.10
    else:
        rate = 0.5    # Bug! Should be 0.05
    return price * (1 - rate)

# Test with: find_discount(100, 2) → should be 95, gets 50
```

**Exercise 3: Reading Tracebacks**
```
For each error, identify: the file, line, function, and fix:

a) NameError: name 'reuslt' is not defined
b) TypeError: unsupported operand type(s) for +: 'int' and 'str'
c) IndexError: list index out of range
d) KeyError: 'naem'
e) AttributeError: 'list' object has no attribute 'push'
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Logging Setup**
```
Create a logging configuration that:
a) Writes DEBUG+ to a file
b) Writes WARNING+ to console
c) Includes timestamps, levels, and module names
d) Rotates log files at 1MB
e) Keeps the last 5 log files
```

**Exercise 5: Debug Detective**
```
This function computes grade averages but returns wrong results.
Use ONLY pdb (no print!) to find ALL 3 bugs:

def calculate_grades(students):
    results = {}
    for student in students:
        total = 0
        for score in student["scores"]:
            total =+ score           # Bug 1
        avg = total / len(students)   # Bug 2
        if avg >= 90:
            grade = 'A'
        elif avg >= 80:
            grade = 'B'
        elif avg >= 70:
            grade = 'B'              # Bug 3
        else:
            grade = 'F'
        results[student["name"]] = {"avg": avg, "grade": grade}
    return results
```

**Exercise 6: Assert-Driven Development**
```
Write a function with comprehensive asserts:

def transfer(from_account, to_account, amount):
    # Add asserts for:
    # - accounts must exist
    # - amount must be positive
    # - from_account must have sufficient balance
    # - total money in system must be conserved
    # - no account can go negative
    pass
```

---

### 🔴 Advanced Exercises

**Exercise 7: Custom Debugger**
```
Build a @debug decorator that:
  - Logs function entry with all arguments
  - Logs return value
  - Logs exceptions with traceback
  - Measures execution time
  - Can be toggled on/off globally

@debug
def calculate(a, b):
    return a / b
```

**Exercise 8: Bug Hunt Challenge**
```
This 50-line program has 10 bugs (some obvious, some subtle).
Find ALL 10 using any debugging technique.
Include your debugging process notes.
(Create the 50-line buggy program yourself!)
```

**Exercise 9: Debugging Dashboard**
```
Build a DebugDashboard class that:
  - Tracks all debug output (prints, logs, asserts)
  - Shows a timeline of events
  - Records variable state changes
  - Generates a bug report
  - Exports to HTML for sharing

Usage:
  dash = DebugDashboard()
  with dash.track():
      result = buggy_function()
  dash.report()
```

---

## 7. 🐛 Debugging Scenario

### The Friday Night Bug Hunt 🔥

```python
# THE SCENARIO: Billing report shows $0 for Wayne Enterprises
# Here's the buggy code:

def generate_report(raw_data):
    """Generate the annual billing report."""
    clients = {}
    
    for record in raw_data:
        name = record["client"]
        hours = record["hours"]           # Bug 1: might be string
        rate = record.get("rate")         # Bug 2: might be None
        
        if name not in clients:
            clients[name] = []
        
        clients[name] = hours * rate      # Bug 3: = instead of .append()
    
    report = {}
    for name, total in clients.items():   # Bug 4: total is last amount, not sum
        discount = 0
        if total > 100000:
            discount = 0.1
        elif total > 50000:
            discount = .5                  # Bug 5: 50% instead of 5%!
        
        report[name] = total * (1 - discount)
    
    return report
```

### The Fix — Step by Step ✅

```python
import logging

logger = logging.getLogger("billing")

def generate_report(raw_data):
    """Generate the annual billing report — FIXED."""
    clients = {}
    
    for record in raw_data:
        name = record["client"]
        
        # Fix 1: Convert and validate types
        try:
            hours = float(record["hours"])
        except (ValueError, TypeError):
            logger.warning(f"Invalid hours for {name}: {record['hours']!r}")
            continue
        
        # Fix 2: Handle missing rate
        rate = record.get("rate")
        if rate is None:
            logger.warning(f"Missing rate for {name}, using default")
            rate = 350    # Default rate
        rate = float(rate)
        
        # Fix 3: Append to list, don't overwrite!
        if name not in clients:
            clients[name] = []
        clients[name].append(hours * rate)
    
    report = {}
    for name, amounts in clients.items():
        # Fix 4: Sum the list of amounts
        total = sum(amounts)
        
        discount = 0
        if total > 100000:
            discount = 0.10
        elif total > 50000:
            discount = 0.05      # Fix 5: 5%, not 50%!
        
        final = total * (1 - discount)
        report[name] = {
            "subtotal": total,
            "discount": f"{discount:.0%}",
            "final": final
        }
        logger.info(f"{name}: ${total:,.2f} - {discount:.0%} = ${final:,.2f}")
    
    return report

# Test
raw = [
    {"client": "Wayne Enterprises", "hours": 85, "rate": 450},
    {"client": "Wayne Enterprises", "hours": 50, "rate": 450},
    {"client": "Stark Industries", "hours": "62", "rate": 400},
]

result = generate_report(raw)
print(f"\nWayne: ${result['Wayne Enterprises']['final']:,.2f}")
# Wayne: $54,675.00 ✅ (not $0!)
```

---

## 8. 🌍 Real-World Application

### Professional Debugging Practices

| Practice | How |
|----------|-----|
| **Rubber duck debugging** | Explain the code to a rubber duck (or colleague) |
| **Binary search debugging** | Print midpoint → narrow search by half |
| **Git bisect** | Find which commit introduced the bug |
| **Minimal reproduction** | Smallest code that triggers the bug |
| **Test-driven debugging** | Write a test that fails, then fix it |

### Debugging Cheat Sheet

```
QUICK FIX?
├── SyntaxError → Read the caret (^), fix the typo
├── NameError → Check spelling, imports, scope
├── TypeError → Check types with type() and repr()
├── KeyError → Use .get() with default
├── IndexError → Check len(), off-by-one
└── AttributeError → Check dir(obj), type(obj)

LOGIC BUG? (no error, wrong answer)
├── Add print(f"{var=}") at suspicious points
├── Check types (strings that look like numbers!)
├── Check mutation (is something modified in place?)
├── Check scope (is the right variable being used?)
├── Use breakpoint() to step through
└── Simplify: test with minimal data first
```

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the first thing you should do when you see a Python error?

**Q2.** What does `f"{variable=}"` do and why is it useful for debugging?

**Q3.** Name three advantages of `logging` over `print()`.

**Q4.** What are the five logging levels in order?

**Q5.** What does `breakpoint()` do?

**Q6.** In pdb, what is the difference between `n` (next) and `s` (step)?

**Q7.** What is the danger of `except Exception: pass`?

**Q8.** How do you tell the difference between `42` (int) and `"42"` (str) when printing?

**Q9.** What is the 6-step debugging method?

**Q10.** Why should you NEVER leave `breakpoint()` in production code?

---

### 📋 Answer Key

> **Q1.** Read the entire traceback carefully — it tells you the file, line, function, and error type.

> **Q2.** It prints `variable='value'` — shows both the variable name and its value, including type indicators like quotes.

> **Q3.** Configurable levels, file output, timestamps, can be disabled without removing code.

> **Q4.** `DEBUG` < `INFO` < `WARNING` < `ERROR` < `CRITICAL`.

> **Q5.** Pauses execution and opens an interactive pdb debugger at that point.

> **Q6.** `n` executes the next line (steps OVER function calls). `s` steps INTO function calls.

> **Q7.** Silently swallows all errors — bugs become invisible, making debugging impossible.

> **Q8.** Use `repr()`: `repr(42)` → `42`, `repr("42")` → `'42'` (with quotes). Or `f"{x=}"`.

> **Q9.** Reproduce → Isolate → Identify → Hypothesize → Fix → Verify.

> **Q10.** It pauses execution for ALL users, effectively freezing the application.

---

## 🎬 End of Lesson Scene — Level 2 Finale

**Harvey:** *(reviewing the fixed report)* "Wayne Enterprises: $54,675. All accounted for."

**Mike:** "Five bugs. String types, missing values, overwriting instead of appending, wrong sums, and a decimal error. All found in under 2 hours."

**Harvey:** "How?"

**Mike:** "Systematic debugging. Prints to see values. Logging for the trail. pdb to freeze and inspect. And most importantly — reading the error messages."

**Jessica:** *(walking in)* "The report is on the client's desk. And from what I hear, you've both mastered Level 2."

**Harvey:** "We've applied what we learned. Functions, classes, data structures, serialization, libraries, environments, and now debugging. We're ready to BUILD."

**Jessica:** "Level 3. Building real projects. That's where it gets interesting."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Reading tracebacks and error types | ✅ |
| 2 | Print debugging techniques | ✅ |
| 3 | f-string `=` debugging (Python 3.8+) | ✅ |
| 4 | `logging` module (levels, handlers, formatting) | ✅ |
| 5 | `assert` statements for sanity checks | ✅ |
| 6 | `pdb` debugger and `breakpoint()` | ✅ |
| 7 | pdb commands (n, s, c, p, l, w) | ✅ |
| 8 | Conditional and strategic breakpoints | ✅ |
| 9 | 6-step debugging methodology | ✅ |
| 10 | Print vs logging vs debugger decision | ✅ |

---

## 🎓 Level 2 Complete!

**You have completed all 13 lessons of Level 2 — Applying!**

| Lesson | Topic | Status |
|:------:|-------|:------:|
| 1 | Default Arguments | ✅ |
| 2 | `*args` and `**kwargs` | ✅ |
| 3 | List Comprehension | ✅ |
| 4 | Built-In Functions in Loops | ✅ |
| 5 | Generator Expressions | ✅ |
| 6 | Lists, Dicts, or Sets | ✅ |
| 7 | Object Attributes | ✅ |
| 8 | Static Variables & Class Methods | ✅ |
| 9 | Mutability & Object References | ✅ |
| 10 | Storing & Retrieving Data | ✅ |
| 11 | Standard Library Imports | ✅ |
| 12 | Virtual Environments & Pip | ✅ |
| 13 | Debugging | ✅ |

> **Next Level →** Level 3: *Building* — Error handling, inheritance, decorators, and real-world project architecture
