# 🏛️ Level 4 – Advancing | Lesson 2: Use Closures and Understand Scoping Rules in Python

## *"The Inner Circle"*

> **O'Reilly Skill Objective:** *Use closures and understand lexical scoping in nested functions. Understand variable scope and name resolution (LEGB rule) to write code that avoids scope-related errors.*

> **Previously Learned:** Functions, nested functions, decorators, `*args`/`**kwargs`, `global`, generators, lambda, first-class functions

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

*Mike builds a billing calculator factory. Each attorney needs their own rate, but the rate should be locked in at creation time.*

```python
def make_biller(rate):
    def bill(hours):
        return hours * rate    # rate "remembered" from make_biller!
    return bill

harvey_bill = make_biller(450)
mike_bill = make_biller(300)

print(harvey_bill(10))    # 4500 — uses rate=450
print(mike_bill(10))      # 3000 — uses rate=300

# Even after make_biller finishes, rate survives inside each biller!
```

**Louis:** "How does `bill()` still know `rate`? The `make_biller` function already returned!"

**Mike:** "Closures. When an inner function references a variable from an outer function, Python captures that variable. It's sealed inside — like a confidential file that only the inner function can access."

**Harvey:** "The inner circle. Privileged access."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Scope determines where a name is visible. Closures determine what a function remembers."

| Concept | What It Is |
|---------|-----------|
| **Scope** | Where a variable can be accessed |
| **LEGB Rule** | The lookup order: Local → Enclosing → Global → Built-in |
| **Closure** | An inner function that "captures" variables from its enclosing scope |
| **Free variable** | A variable used in a function but defined in an enclosing scope |
| **`nonlocal`** | Keyword to modify a variable in the enclosing scope |
| **`global`** | Keyword to modify a variable in the module (global) scope |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 The LEGB Rule — Variable Lookup Order ⭐⭐

```python
# L = Local: defined in the current function
# E = Enclosing: defined in an outer (enclosing) function
# G = Global: defined at module level
# B = Built-in: pre-defined names (len, print, range, etc.)

x = "Global"                        # G — Global scope

def outer():
    x = "Enclosing"                 # E — Enclosing scope
    
    def inner():
        x = "Local"                 # L — Local scope
        print(f"Inner sees: {x}")   # → "Local"
    
    inner()
    print(f"Outer sees: {x}")       # → "Enclosing"

outer()
print(f"Module sees: {x}")          # → "Global"
print(f"Built-in: {len}")           # → <built-in function len>  (B)
```

**Lookup order visualized:**

```
inner() looks for 'x':
  1. Local scope (inner)     → Found "Local"?     YES → use it ✅
  2. Enclosing scope (outer) → Found "Enclosing"? (not checked)
  3. Global scope (module)   → Found "Global"?    (not checked)
  4. Built-in scope          → Found x?           (not checked)

If NOT found in Local:
  1. Local scope (inner)     → Found?  NO
  2. Enclosing scope (outer) → Found "Enclosing"? YES → use it ✅
  ...

If NOT found anywhere:
  NameError: name 'x' is not defined
```

---

### 3.2 Local Scope — Function Variables ⭐

```python
def calculate_billing(hours, rate):
    # hours, rate, subtotal, tax, total — all LOCAL
    subtotal = hours * rate
    tax = subtotal * 0.09
    total = subtotal + tax
    return total

result = calculate_billing(85, 450)
# print(subtotal)    # NameError! subtotal is local to calculate_billing

# Each call gets its own local scope
def counter():
    count = 0    # Local — fresh each call!
    count += 1
    return count

print(counter())    # 1
print(counter())    # 1 (not 2! count resets each call)
```

---

### 3.3 Global Scope — Module-Level Variables ⭐

```python
firm_name = "Pearson Specter Litt"    # Global

def display_firm():
    print(f"Firm: {firm_name}")       # ✅ Can READ global

display_firm()    # Firm: Pearson Specter Litt

# ❌ Can't MODIFY global without 'global' keyword
def rename_firm_bad():
    firm_name = "Specter Litt"        # Creates LOCAL variable!
    print(f"Inside: {firm_name}")     # Specter Litt

rename_firm_bad()
print(f"Outside: {firm_name}")        # Still "Pearson Specter Litt"!

# ✅ Use 'global' to modify
def rename_firm_good():
    global firm_name
    firm_name = "Specter Litt"

rename_firm_good()
print(f"Outside: {firm_name}")        # Now "Specter Litt"

# ⚠️ Using 'global' is generally discouraged!
# Better: pass as argument, return new value, use a class
```

