# 🏛️ Level 2 – Applying | Lesson 1: Define Functions with Default Arguments

## *"The Standard Retainer"*

> **O'Reilly Skill Objective:** *Define a function with default argument values.*

> **Previously Learned (Level 1):** Variables, strings, lists, tuples, dictionaries, `input()`, conditionals, boolean operators, loops, functions, classes, objects, file I/O

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

*Monday morning. Week three. Level 2 begins.*

*You arrive to find the floor reorganized. The associates now sit in a new wing. Harvey's corner office has a new sign: "Advanced Training — Authorized Personnel Only."*

**Harvey:** "Level 1 was kindergarten. You learned the building blocks. Level 2 is where you learn to use them like a professional."

*He pulls up your billing function from Lesson 11.*

```python
def calculate_billing(hours, rate, tax_rate, discount, rush_fee, format_style):
    ...
```

**Harvey:** "Six required arguments. Every time someone calls this function, they have to pass all six — even if the tax rate is always 9%, the discount is usually 0, and there's rarely a rush fee. That's bad API design."

**Mike:** "Default arguments let you set **sensible defaults** so callers only pass what's different from the norm. The common case becomes simple. The special case stays possible."

```python
# Before: Every caller must specify ALL args
calculate_billing(85, 450, 0.09, 0, 0, "standard")

# After: Defaults handle the common case
calculate_billing(85, 450)
```

**Harvey:** "Level 2, Lesson 1. Design functions that are easy to use, hard to misuse."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Think of default arguments like a standard retainer agreement."

**Harvey:** "When a client signs with us, the retainer has standard terms — billing rate, payment schedule, jurisdiction. We don't renegotiate every single term for every client. Clients only specify what's different from the standard."

| Without Defaults | With Defaults |
|------------------|--------------|
| Caller specifies everything every time | Caller only specifies what's different |
| `billing(85, 450, 0.09, 0, 0, "std")` | `billing(85, 450)` |
| Verbose, error-prone calls | Clean, readable calls |
| Hard to remember argument order | Self-documenting |
| Adding new params breaks all callers | New params with defaults = backward compatible |

**Harvey:** "The best functions are like the best contracts — they handle the defaults so you only deal with the exceptions."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Default arguments in depth — going well beyond the basics from Lesson 11."

---

### 3.1 Basic Default Arguments (Review + Expansion)

```python
def greet(name, greeting="Hello", punctuation="!"):
    """Greet someone with configurable greeting and punctuation."""
    return f"{greeting}, {name}{punctuation}"

# All defaults
print(greet("Harvey"))                             # Hello, Harvey!

# Override one default
print(greet("Harvey", greeting="Good morning"))    # Good morning, Harvey!
print(greet("Harvey", punctuation="."))            # Hello, Harvey.

# Override both
print(greet("Harvey", "Welcome", "!!"))            # Welcome, Harvey!!
```

**Mike:** "Rule recap — required parameters FIRST, default parameters AFTER:"

```python
# ✅ Correct
def func(required1, required2, optional1="a", optional2="b"):
    pass

# ❌ SyntaxError — default before required
# def func(optional1="a", required1, required2):
#     pass
```

---

### 3.2 When Default Values Are Evaluated ⭐

**Mike:** "This is the critical distinction that separates professionals from amateurs."

```python
import datetime

def log_event(message, timestamp=datetime.datetime.now()):
    """⚠️ BAD — timestamp is fixed at FUNCTION DEFINITION time!"""
    print(f"[{timestamp}] {message}")

log_event("Event 1")    # [2026-03-13 09:00:00] Event 1
# ... 5 seconds later ...
log_event("Event 2")    # [2026-03-13 09:00:00] Event 2 ← SAME timestamp!
```

**Mike:** "Default values are evaluated **once** — when the function is defined (when Python first reads the `def` line). NOT each time the function is called."

