# Phase 2 · Lesson 9 — Debugging
### *"The Closing Argument"*

> **O'Reilly Skill:** Debug Python code using print statements, logging, and the built-in debugger (pdb).
> **DevOps connection:** Your script runs without errors but gives wrong output. Silent failures are the hardest bugs. These are the tools that find them.
> **10-min sprint:** Take a script that outputs wrong results. Add `print(f"{variable=}")` at 3 key points. Find where the value first goes wrong.

---

## Why This Matters (Harvey's take)

The script produces wrong numbers but no exceptions, no crashes. These are the hardest bugs — silent failures. Debugging is detective work: making invisible logic visible. Three tools. Print statements to see values. Logging for a permanent trail. The debugger to freeze time and inspect everything.

---

## The Core Concept (Mike's breakdown)

### Step 0: Read the Error Message

90% of bugs are solved by reading the traceback carefully:

```
Traceback (most recent call last):
  File "billing.py", line 42, in calculate_total
    total = hours * rate
            ~~~~~^~~~~~
TypeError: can't multiply sequence by non-int of type 'str'
```

This tells you: **file** (`billing.py`), **line** (`42`), **function** (`calculate_total`), **problem** (`hours` is a string, not a number). Read it before reaching for any debugging tool.

### Common Error Types

| Error | Meaning | First thing to check |
|-------|---------|---------------------|
| `SyntaxError` | Python can't parse the code | Typo, missing colon/bracket |
| `NameError` | Variable doesn't exist | Spelling, scope, did you define it? |
| `TypeError` | Wrong type for operation | Check types with `type()` or `repr()` |
| `ValueError` | Right type, wrong value | Validate input before using |
| `KeyError` | Dict key doesn't exist | Use `.get()` or check `in` |
| `IndexError` | List index out of range | Check `len()`, off-by-one? |
| `AttributeError` | Object doesn't have that attribute | Check `dir(obj)`, check type |
| `FileNotFoundError` | File doesn't exist | Check path, `os.path.exists()` |

### Print Debugging

```python
def calculate_billing(cases):
    total = 0
    for case in cases:
        print(f"Processing: {case}")          # what are we working with?

        hours = case.get("hours", 0)
        rate = case.get("rate", 0)

        print(f"  hours={hours!r} ({type(hours).__name__})")   # !r shows repr
        print(f"  rate={rate!r} ({type(rate).__name__})")

        amount = hours * rate
        print(f"  amount={amount}")

        total += amount

    print(f"TOTAL: {total}")
    return total
```

### f-string `=` Debugging (Python 3.8+) — your best friend

```python
name = "Harvey"
rate = 450
cases = ["Wayne", "Stark"]

# Without = 
print(f"name: {name}, rate: {rate}")    # name: Harvey, rate: 450

# WITH = — shows variable name AND value automatically
print(f"{name=}")              # name='Harvey'
print(f"{rate=}")              # rate=450
print(f"{type(rate)=}")        # type(rate)=<class 'int'>
print(f"{len(cases)=}")        # len(cases)=2
print(f"{rate * 85=}")         # rate * 85=38250
print(f"{cases[0]=}")          # cases[0]='Wayne'
```

**This is the single most useful debugging trick in Python.** Shows the name and value in one expression.

### Strategic Debug Patterns

```python
# Pattern 1: Marker prints — "Did execution reach here?"
print(">>> ENTERING function")
if condition:
    print(">>> Taking path A")
else:
    print(">>> Taking path B")

# Pattern 2: Toggle debug mode on/off
DEBUG = True

def dprint(*args, **kwargs):
    if DEBUG:
        print("[DEBUG]", *args, **kwargs)

dprint(f"Value is {x}")   # only prints when DEBUG=True
# Change DEBUG=False to silence all debug prints at once

# Pattern 3: Variable dump helper
def dump(**kwargs):
    print("=" * 40)
    for name, value in kwargs.items():
        print(f"  {name} = {value!r} ({type(value).__name__})")
    print("=" * 40)

dump(hours=hours, rate=rate, client=client_name, total=total)
```

### `pdb` — The Interactive Debugger

```python
def calculate_total(cases):
    total = 0
    for case in cases:
        hours = case.get("hours", 0)
        rate = case.get("rate", 0)

        breakpoint()    # execution PAUSES here — you get an interactive prompt
                        # Python 3.7+, same as: import pdb; pdb.set_trace()

        amount = hours * rate
        total += amount
    return total
```

**Key pdb commands:**

| Command | Short | Action |
|---------|:-----:|--------|
| `help` | `h` | Show all commands |
| `next` | `n` | Next line (step OVER function calls) |
| `step` | `s` | Next line (step INTO function calls) |
| `continue` | `c` | Run until next breakpoint |
| `print expr` | `p expr` | Print the value of an expression |
| `list` | `l` | Show code around current line |
| `where` | `w` | Show call stack |
| `quit` | `q` | Exit debugger |

**Example pdb session:**
```
(Pdb) p hours
'62'                     ← FOUND IT! It's a string, not a number
(Pdb) p type(hours)
<class 'str'>
(Pdb) p float(hours) * rate
24800.0                  ← This would be the correct answer
(Pdb) c                  ← continue
```

### `assert` — Catch Bad Assumptions Early