---

### 3.4 Enclosing Scope and Closures ⭐⭐⭐

```python
# A CLOSURE is a function that remembers variables
# from its enclosing (outer) function, EVEN AFTER
# the outer function has returned.

def make_biller(attorney_name, rate):
    """Factory: creates a billing function with locked-in rate."""
    
    def bill(client, hours):
        amount = hours * rate    # rate is a FREE VARIABLE (from enclosing scope)
        return f"{attorney_name} billed {client}: ${amount:,.2f}"
    
    return bill    # Returns the closure

# Create closures — each captures its own rate
harvey = make_biller("Harvey", 450)
mike = make_biller("Mike", 300)
louis = make_biller("Louis", 400)

# The outer function (make_biller) is DONE — but rate survives!
print(harvey("Wayne", 85))     # Harvey billed Wayne: $38,250.00
print(mike("Wayne", 85))       # Mike billed Wayne: $25,500.00
print(louis("Wayne", 85))      # Louis billed Wayne: $34,000.00

# Inspect the closure
print(harvey.__closure__)
# (<cell at 0x...: str object at 0x...>, <cell at 0x...: int object at 0x...>)

print(harvey.__closure__[0].cell_contents)   # "Harvey"
print(harvey.__closure__[1].cell_contents)   # 450

# Free variables
print(harvey.__code__.co_freevars)   # ('attorney_name', 'rate')
```

---

### 3.5 `nonlocal` — Modifying Enclosing Variables ⭐⭐

```python
# Without nonlocal: can READ enclosing, but can't MODIFY

def make_counter():
    count = 0
    
    def increment():
        nonlocal count    # ← Required to modify 'count' in enclosing scope!
        count += 1
        return count
    
    def reset():
        nonlocal count
        count = 0
    
    def get():
        return count      # Reading doesn't need nonlocal
    
    return increment, reset, get

inc, reset, get = make_counter()
print(inc())    # 1
print(inc())    # 2
print(inc())    # 3
reset()
print(get())    # 0
print(inc())    # 1

# Without nonlocal:
def broken_counter():
    count = 0
    def increment():
        count += 1    # UnboundLocalError! Assignment makes it LOCAL.
        return count
    return increment
```

**`nonlocal` vs `global`:**

| Keyword | Modifies | Scope Targeted |
|---------|----------|:---:|
| `nonlocal` | Enclosing function's variable | One level up |
| `global` | Module-level variable | Module scope |
| (neither) | Creates LOCAL variable | Current function |

---

### 3.6 Closures as State Containers ⭐⭐

```python
# Closures can replace simple classes for maintaining state

def make_account(owner, initial_balance=0):
    """A bank account using closures instead of a class."""
    balance = initial_balance
    history = []
    
    def deposit(amount):
        nonlocal balance
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        balance += amount
        history.append(f"+${amount:,.2f}")
        return balance
    
    def withdraw(amount):
        nonlocal balance
        if amount > balance:
            raise ValueError(f"Insufficient funds: ${balance:,.2f} available")
        balance -= amount
        history.append(f"-${amount:,.2f}")
        return balance
    
    def get_balance():
        return balance
    
    def get_history():
        return list(history)    # Return a copy
    
    def statement():
        print(f"\n  Account: {owner}")
        print(f"  Balance: ${balance:,.2f}")
        for entry in history:
            print(f"    {entry}")
    
    # Return a namespace-like dict
    return {
        "deposit": deposit,
        "withdraw": withdraw,
        "balance": get_balance,
        "history": get_history,
        "statement": statement,
    }

# Usage
acct = make_account("Wayne Enterprises", 1_000_000)
acct["deposit"](500_000)
acct["withdraw"](200_000)
acct["deposit"](100_000)
acct["statement"]()
# Account: Wayne Enterprises
# Balance: $1,400,000.00
#   +$500,000.00
#   -$200,000.00
#   +$100,000.00
```

---

### 3.7 Closures Power Decorators