```python
# ✅ FIX: Use None sentinel, compute value inside the function
def log_event(message, timestamp=None):
    if timestamp is None:
        timestamp = datetime.datetime.now()   # Evaluated each call
    print(f"[{timestamp}] {message}")

log_event("Event 1")    # [2026-03-13 09:00:00] Event 1
# ... 5 seconds later ...
log_event("Event 2")    # [2026-03-13 09:00:05] Event 2 ← Correct!
```

---

### 3.3 The Mutable Default Trap — Deep Dive 🔥

**Mike:** "From Lesson 11 you know this is dangerous. Now let's understand *why* at a deeper level."

```python
# ❌ DANGEROUS
def add_case(case, cases=[]):
    cases.append(case)
    return cases

result1 = add_case("Merger")
print(result1)     # ['Merger']

result2 = add_case("Patent")
print(result2)     # ['Merger', 'Patent'] ← SHARED list!

# Even worse:
print(result1)     # ['Merger', 'Patent'] ← result1 ALSO changed!
print(result1 is result2)   # True — SAME object!
```

**Mike:** "What's happening under the hood:"

```python
# When Python reads: def add_case(case, cases=[]):
# Python creates ONE list object: []
# That SAME object is used as the default EVERY TIME
# It's stored as: add_case.__defaults__

print(add_case.__defaults__)   # (['Merger', 'Patent'],) — the shared list!
```

**The complete fix patterns:**

```python
# Pattern 1: None sentinel (most common)
def add_case(case, cases=None):
    if cases is None:
        cases = []
    cases.append(case)
    return cases

# Pattern 2: Conditional expression
def add_case(case, cases=None):
    cases = cases if cases is not None else []
    cases.append(case)
    return cases

# Pattern 3: Walrus operator (Python 3.8+)
def add_case(case, cases=None):
    (cases := cases or []).append(case)
    return cases
```

**Mike:** "This applies to ALL mutable types:"
```python
# ❌ All of these are dangerous defaults
def f(data={}):     ...   # Mutable dict
def f(data=set()): ...    # Mutable set
def f(data=[]):     ...   # Mutable list

# ✅ Use None for all of them
def f(data=None):
    data = data if data is not None else {}
```

---

### 3.4 Default Arguments as Configuration

```python
def generate_invoice(
    client_name,
    hours,
    rate,
    tax_rate=0.09,
    discount_pct=0.0,
    rush_multiplier=1.0,
    currency="USD",
    include_breakdown=True,
    output_format="text"
):
    """
    Generate an invoice with configurable defaults.
    
    Most callers only need: client_name, hours, rate
    Everything else has sensible defaults.
    """
    subtotal = hours * rate * rush_multiplier
    discount = subtotal * discount_pct
    taxable = subtotal - discount
    tax = taxable * tax_rate
    total = taxable + tax
    
    symbols = {"USD": "$", "EUR": "€", "GBP": "£"}
    sym = symbols.get(currency, "$")
    
    if output_format == "text" and include_breakdown:
        print(f"  Invoice for {client_name}")
        print(f"  {'─'*35}")
        print(f"  Hours: {hours} × {sym}{rate:,.2f}")
        if rush_multiplier > 1:
            print(f"  Rush:  ×{rush_multiplier}")
        print(f"  Subtotal: {sym}{subtotal:,.2f}")
        if discount_pct > 0:
            print(f"  Discount: -{sym}{discount:,.2f} ({discount_pct*100:.0f}%)")
        print(f"  Tax:      {sym}{tax:,.2f} ({tax_rate*100:.1f}%)")
        print(f"  {'─'*35}")
        print(f"  Total:    {sym}{total:,.2f}")
    
    return {"subtotal": subtotal, "discount": discount, "tax": tax, "total": total}

# Common case — simple!
generate_invoice("Wayne Enterprises", 85, 450)

# Special case — override only what's different
print()
generate_invoice("Stark Industries", 62, 375,
                 discount_pct=0.10,
                 rush_multiplier=1.5,
                 currency="EUR")
```

---

### 3.5 Combining Positional and Keyword-Only Arguments