```python
def calculate_billing(hours, rate, client_name):
    # Assert your assumptions at function entry
    assert isinstance(hours, (int, float)), f"hours must be numeric, got {type(hours)}"
    assert hours >= 0, f"hours cannot be negative: {hours}"
    assert rate > 0, f"rate must be positive: {rate}"
    assert client_name, "client_name cannot be empty"

    return hours * rate

# Good call — passes assertions
calculate_billing(85, 450, "Wayne")

# Bad call — catches the bug immediately with a clear message
calculate_billing("85", 450, "Wayne")
# AssertionError: hours must be numeric, got <class 'str'>
```

> ⚠️ Use `assert` for development sanity checks only. For production validation, use `raise ValueError()`. Asserts can be disabled with `python -O`.

---

## DevOps Example — The 6-Step Debugging Method

```python
"""
THE 6-STEP METHOD — use every time:

1. REPRODUCE  — Can you make the bug happen consistently?
               → Write a minimal test that triggers it

2. ISOLATE    — Where EXACTLY does it go wrong?
               → Binary search: print at start/middle/end

3. IDENTIFY   — What is the actual value? What should it be?
               → print(f"{var=}"), check type AND value

4. HYPOTHESIZE — Why is it wrong?
               → Type mismatch? Off-by-one? Wrong variable scope?

5. FIX        — Change the minimum amount of code
               → Fix the root cause, not the symptom

6. VERIFY     — Did the fix work? Did it break anything else?
               → Run the failing case + all other tests
"""

# Example: Binary search debugging
def find_bug(data):
    print(f"INPUT: {data[:3]}... ({len(data)} items)")    # ← Check 1

    processed = transform(data)
    print(f"AFTER TRANSFORM: {processed[:3]}...")          # ← Check 2

    filtered = filter_items(processed)
    print(f"AFTER FILTER: {filtered[:3]}...")              # ← Check 3

    result = aggregate(filtered)
    print(f"RESULT: {result}")                             # ← Check 4

    return result

# If the bug appears between Check 2 and Check 3:
# → The problem is in filter_items()
# Zoom in: add prints inside filter_items()
```

---

## `logging` vs `print` for Debugging (reminder)

| Situation | Use |
|-----------|-----|
| Quick one-off investigation | `print(f"{var=}")` |
| Script you'll run repeatedly | `logging.debug()` |
| Script running in production | `logging.debug()` with file handler |
| Finding where code reaches | `print(">>> STEP 3")` |
| Inspecting values interactively | `breakpoint()` + pdb |

---

## Gotchas (Louis's warnings)

**1. Print doesn't show type issues — use `!r` or `=`:**
```python
x = "42"
print(x)         # 42 — looks like an integer!
print(repr(x))   # '42' — clearly a string (has quotes)
print(f"{x=}")   # x='42' — same thing
```

**2. Don't leave `breakpoint()` in production code:**
```bash
# Breakpoint pauses ALL users — effectively kills the application
# Use environment variable to disable all breakpoints:
PYTHONBREAKPOINT=0 python main.py
```

**3. Never use `except: pass` — it makes debugging impossible:**
```python
# ❌ Swallows the bug — you'll never find it
try:
    result = calculate(data)
except:
    pass

# ✅ At minimum, log it
try:
    result = calculate(data)
except Exception as e:
    logger.error(f"calculate() failed: {e}", exc_info=True)
    result = None
```

**4. Print returns `None` — a common trap:**
```python
x = print("debug")   # print() returns None!
print(x)             # None — not the value you wanted
```

**5. Side effects in debug prints can change behavior:**
```python
# If this has a side effect (e.g., pops from a queue), your "debug" changes the bug
print(f"Value: {queue.pop()}")  # ❌ Changed the data!
item = queue[-1]                # ✅ Peek, don't consume
print(f"Next item: {item=}")
```

---

## Practice (pick one, 10 minutes)

**Beginner:** This function has a bug — use only `print(f"{var=}")` to find it:
```python
def average(numbers):
    total = 0
    for num in numbers:
        total += num
    return total / len(numbers) - 1   # wrong answer
# Expected: average([10, 20, 30]) → 20.0
```

**Intermediate:** Set a `breakpoint()` inside a function that processes a list of dicts. Use pdb to inspect each dict as it's processed. Find the one with a string where a number is expected.

**DevOps-specific:** Write a `debug_mode` context manager. When used as `with debug_mode():`, it sets `DEBUG=True` on entry and `DEBUG=False` on exit. Use it to wrap a suspicious block of code so debug prints only appear inside that block.

---

## Knowledge Check

1. What's the first thing you should do when you see a Python error?
2. What does `print(f"{variable=}")` output?
3. What does `breakpoint()` do?
4. What is the pdb command to step to the next line without entering a function?
5. Why is `except Exception: pass` bad?

**Answers:** (1) Read the full traceback — it tells you the exact file, line, function, and error type. (2) Prints `variable=<value>` — shows the name and value together. (3) Pauses execution and opens an interactive debugger at that line. (4) `n` (next). (5) It silently swallows all errors — bugs become invisible and impossible to find.

---

> **Phase 2 complete!** You now have the full DevOps Python toolkit: exception handling, logging, subprocess, filesystem ops, HTTP/APIs, YAML, templating, virtual environments, and debugging. Time to build something real.