```python
import functools
import time

# A decorator IS a closure!
def timer(func):
    """The wrapper closes over 'func' from the enclosing scope."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)    # func is a free variable!
        elapsed = time.perf_counter() - start
        print(f"  ⏱️ {func.__name__}: {elapsed:.4f}s")
        return result
    return wrapper

# Decorator with arguments = MORE closures
def repeat(n):
    """n is captured by the decorator closure."""
    def decorator(func):
        """func is captured by the wrapper closure."""
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):        # n: free variable from repeat
                func(*args, **kwargs)  # func: free variable from decorator
        return wrapper
    return decorator

@repeat(3)
def say_hello(name):
    print(f"  Hello, {name}!")

say_hello("Harvey")
# Hello, Harvey!
# Hello, Harvey!
# Hello, Harvey!

# Inspect the closure chain
print(say_hello.__code__.co_freevars)    # ('func', 'n')
```

---

### 3.8 Scope in Comprehensions and Loops

```python
# List comprehensions have their OWN scope (Python 3)
x = 10
result = [x for x in range(5)]    # x inside is local to comprehension
print(x)    # 10 (unchanged!)

# ❌ The classic loop closure trap
functions = []
for i in range(5):
    functions.append(lambda: i)    # All lambdas share the SAME 'i'!

print([f() for f in functions])    # [4, 4, 4, 4, 4] ← All 4!

# Why? The lambda captures 'i' by REFERENCE, not by VALUE.
# When called, i = 4 (loop is done).

# ✅ Fix 1: Default argument (captures by value)
functions = []
for i in range(5):
    functions.append(lambda i=i: i)    # i=i captures current value!

print([f() for f in functions])    # [0, 1, 2, 3, 4] ✅

# ✅ Fix 2: Use a closure factory
def make_func(i):
    def func():
        return i    # Each closure captures its own 'i'
    return func

functions = [make_func(i) for i in range(5)]
print([f() for f in functions])    # [0, 1, 2, 3, 4] ✅

# ✅ Fix 3: functools.partial
from functools import partial

def return_value(i):
    return i

functions = [partial(return_value, i) for i in range(5)]
print([f() for f in functions])    # [0, 1, 2, 3, 4] ✅
```

---

### 3.9 Built-in Scope and Name Shadowing

```python
# Built-in scope: names pre-defined by Python
print(len([1, 2, 3]))    # 3 — len is in Built-in scope

# ❌ Shadowing built-ins — a common mistake!
list = [1, 2, 3]    # Now 'list' refers to [1, 2, 3], not the built-in!
# list("hello")     # TypeError: 'list' object is not callable!

# Fix: use a different name
my_list = [1, 2, 3]

# ❌ Shadowing print
# print = "hello"
# print("test")    # TypeError: 'str' object is not callable!

# Common victims of shadowing:
# list, dict, set, str, int, float, type, id, input, sum, min, max

# Access shadowed built-ins via builtins module
import builtins
print(builtins.list("hello"))    # ['h', 'e', 'l', 'l', 'o']
```

---

### 3.10 Solving the Billing Problem — Closure-Based System