```python
# After *, arguments MUST be passed by keyword
def search_clients(
    database,          # Positional (required)
    *,                 # Everything after * is keyword-only
    name=None,         # Keyword-only with default
    min_revenue=0,     # Keyword-only with default
    tier=None,         # Keyword-only with default
    active_only=True   # Keyword-only with default
):
    results = database
    
    if name:
        results = [c for c in results if name.lower() in c["name"].lower()]
    if min_revenue > 0:
        results = [c for c in results if c["revenue"] >= min_revenue]
    if tier:
        results = [c for c in results if c.get("tier") == tier]
    if active_only:
        results = [c for c in results if c.get("active", True)]
    
    return results

clients_db = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "tier": "Platinum", "active": True},
    {"name": "Stark Industries", "revenue": 800000, "tier": "Gold", "active": True},
    {"name": "Oscorp", "revenue": 600000, "tier": "Gold", "active": False},
]

# Clear, readable API calls
results = search_clients(clients_db, min_revenue=500000)
results = search_clients(clients_db, tier="Gold", active_only=False)
results = search_clients(clients_db, name="wayne")

# ❌ Can't pass keyword-only args positionally:
# search_clients(clients_db, "wayne", 500000, "Gold", True)  # TypeError!
```

**Mike:** "The `*` separator forces callers to use keyword arguments — making the code self-documenting and resistant to position-based mistakes."

---

### 3.6 Positional-Only Parameters (Python 3.8+)

```python
# Before /, arguments MUST be passed positionally
def power(base, exponent, /, mod=None):
    if mod is not None:
        return pow(base, exponent, mod)
    return base ** exponent

# ✅ Positional
power(2, 10)         # 1024
power(2, 10, mod=3)  # 2

# ❌ Can't name positional-only params:
# power(base=2, exponent=10)  # TypeError!
```

```python
# Full syntax: positional-only / normal , * keyword-only
def complex_func(
    a, b,           # positional-only (before /)
    /,
    c, d=10,        # normal (positional or keyword)
    *,
    e=20, f=30      # keyword-only (after *)
):
    return a + b + c + d + e + f

complex_func(1, 2, 3)                    # ✅ a=1, b=2, c=3, d=10, e=20, f=30
complex_func(1, 2, c=3, e=100)           # ✅ a=1, b=2, c=3, d=10, e=100, f=30
# complex_func(a=1, b=2, c=3)            # ❌ a,b are positional-only
# complex_func(1, 2, 3, 4, 5)            # ❌ e is keyword-only
```

---

### 3.7 Default Arguments in Class Methods

```python
class Invoice:
    DEFAULT_TAX_RATE = 0.09
    DEFAULT_CURRENCY = "USD"
    
    def __init__(self, client, hours, rate,
                 tax_rate=None,
                 currency=None,
                 discount=0.0):
        self.client = client
        self.hours = hours
        self.rate = rate
        self.tax_rate = tax_rate if tax_rate is not None else Invoice.DEFAULT_TAX_RATE
        self.currency = currency or Invoice.DEFAULT_CURRENCY
        self.discount = discount
    
    def calculate(self):
        subtotal = self.hours * self.rate
        discount_amt = subtotal * self.discount
        taxable = subtotal - discount_amt
        tax = taxable * self.tax_rate
        return taxable + tax
    
    @classmethod
    def quick(cls, client, hours, rate):
        """Factory: create a standard invoice with all defaults."""
        return cls(client, hours, rate)
    
    @classmethod
    def premium(cls, client, hours, rate):
        """Factory: create a premium invoice with rush pricing."""
        return cls(client, hours, rate * 1.5, discount=0)
    
    @classmethod
    def probono(cls, client, hours):
        """Factory: create a pro-bono invoice."""
        return cls(client, hours, 0, tax_rate=0, discount=0)

# Clean API surface
inv1 = Invoice("Wayne", 85, 450)                          # All defaults
inv2 = Invoice("Stark", 62, 375, discount=0.10)           # One override
inv3 = Invoice.probono("Legal Aid", 40)                    # Factory
```

---

### 3.8 Designing Good Default APIs