```python
# ============================================
# Pearson Specter Litt — Closure-Based Billing System
# Closures for state, factories, and encapsulation
# ============================================

def make_billing_system(firm_name):
    """
    Creates an entire billing system using closures.
    All state is encapsulated — no classes, no globals.
    """
    
    # Enclosing state (captured by all inner functions)
    attorneys = {}
    invoices = []
    invoice_counter = [0]    # Mutable container for nonlocal-free modification
    
    def register_attorney(name, rate, role="Associate"):
        """Register an attorney with their billing rate."""
        attorneys[name] = {"rate": rate, "role": role, "total_billed": 0}
        return f"✅ Registered: {name} ({role}) at ${rate}/hr"
    
    def make_biller(attorney_name):
        """Factory: create a billing function for a specific attorney."""
        if attorney_name not in attorneys:
            raise ValueError(f"Unknown attorney: {attorney_name}")
        
        atty = attorneys[attorney_name]
        
        def bill(client, hours, description="Legal services"):
            """Bill a client — captures attorney info via closure."""
            nonlocal invoice_counter
            invoice_counter[0] += 1
            
            amount = hours * atty["rate"]
            invoice = {
                "id": f"INV-{invoice_counter[0]:04d}",
                "attorney": attorney_name,
                "client": client,
                "hours": hours,
                "rate": atty["rate"],
                "amount": amount,
                "description": description,
            }
            invoices.append(invoice)
            atty["total_billed"] += amount
            
            return invoice
        
        bill.attorney = attorney_name
        bill.rate = atty["rate"]
        return bill
    
    def get_report():
        """Generate a billing report — reads enclosing state."""
        total = sum(inv["amount"] for inv in invoices)
        
        lines = [
            f"\n{'='*55}",
            f"{'🏛️  ' + firm_name + ' — Billing Report':^55}",
            f"{'='*55}",
            f"\n  📋 Invoices ({len(invoices)}):",
        ]
        
        for inv in invoices:
            lines.append(
                f"    {inv['id']} | {inv['attorney']:<10} → {inv['client']:<20} "
                f"| {inv['hours']:>4}h × ${inv['rate']}/hr = ${inv['amount']:>10,.2f}"
            )
        
        lines.append(f"\n  👤 Attorney Totals:")
        for name, info in sorted(attorneys.items(), key=lambda x: -x[1]["total_billed"]):
            lines.append(f"    {name:<12} ({info['role']:<10}) ${info['total_billed']:>12,.2f}")
        
        lines.append(f"\n  💰 Grand Total: ${total:,.2f}")
        lines.append(f"{'='*55}")
        
        return "\n".join(lines)
    
    # Return the system's public interface
    return {
        "register": register_attorney,
        "make_biller": make_biller,
        "report": get_report,
    }


# === Demonstrate ===
system = make_billing_system("Pearson Specter Litt")

# Register attorneys
print(system["register"]("Harvey", 450, "Partner"))
print(system["register"]("Mike", 300, "Associate"))
print(system["register"]("Louis", 400, "Partner"))

# Create dedicated billers (closures!)
harvey_bill = system["make_biller"]("Harvey")
mike_bill = system["make_biller"]("Mike")
louis_bill = system["make_biller"]("Louis")

# Bill clients — each biller has its own captured rate
harvey_bill("Wayne Enterprises", 85, "Merger negotiation")
harvey_bill("Stark Industries", 40, "Patent defense")
mike_bill("Wayne Enterprises", 62, "Due diligence")
mike_bill("Oscorp", 55, "Corporate restructuring")
louis_bill("Dalton Industries", 70, "Litigation")

# Generate report — reads ALL enclosing state
print(system["report"]())

# Each biller still knows its own attorney
print(f"\n🔍 harvey_bill is locked to: {harvey_bill.attorney} at ${harvey_bill.rate}/hr")
print(f"🔍 mike_bill is locked to: {mike_bill.attorney} at ${mike_bill.rate}/hr")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: UnboundLocalError — The #1 Scope Bug 🔥🔥

```python
x = 10

def broken():
    print(x)    # Reads 'x'? No! Python sees assignment below → x is LOCAL
    x = 20      # This makes ALL references to 'x' in this function LOCAL!

# broken()    # UnboundLocalError: local variable 'x' referenced before assignment

# WHY? Python decides scope at COMPILE time, not runtime.
# It scans the function body, sees 'x = 20', marks x as LOCAL.
# Then print(x) tries to read the local x, which doesn't exist yet.

# ✅ Fix with global (if module-level)
def fixed_global():
    global x
    print(x)    # 10
    x = 20

# ✅ Fix with nonlocal (if enclosing)
def outer():
    x = 10
    def inner():
        nonlocal x
        print(x)    # 10
        x = 20
    inner()
```

---

### Pitfall 2: Loop Closure Trap 🔥

```python
# ❌ All callbacks share the same loop variable
buttons = []
for i in range(5):
    buttons.append(lambda: print(f"Button {i}"))

buttons[0]()    # Button 4 ← WRONG!
buttons[1]()    # Button 4 ← ALL say 4!

# ✅ Capture with default argument
buttons = []
for i in range(5):
    buttons.append(lambda i=i: print(f"Button {i}"))

buttons[0]()    # Button 0 ✅
buttons[3]()    # Button 3 ✅
```

---

### Pitfall 3: Mutable Closures — Shared State

```python
def make_adders():
    adders = []
    for i in range(3):
        def adder(x):
            return x + i    # i is shared! Not copied!
        adders.append(adder)
    return adders

adders = make_adders()
print(adders[0](10))    # 12 (not 10!) — i = 2 after loop

# ✅ Fix: closure factory
def make_adders_fixed():
    def make_adder(i):
        def adder(x):
            return x + i    # Each closure gets its own 'i'
        return adder
    return [make_adder(i) for i in range(3)]

adders = make_adders_fixed()
print(adders[0](10))    # 10 ✅
print(adders[1](10))    # 11 ✅
```

---

### Pitfall 4: Shadowing Built-in Names

```python
# ❌ Overwriting built-in names
list = [1, 2, 3]        # Now 'list' is no longer the constructor!
# list("hello")         # TypeError!

dict = {"a": 1}          # Now 'dict' is broken!
str = "hello"            # Now 'str' is broken!
type = "merger"           # Now 'type' is broken!

# ✅ Use descriptive names
client_list = [1, 2, 3]
client_dict = {"a": 1}
name_str = "hello"
case_type = "merger"
```

---

### Pitfall 5: `nonlocal` vs `global` Confusion

```python
x = "global"

def outer():
    x = "enclosing"
    
    def inner_nonlocal():
        nonlocal x
        x = "modified by nonlocal"
    
    def inner_global():
        global x
        x = "modified by global"
    
    inner_nonlocal()
    print(f"After nonlocal: {x}")    # "modified by nonlocal"
    
    inner_global()
    print(f"After global (enclosing): {x}")    # "modified by nonlocal" — unchanged!

outer()
print(f"Global x: {x}")    # "modified by global"

# nonlocal → modifies nearest enclosing scope
# global → modifies MODULE scope, skipping enclosing entirely!
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏢 Analogy: Scoping = The Firm's Information Access

```
LEGB = WHO CAN SEE WHAT:

  L = Local Office (your desk)
    → Only YOU see your notes
    → Created when you start working, destroyed when you leave

  E = Enclosing Team (Harvey's office, your team)
    → Your team shares access
    → Closures: you take team secrets with you when reassigned

  G = Global Floor (the firm directory)
    → Everyone can read it
    → Need special authority (global) to change it

  B = Building Directory (universal knowledge)
    → len, print, range — everyone knows these
    → Don't put your name on a built-in office!

CLOSURES = CONFIDENTIAL FILES:
  When Harvey assigns Mike a case, Mike takes the case file.
  Even if Harvey leaves the firm, Mike still has the file.
  The file is sealed — only Mike can access it.

DONNA'S RULE:
  "A closure is a confidential file.
   The outer function wrote it.
   The inner function carries it.
   Nobody else can touch it."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: LEGB Demonstrator**
```
Write a function that demonstrates all 4 LEGB scopes.
Print which scope each variable comes from.
Show what happens when inner shadows outer.
```

**Exercise 2: Counter Factory**
```
Write make_counter(start=0, step=1) that returns:
  increment() → next value
  decrement() → previous value
  reset() → back to start
  value() → current value
```

**Exercise 3: Greeting Factory**
```
Write make_greeter(greeting, punctuation="!"):
  greeter = make_greeter("Hello")
  greeter("Harvey")     # "Hello, Harvey!"
  greeter("Mike")       # "Hello, Mike!"
  
  formal = make_greeter("Good evening", ".")
  formal("Jessica")     # "Good evening, Jessica."
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Accumulator**
```
Write make_accumulator() returning:
  add(value)     → adds value, returns running total
  subtract(value) → subtracts, returns running total
  history()      → list of all operations
  undo()         → reverses last operation
```

**Exercise 5: Rate Limiter (closure-based)**
```
Write make_rate_limiter(max_calls, period_seconds):
  limiter = make_rate_limiter(5, 60)
  limiter()    # True (allowed)
  limiter()    # True
  ...
  limiter()    # False (exceeded 5 calls in 60s)
```

**Exercise 6: Closure vs Class**
```
Implement the SAME functionality using:
  a) A closure-based approach
  b) A class-based approach
  
Compare: code length, readability, performance, testability.
Which would you choose and why?
```

---

### 🔴 Advanced Exercises

**Exercise 7: Event System**
```
Build a closure-based event system:
  events = make_event_system()
  events["on"]("click", handler1)
  events["on"]("click", handler2)
  events["emit"]("click", data)    # Both handlers called
  events["off"]("click", handler1)
```

**Exercise 8: Memoization with Scope Control**
```
Build make_memoizer(max_size=None):
  - Caches function results
  - LRU eviction when max_size reached
  - cache_info() returns hit/miss stats
  - clear() resets cache
  All state in closures — no classes!
```

**Exercise 9: Scope Analyzer**
```
Build a function that analyzes another function:
  analyze(func) → {
    "local_vars": [...],
    "free_vars": [...],
    "global_refs": [...],
    "is_closure": bool,
    "closure_data": {...}
  }
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: UnboundLocalError
total = 100
def add_to_total(amount):
    total = total + amount    # UnboundLocalError!
    return total

# Bug 2: Loop closure trap
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])    # [2, 2, 2] not [0, 1, 2]!

# Bug 3: No nonlocal
def counter():
    n = 0
    def inc():
        n += 1    # UnboundLocalError!
        return n
    return inc

# Bug 4: Shadowing built-in
list = [1, 2, 3]
new_list = list(range(5))    # TypeError!

# Bug 5: global instead of nonlocal
def outer():
    x = 10
    def inner():
        global x    # Modifies MODULE x, not outer's x!
        x = 20
    inner()
    print(x)    # Still 10!
```