**Mike:** "The art of default argument design."

| Principle | Example |
|-----------|---------|
| **Defaults should be the common case** | `tax_rate=0.09` — most clients pay 9% |
| **Use `None` for mutable defaults** | `items=None` → `items = items or []` |
| **Use `None` for computed defaults** | `timestamp=None` → `now()` if None |
| **Required params first, optional last** | `def f(name, age, city="NY")` |
| **Use keyword-only for clarity** | `def f(data, *, reverse=False)` |
| **Don't use too many defaults** | If 10+ params, consider a config object |
| **Document default behavior** | Docstrings should explain defaults |

```python
def send_report(
    recipient,                     # Required — who gets it
    subject,                       # Required — what it's about
    *,                             # Everything below is keyword-only
    body="",                       # Optional body text
    format="pdf",                  # Most reports are PDFs
    priority="normal",             # Usually normal priority
    cc=None,                       # Usually no CC
    attach_data=True,              # Usually attach the data
    compress=False,                # Usually don't compress
    retry_on_fail=True,            # Usually retry
    max_retries=3,                 # 3 retries is standard
):
    """Send a report with sensible defaults.
    
    Most callers: send_report("harvey@psl.com", "Q1 Report")
    """
    cc = cc or []
    print(f"  Sending {format.upper()} to {recipient}: {subject}")
    print(f"  Priority: {priority} | CC: {len(cc)} | Compress: {compress}")
    print(f"  Retry: {retry_on_fail} (max {max_retries})")

# Simple call — most defaults are perfect
send_report("harvey@psl.com", "Q1 Revenue Report")

# Customized call — only override what's different
print()
send_report("jessica@psl.com", "Urgent: Case Update",
            priority="high",
            cc=["louis@psl.com"],
            compress=True)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Default arguments have three traps. Two you know. One will surprise you."

---

### Pitfall 1: Mutable Default — The Classic (Full Pattern)

```python
# ❌ Shared between calls
def append_to(item, target=[]):
    target.append(item)
    return target

# Inspect the default object
print(append_to.__defaults__)        # ([],)
append_to("a")
print(append_to.__defaults__)        # (['a'],) — it's mutated!
append_to("b")
print(append_to.__defaults__)        # (['a', 'b'],) — still the same object!

# ✅ The fix
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```

---

### Pitfall 2: Early-Bound Default with Non-Constants

```python
import random

# ❌ default_seed is computed ONCE at definition
def get_random(n=random.randint(1, 100)):
    return n

print(get_random())    # Always the same number!
print(get_random())    # Same number again!

# ✅ Compute inside the function
def get_random(n=None):
    if n is None:
        n = random.randint(1, 100)
    return n

print(get_random())    # Different each time
print(get_random())    # Different each time
```

---

### Pitfall 3: `None` vs `False` / `0` / `""` Ambiguity

```python
# ❌ Bug: 'or' treats 0, False, "" as falsy!
def set_value(x=None):
    x = x or 10       # If x is 0, this becomes 10!

set_value(0)    # x becomes 10, not 0!
set_value("")   # x becomes 10, not ""!
set_value(False)  # x becomes 10!

# ✅ Fix: explicit None check
def set_value(x=None):
    if x is None:
        x = 10

set_value(0)      # x stays 0  ✅
set_value("")     # x stays "" ✅
set_value(False)  # x stays False ✅
```

**Louis:** "Always use `is None`, never use `or`, when `0`, `False`, or `""` are valid inputs."

---

### Pitfall 4: Default Arguments and Inheritance

```python
class Base:
    def process(self, data, verbose=False):
        if verbose:
            print(f"Processing: {data}")
        return data.upper()

class Child(Base):
    # ⚠️ If you override a method, you should match the default values
    def process(self, data, verbose=True):   # Changed default!
        return super().process(data, verbose)

b = Base()
c = Child()