### Fixed Code ✅

```python
# Fix 1: Use global or pass as argument
def add_to_total(amount):
    global total
    total = total + amount
    return total

# Fix 2: Default argument capture
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])    # [0, 1, 2] ✅

# Fix 3: Add nonlocal
def counter():
    n = 0
    def inc():
        nonlocal n
        n += 1
        return n
    return inc

# Fix 4: Don't shadow built-ins
my_list = [1, 2, 3]
new_list = list(range(5))    # ✅

# Fix 5: Use nonlocal, not global
def outer():
    x = 10
    def inner():
        nonlocal x    # ✅ Modifies outer's x
        x = 20
    inner()
    print(x)    # 20 ✅
```

---

## 8. 🌍 Real-World Application

### Closures in the Wild

| Pattern | Uses Closures For |
|---------|------------------|
| **Decorators** | Capturing the wrapped function |
| **Callbacks** | Remembering context data |
| **Factory functions** | Configuring behavior at creation |
| **Partial application** | Pre-filling function arguments |
| **Data hiding** | Encapsulation without classes |
| **Event handlers** | Capturing surrounding state |
| **Middleware** | Flask/Django request processing |
| **Memoization** | Caching in enclosed dict |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does LEGB stand for?

**Q2.** What is a closure?

**Q3.** What is a free variable?

**Q4.** When do you need `nonlocal`?

**Q5.** What causes `UnboundLocalError`?

**Q6.** Why does the loop closure trap produce wrong values?

**Q7.** How do you fix the loop closure trap?

**Q8.** What is the difference between `nonlocal` and `global`?

**Q9.** Can closures replace classes?

**Q10.** How do closures relate to decorators?

---

### 📋 Answer Key

> **Q1.** Local → Enclosing → Global → Built-in. The order Python searches for a variable name.

> **Q2.** An inner function that captures and remembers variables from its enclosing function's scope, even after the enclosing function has returned.

> **Q3.** A variable used inside a function but defined in an enclosing scope — it's "free" because it's not local to the function.

> **Q4.** When you need to MODIFY (reassign) a variable in the enclosing scope. Reading doesn't require `nonlocal`.

> **Q5.** When a function has both a read AND an assignment to the same variable name. Python sees the assignment, marks it as local, then the read fails because the local doesn't exist yet.

> **Q6.** Closures capture variables by REFERENCE, not by value. All lambdas in the loop share the same `i`, which equals the final loop value when called.

> **Q7.** Three ways: (1) default argument `lambda i=i:`, (2) closure factory, (3) `functools.partial`.

> **Q8.** `nonlocal` modifies the nearest enclosing function's variable. `global` modifies the module-level variable, skipping all enclosing scopes.

> **Q9.** For simple state management, yes. But classes are better for complex objects, inheritance, and code that others will maintain.

> **Q10.** Every decorator IS a closure. The wrapper function closes over the decorated function (`func` is a free variable captured from the decorator's scope).

---

## 🎬 End of Lesson Scene

**Louis:** "So closures are the reason decorators work?"

**Mike:** "Exactly. When `@timer` wraps a function, the wrapper is a closure that captures `func` from the decorator's scope. The timer decorator returns, but `func` lives on inside the wrapper. That's the enclosing scope at work."

**Harvey:** "The inner circle. Privileged access to information that no one else can touch."

**Mike:** "Next: inheritance. Extending classes to build specialized versions."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | LEGB rule (Local → Enclosing → Global → Built-in) | ✅ |
| 2 | Local scope and function variables | ✅ |
| 3 | Global scope and `global` keyword | ✅ |
| 4 | Enclosing scope and closures | ✅ |
| 5 | Free variables and `__closure__` inspection | ✅ |
| 6 | `nonlocal` keyword for modification | ✅ |
| 7 | Closures as state containers | ✅ |
| 8 | Closures powering decorators | ✅ |
| 9 | Loop closure trap and fixes | ✅ |
| 10 | Built-in scope and name shadowing | ✅ |

---

> **Next Lesson →** Level 4, Lesson 3: *Use Inheritance to Extend Python Classes*