b.process("hello")    # Silent (verbose=False)
c.process("hello")    # Prints! (verbose=True) — different behavior, same call
```

**Louis:** "Changing defaults in subclasses silently alters behavior. Document it clearly."

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Default arguments are like a restaurant order."

### 🍽️ Analogy: Default Arguments = A Restaurant Order Form

```
Order Form:
┌─────────────────────────────────────┐
│  Burger:  ☑ Medium (default)        │  ← You can change this
│  Cheese:  ☑ Cheddar (default)       │  ← Or keep the default
│  Bun:     ☑ Brioche (default)       │  ← Defaults make ordering easy
│  Side:    ☐ __________ (required)   │  ← This MUST be specified
│  Drink:   ☐ __________ (required)   │  ← This MUST be specified
└─────────────────────────────────────┘

Simple order:   "Burger with fries and a coke" (3 things to specify)
Custom order:   "Burger, well-done, Swiss, with fries and water" (5 things)
```

**Donna:** "The most popular options become defaults. You only specify what's different. That's good API design — minimize the decisions for the common case."

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Greeting Generator**
```
Write: greet(name, greeting="Hello", title="", suffix="")
Returns: "{greeting}, {title} {name}{suffix}"

Test:
  greet("Harvey")                        → "Hello, Harvey"
  greet("Harvey", title="Mr.")           → "Hello, Mr. Harvey"
  greet("Harvey", "Good morning", "Mr.") → "Good morning, Mr. Harvey"
  greet("Harvey", suffix=", Esq.")       → "Hello, Harvey, Esq."
```

**Exercise 2: Power Calculator**
```
Write: calculate_power(base, exponent=2, modulo=None)
If modulo is None: return base ** exponent
If modulo is given: return (base ** exponent) % modulo

Test with: (5,), (2,10), (2,10,1000), (3,3,5)
```

**Exercise 3: List Builder**
```
Write: create_list(item, count=1, prefix="")
Returns a list with 'count' copies of 'prefix + item'

Test:
  create_list("file") → ["file"]
  create_list("doc", 3) → ["doc", "doc", "doc"]
  create_list("report", 2, "PSL_") → ["PSL_report", "PSL_report"]
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Configurable Search**
```
Write: search(items, *, query=None, min_value=None, max_value=None,
              sort_by=None, reverse=False, limit=None)

Apply all non-None filters, then sort, then limit.

Test with a list of dictionaries (clients with name, revenue).
Show how keyword-only arguments make calls readable.
```

**Exercise 5: Smart Formatter**
```
Write: format_number(value, *, decimals=2, thousands=True, 
                     currency=None, percentage=False, sign=False)

Test:
  format_number(1234567.8)                    → "1,234,567.80"
  format_number(1234567.8, currency="$")      → "$1,234,567.80"
  format_number(0.157, percentage=True)       → "15.70%"
  format_number(42, sign=True)                → "+42.00"
  format_number(-42, sign=True)               → "-42.00"
```

**Exercise 6: Retry Function**
```
Write: retry(func, *, max_attempts=3, delay=1.0, backoff=2.0,
             on_fail=None, verbose=True)

Calls func() up to max_attempts times.
Delay between attempts increases by backoff multiplier.
If on_fail is provided, call it with the error on each failure.
If verbose, print attempt info.

Test with a function that randomly fails.
```

---

### 🔴 Advanced Exercises

**Exercise 7: HTML Tag Builder**
```
Write: tag(name, content="", *, id=None, class_name=None, 
           style=None, self_closing=False, **attrs)

Returns an HTML string:
  tag("p", "Hello")              → '<p>Hello</p>'
  tag("div", "Hi", id="main")   → '<div id="main">Hi</div>'
  tag("br", self_closing=True)   → '<br />'
  tag("a", "Click", href="/")    → '<a href="/">Click</a>'

Handle arbitrary attributes via **kwargs.
```

**Exercise 8: Query Builder Class**
```
Build a QueryBuilder class with chainable methods that use defaults:

qb = QueryBuilder("clients")
qb.select("name", "revenue")         # default: "*" (all columns)
qb.where(revenue_gt=500000)          # keyword arguments as filters
qb.order_by("revenue", desc=True)    # default: ascending
qb.limit(10)                          # default: no limit
print(qb.build())

Output:
  SELECT name, revenue FROM clients 
  WHERE revenue > 500000 
  ORDER BY revenue DESC 
  LIMIT 10
```

**Exercise 9: Configuration System**
```
Build a Config class with layered defaults:

1. Built-in defaults (hardcoded)
2. File-based config (loaded from JSON)
3. Runtime overrides (passed as arguments)

Runtime > File > Built-in (later layers win)

Methods:
  Config.get(key, default=None)
  Config.set(key, value)
  Config.reset(key)  → back to file/built-in default
  Config.dump()       → print all settings with source labels
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Mutable default
def register(name, attendees=[]):
    attendees.append(name)
    return attendees

event1 = register("Harvey")
event2 = register("Mike")
print(f"Event2: {event2}")   # Expected: ['Mike']

# Bug 2: Early-bound default
from datetime import datetime

def create_entry(text, created=datetime.now()):
    return {"text": text, "created": created}

e1 = create_entry("First")
import time; time.sleep(1)
e2 = create_entry("Second")
print(e1["created"] == e2["created"])   # Expected: False

# Bug 3: None vs falsy
def set_timeout(seconds=None):
    seconds = seconds or 30
    return seconds

result = set_timeout(0)     # Expected: 0 (zero timeout)
print(result)

# Bug 4: Default parameter order
def greet(greeting="Hello", name):
    return f"{greeting}, {name}!"

# Bug 5: Modifying a passed default-like object
def process(config=None):
    config = config or {"verbose": False}
    config["processed"] = True
    return config

default_config = {"verbose": False}
r1 = process(default_config)
print(default_config)   # Expected: {"verbose": False}
```

### Fixed Code ✅

```python
# Fix 1: None sentinel
def register(name, attendees=None):
    if attendees is None:
        attendees = []
    attendees.append(name)
    return attendees

event1 = register("Harvey")
event2 = register("Mike")
print(f"Event2: {event2}")   # ['Mike'] ✅

# Fix 2: Compute inside function
from datetime import datetime

def create_entry(text, created=None):
    if created is None:
        created = datetime.now()
    return {"text": text, "created": created}

# Fix 3: Explicit None check
def set_timeout(seconds=None):
    if seconds is None:
        seconds = 30
    return seconds

result = set_timeout(0)
print(result)   # 0 ✅

# Fix 4: Required params before defaults
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# Fix 5: Don't mutate passed objects — work on a copy
def process(config=None):
    config = dict(config) if config else {"verbose": False}  # Copy!
    config["processed"] = True
    return config

default_config = {"verbose": False}
r1 = process(default_config)
print(default_config)   # {"verbose": False} ✅ Unchanged
```

---

## 8. 🌍 Real-World Application

### Where Default Arguments Shine in Production

| Library/Framework | Default Arguments In Action |
|---|---|
| **requests** | `requests.get(url, timeout=None, headers=None, verify=True)` |
| **pandas** | `pd.read_csv(path, sep=",", header="infer", encoding=None)` |
| **Flask** | `@app.route(path, methods=["GET"])` |
| **SQLAlchemy** | `Column(type, nullable=True, default=None, index=False)` |
| **logging** | `logging.basicConfig(level=WARNING, format=DEFAULT_FORMAT)` |

### Production Example: API Client with Defaults

```python
class PSLApiClient:
    """API client with sensible defaults."""
    
    def __init__(self, base_url, *, api_key=None, timeout=30, 
                 retries=3, verify_ssl=True, verbose=False):
        self.base_url = base_url
        self.api_key = api_key
        self.timeout = timeout
        self.retries = retries
        self.verify_ssl = verify_ssl
        self.verbose = verbose
    
    def get(self, endpoint, *, params=None, timeout=None, headers=None):
        timeout = timeout if timeout is not None else self.timeout
        headers = headers or {}
        if self.api_key:
            headers["Authorization"] = f"Bearer {self.api_key}"
        
        if self.verbose:
            print(f"  GET {self.base_url}/{endpoint}")
            print(f"  Timeout: {timeout}s | Params: {params}")
        
        return {"status": 200, "endpoint": endpoint}

# Simple use — all defaults
client = PSLApiClient("https://api.psl.com")
client.get("clients")

# Configured use — override specific defaults
client = PSLApiClient("https://api.psl.com", 
                       api_key="sk_live_xyz", 
                       timeout=60,
                       verbose=True)
client.get("clients", params={"tier": "platinum"})
```

---

## 9. ✅ Knowledge Check

---

**Q1.** When are default argument values evaluated? What problems can this cause?

**Q2.** Why is `def f(items=[])` dangerous? What's the fix?

**Q3.** What is the difference between `x or default` and `x if x is not None else default`?

**Q4.** What does `*` (bare asterisk) in a parameter list do?

**Q5.** What is the output?
```python
def f(a, b=10, c=20):
    return a + b + c

print(f(1))
print(f(1, 2))
print(f(1, c=5))
```

**Q6.** How can you inspect a function's default values programmatically?

**Q7.** Why should you use `is None` instead of `== None`?

**Q8.** What's wrong with `def f(a=1, b, c=3)`?

**Q9.** Write a function `connect(host, port=5432, *, ssl=True, timeout=30)` and show 3 different ways to call it.

**Q10.** When should you use a config object instead of many default parameters?

---

### 📋 Answer Key

> **Q1.** At function **definition** time (once). This causes issues with mutable objects (shared state) and computed values (frozen timestamps).

> **Q2.** The list is created once and shared across all calls. Fix: `def f(items=None): items = items if items is not None else []`.

> **Q3.** `or` treats `0`, `""`, `False`, `[]` as falsy (replaces them). `is not None` only replaces `None` — preserving valid falsy values.

> **Q4.** Forces all following parameters to be keyword-only.

> **Q5.** `31` (1+10+20), `23` (1+2+20), `16` (1+10+5).

> **Q6.** `func.__defaults__` for positional defaults, `func.__kwdefaults__` for keyword-only defaults.

> **Q7.** `is None` checks identity (is it the None singleton?). `== None` calls `__eq__` which could be overridden.

> **Q8.** SyntaxError — `b` (no default) comes after `a` (has default). Required params must come before defaults.

> **Q9.**
```python
connect("db.psl.com")                      # port=5432, ssl=True, timeout=30
connect("db.psl.com", 3306)                # port=3306, ssl=True, timeout=30
connect("db.psl.com", ssl=False, timeout=5) # port=5432, ssl=False, timeout=5
```

> **Q10.** When you have 10+ parameters, when defaults are complex/dependent, or when the same configuration is reused across multiple function calls.

---

## 🎬 End of Lesson Scene

*Harvey reviews the updated `generate_invoice` function.*

```python
# Before: Every caller confused
generate_invoice("Wayne", 85, 450, 0.09, 0, 1.0, "USD", True, "text")

# After: Crystal clear
generate_invoice("Wayne", 85, 450)
generate_invoice("Wayne", 85, 450, discount_pct=0.10, priority="high")
```

**Harvey:** "Clean. Professional. Self-documenting. That's the kind of code this firm produces."

**Mike:** "Next lesson: `*args` and `**kwargs`. When you don't know how MANY arguments will come in."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Default argument syntax and ordering | ✅ |
| 2 | When defaults are evaluated (definition time!) | ✅ |
| 3 | Mutable default trap — deep understanding | ✅ |
| 4 | `None` sentinel pattern | ✅ |
| 5 | `None` vs falsy (`or` vs `is None`) | ✅ |
| 6 | Keyword-only arguments with `*` | ✅ |
| 7 | Positional-only arguments with `/` | ✅ |
| 8 | Defaults in class methods and factories | ✅ |
| 9 | API design principles for defaults | ✅ |

---

> **Next Lesson →** Level 2, Lesson 2: *Use `*args` and `**kwargs` in Python Functions*
