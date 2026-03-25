# 🏛️ Level 3 – Building | Lesson 6: Use Lambda Functions in Python

## *"The One-Liner"*

> **O'Reilly Skill Objective:** *Use lambda functions for short, anonymous functions.*

> **Previously Learned:** Functions, `*args`/`**kwargs`, comprehensions, generators, `sorted()`, `map()`, `filter()`, closures, higher-order functions

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

*Mike is sorting the client roster. Harvey walks by and sees 5 lines of code for a simple sort.*

```python
def sort_by_revenue(client):
    return client["revenue"]

sorted_clients = sorted(clients, key=sort_by_revenue)
```

**Harvey:** "Five lines to sort a list? You wrote a whole function just to return one value?"

**Mike:** "I need a key function for `sorted()`."

**Harvey:** "Use a lambda. One line."

```python
sorted_clients = sorted(clients, key=lambda c: c["revenue"])
```

**Harvey:** "Lambda functions are throwaway functions — anonymous, inline, one expression. When you need a quick function and naming it would be overkill, use a lambda."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Lambda is a function without a name."

| Feature | Regular Function | Lambda |
|---------|:---:|:---:|
| Keyword | `def` | `lambda` |
| Has a name | ✅ | ❌ (anonymous) |
| Multiple expressions | ✅ | ❌ (single expression) |
| Statements (if/for/while) | ✅ | ❌ (expressions only) |
| Docstrings | ✅ | ❌ |
| Multiple lines | ✅ | ❌ |
| Return keyword | ✅ explicit | ❌ implicit return |
| Readability at scale | ✅ | ❌ (can get cryptic) |

**Syntax:**

```python
lambda parameters: expression

# Equivalent to:
def anonymous(parameters):
    return expression
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Lambda Syntax ⭐

```python
# Regular function
def double(x):
    return x * 2

# Lambda equivalent
double = lambda x: x * 2

# They work the same:
print(double(5))       # 10

# More examples:
add = lambda a, b: a + b
greet = lambda name: f"Hello, {name}"
is_positive = lambda x: x > 0

print(add(3, 4))             # 7
print(greet("Harvey"))        # Hello, Harvey
print(is_positive(-5))        # False
```

**Mike:** "But wait — assigning a lambda to a variable defeats the purpose. If you're going to name it, use `def`. Lambdas shine when used INLINE."

---

### 3.2 Lambdas with `sorted()` — The #1 Use Case ⭐⭐

```python
clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "cases": 12},
    {"name": "Stark Industries", "revenue": 800000, "cases": 8},
    {"name": "Oscorp", "revenue": 600000, "cases": 15},
    {"name": "Dalton Industries", "revenue": 1200000, "cases": 5},
]

# Sort by revenue (ascending)
by_revenue = sorted(clients, key=lambda c: c["revenue"])

# Sort by revenue (descending)
top_clients = sorted(clients, key=lambda c: c["revenue"], reverse=True)

# Sort by name (alphabetical)
by_name = sorted(clients, key=lambda c: c["name"])

# Sort by cases (descending)
most_cases = sorted(clients, key=lambda c: c["cases"], reverse=True)

# Multi-key sort: by revenue, then by name
multi_sort = sorted(clients, key=lambda c: (-c["revenue"], c["name"]))

# Display
for c in top_clients:
    print(f"  {c['name']:<22} ${c['revenue']:>12,}")
# Wayne Enterprises       $2,500,000
# Dalton Industries        $1,200,000
# Stark Industries           $800,000
# Oscorp                     $600,000
```

---

### 3.3 Lambdas with `map()` and `filter()` ⭐

```python
revenues = [2500000, 800000, 600000, 1200000, 150000]

# map: transform each element
formatted = list(map(lambda r: f"${r:,.2f}", revenues))
print(formatted)
# ['$2,500,000.00', '$800,000.00', '$600,000.00', ...]

doubled = list(map(lambda r: r * 2, revenues))
print(doubled)
# [5000000, 1600000, 1200000, 2400000, 300000]

# filter: keep elements that pass the test
big_clients = list(filter(lambda r: r >= 1000000, revenues))
print(big_clients)    # [2500000, 1200000]

small_clients = list(filter(lambda r: r < 500000, revenues))
print(small_clients)  # [150000]

# Combining map + filter
result = list(map(
    lambda r: f"${r/1_000_000:.1f}M",
    filter(lambda r: r >= 1000000, revenues)
))
print(result)         # ['$2.5M', '$1.2M']

# BUT: comprehensions are usually clearer!
result = [f"${r/1_000_000:.1f}M" for r in revenues if r >= 1000000]
print(result)         # ['$2.5M', '$1.2M'] — same result, more readable
```

**Mike:** "For `map()` and `filter()`, list comprehensions are usually cleaner. But for `sorted()`, `min()`, `max()`, and `reduce()` — lambdas are the natural choice."

---

### 3.4 Lambdas with `min()`, `max()`, and `reduce()` ⭐

```python
clients = [
    {"name": "Wayne", "revenue": 2500000},
    {"name": "Stark", "revenue": 800000},
    {"name": "Oscorp", "revenue": 600000},
]

# max/min with key
richest = max(clients, key=lambda c: c["revenue"])
print(richest["name"])     # Wayne

smallest = min(clients, key=lambda c: c["revenue"])
print(smallest["name"])    # Oscorp

# reduce: combine all elements into one
from functools import reduce

total = reduce(lambda acc, c: acc + c["revenue"], clients, 0)
print(f"Total: ${total:,}")   # Total: $3,900,000

# reduce to build a string
names = reduce(lambda acc, c: f"{acc}, {c['name']}", clients, "")[2:]
print(names)   # Wayne, Stark, Oscorp
```

---

### 3.5 Lambdas as Callback Functions ⭐

```python
# Lambda as an event handler
def on_event(event_type, callback):
    """Simulate an event system."""
    print(f"Event '{event_type}' triggered!")
    callback()

on_event("login", lambda: print("  User logged in"))
on_event("logout", lambda: print("  Session cleared"))

# Lambda as a default action
def process(data, transform=lambda x: x):
    """Process data with optional transform (identity by default)."""
    return [transform(item) for item in data]

rates = [450, 400, 375, 500]
print(process(rates))                                    # [450, 400, 375, 500]
print(process(rates, transform=lambda r: r * 1.1))       # [495, 440, 412.5, 550]
print(process(rates, transform=lambda r: f"${r}/hr"))    # ['$450/hr', ...]
```

---

### 3.6 Lambdas with `defaultdict` and `Counter`

```python
from collections import defaultdict

# defaultdict with lambda for custom defaults
client_cases = defaultdict(lambda: {"cases": [], "total": 0})

client_cases["Wayne"]["cases"].append("Merger")
client_cases["Wayne"]["total"] += 38250
client_cases["Stark"]["cases"].append("Patent")
client_cases["Stark"]["total"] += 24800

print(dict(client_cases))
# {'Wayne': {'cases': ['Merger'], 'total': 38250},
#  'Stark': {'cases': ['Patent'], 'total': 24800}}

# Sorting a dictionary by value
scores = {"Harvey": 95, "Mike": 88, "Louis": 72, "Rachel": 91}

# Sort by value (descending)
ranked = sorted(scores.items(), key=lambda item: item[1], reverse=True)
print(ranked)
# [('Harvey', 95), ('Rachel', 91), ('Mike', 88), ('Louis', 72)]

# Top 2
top_two = dict(sorted(scores.items(), key=lambda kv: kv[1], reverse=True)[:2])
print(top_two)   # {'Harvey': 95, 'Rachel': 91}
```

---

### 3.7 Lambda vs `def` — When to Use Which

```python
# ✅ USE LAMBDA: sorting key
sorted(items, key=lambda x: x.revenue)

# ✅ USE LAMBDA: simple callback
button.on_click(lambda: print("Clicked!"))

# ✅ USE LAMBDA: defaultdict factory
d = defaultdict(lambda: 0)

# ✅ USE LAMBDA: simple transform with map
list(map(lambda x: x.upper(), names))

# ❌ DON'T USE LAMBDA: complex logic
# Bad:
process = lambda x: x * 2 if x > 0 else -x * 3 if x < -10 else 0
# Good:
def process(x):
    if x > 0:
        return x * 2
    elif x < -10:
        return -x * 3
    return 0

# ❌ DON'T USE LAMBDA: assigning to a variable
# Bad:
double = lambda x: x * 2
# Good:
def double(x):
    return x * 2

# ❌ DON'T USE LAMBDA: needs a docstring
# Lambdas can't have docstrings!

# ❌ DON'T USE LAMBDA: needs error handling
# Lambdas can't have try/except!
```

**The Rule:**

> If the lambda is longer than the equivalent `def`, use `def`.
> If you're assigning the lambda to a name, use `def`.
> If it needs a docstring, use `def`.
> Lambdas are for SHORT, INLINE, THROWAWAY functions.

---

### 3.8 Conditional Expressions in Lambdas

```python
# Ternary expression (the ONLY conditional available in lambda)
classify = lambda x: "positive" if x > 0 else "negative" if x < 0 else "zero"

print(classify(5))    # positive
print(classify(-3))   # negative
print(classify(0))    # zero

# Practical examples:
tier = lambda rev: "Platinum" if rev >= 1000000 else "Gold" if rev >= 500000 else "Silver"
cap = lambda x, limit: min(x, limit)
clamp = lambda x, lo, hi: max(lo, min(hi, x))

# In sorting:
# Sort with None values last
data = [3, None, 1, None, 2]
sorted(data, key=lambda x: (x is None, x if x is not None else 0))
# [1, 2, 3, None, None]
```

---

### 3.9 `operator` Module — Lambda Alternatives

```python
from operator import itemgetter, attrgetter, methodcaller

clients = [
    {"name": "Wayne", "revenue": 2500000},
    {"name": "Stark", "revenue": 800000},
    {"name": "Oscorp", "revenue": 600000},
]

# itemgetter — replaces lambda for dict/tuple access
# Lambda:
sorted(clients, key=lambda c: c["revenue"])
# itemgetter (faster!):
sorted(clients, key=itemgetter("revenue"))

# Multi-key:
sorted(clients, key=itemgetter("revenue", "name"))

# attrgetter — for object attributes
class Attorney:
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate

attorneys = [Attorney("Harvey", 450), Attorney("Mike", 375)]
sorted(attorneys, key=attrgetter("rate"))

# methodcaller — calls a method
names = ["Harvey", "mike", "LOUIS"]
sorted(names, key=methodcaller("lower"))
# ['Harvey', 'LOUIS', 'mike'] — sorted case-insensitively
```

| `operator` function | Lambda equivalent | Use for |
|:---:|:---:|:---:|
| `itemgetter("key")` | `lambda x: x["key"]` | Dict/tuple access |
| `attrgetter("attr")` | `lambda x: x.attr` | Object attributes |
| `methodcaller("method")` | `lambda x: x.method()` | Method calls |

---

### 3.10 Solving Harvey's Problem — Smart Reporting

```python
# ============================================
# Pearson Specter Litt — Client Report Generator
# Lambda functions for inline data processing
# ============================================

from collections import defaultdict
from functools import reduce

clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "type": "Merger", "attorney": "Harvey", "cases": 12},
    {"name": "Stark Industries", "revenue": 800000, "type": "Patent", "attorney": "Mike", "cases": 8},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation", "attorney": "Louis", "cases": 15},
    {"name": "Dalton Industries", "revenue": 1200000, "type": "Merger", "attorney": "Harvey", "cases": 5},
    {"name": "Queen Industries", "revenue": 950000, "type": "Corporate", "attorney": "Rachel", "cases": 9},
    {"name": "Palmer Tech", "revenue": 450000, "type": "Patent", "attorney": "Mike", "cases": 7},
    {"name": "Luthor Corp", "revenue": 1800000, "type": "Litigation", "attorney": "Louis", "cases": 11},
    {"name": "Rand Corp", "revenue": 350000, "type": "Corporate", "attorney": "Rachel", "cases": 4},
]

print("=" * 60)
print(f"{'🏛️  CLIENT ANALYTICS DASHBOARD':^60}")
print("=" * 60)

# === Lambdas for sorting ===
rev = lambda c: c["revenue"]
fmt = lambda r: f"${r:>12,.2f}" if isinstance(r, (int, float)) else r

print("\n📊 TOP CLIENTS BY REVENUE:")
for c in sorted(clients, key=rev, reverse=True)[:5]:
    tier = (lambda r: "💎" if r >= 1_000_000 else "🥇" if r >= 500_000 else "🥈")(c["revenue"])
    print(f"  {tier} {c['name']:<22} {fmt(c['revenue'])}")

# === Lambdas for aggregation ===
total = reduce(lambda acc, c: acc + c["revenue"], clients, 0)
avg = total / len(clients)
print(f"\n💰 Total Revenue: {fmt(total)}")
print(f"   Average: {fmt(avg)}")

# === Lambdas for grouping ===
print("\n👤 REVENUE BY ATTORNEY:")
attorney_rev = defaultdict(lambda: 0)
for c in clients:
    attorney_rev[c["attorney"]] += c["revenue"]

for atty, rev_total in sorted(attorney_rev.items(), key=lambda kv: kv[1], reverse=True):
    bar = "█" * int(rev_total / max(attorney_rev.values()) * 20)
    print(f"  {atty:<10} {fmt(rev_total)} [{bar}]")

# === Lambdas for filtering ===
print("\n🔍 FILTERS:")
big = list(filter(lambda c: c["revenue"] >= 1_000_000, clients))
print(f"  Clients ≥ $1M: {len(big)}")

mergers = list(filter(lambda c: c["type"] == "Merger", clients))
print(f"  Merger clients: {len(mergers)}")

harveys = list(filter(lambda c: c["attorney"] == "Harvey", clients))
print(f"  Harvey's clients: {len(harveys)}")

# === Lambdas for max/min ===
print(f"\n🏆 Richest: {max(clients, key=rev)['name']}")
print(f"📉 Smallest: {min(clients, key=rev)['name']}")
print(f"📋 Most cases: {max(clients, key=lambda c: c['cases'])['name']}")

# === Lambda for transformation ===
print("\n📋 COMPACT ROSTER:")
compact = list(map(
    lambda c: f"  {c['name']:<22} ({c['type']:<12}) [{c['attorney']}]",
    sorted(clients, key=lambda c: c["type"])
))
print("\n".join(compact))

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Lambda in a Loop — Late Binding 🔥🔥

```python
# ❌ All lambdas capture the SAME variable!
funcs = []
for i in range(5):
    funcs.append(lambda: i)

print([f() for f in funcs])    # [4, 4, 4, 4, 4] — NOT [0, 1, 2, 3, 4]!

# Why? lambda captures the variable 'i', not its VALUE.
# When called, i == 4 (loop ended).

# ✅ Fix: default argument (captures value at creation time)
funcs = []
for i in range(5):
    funcs.append(lambda i=i: i)    # i=i captures current value!

print([f() for f in funcs])    # [0, 1, 2, 3, 4] ✅

# ✅ Even better: list comprehension
funcs = [lambda i=i: i for i in range(5)]
print([f() for f in funcs])    # [0, 1, 2, 3, 4] ✅
```

---

### Pitfall 2: Assigning Lambda to a Variable

```python
# ❌ PEP 8 violation — use def instead
double = lambda x: x * 2

# ✅ PEP 8 approved
def double(x):
    return x * 2

# Why? Lambda assigned to a name:
# - Can't have a docstring
# - Harder to debug (shows as <lambda> in tracebacks)
# - No readability benefit over def
```

---

### Pitfall 3: Lambda Can't Have Statements

```python
# ❌ No print, no assignment, no if/else blocks, no loops
# bad = lambda x: print(x)     # print() works but returns None
# bad = lambda x: x = 5        # SyntaxError!
# bad = lambda x: for i in x   # SyntaxError!
# bad = lambda x: try: ...     # SyntaxError!

# ✅ Lambda can only have a SINGLE EXPRESSION
# If you need statements, use def
```

---

### Pitfall 4: Complex Lambdas Are Unreadable

```python
# ❌ What does this even do?
f = lambda x: (lambda y: y * 2)(x + 1) if x > 0 else (lambda y: -y)(x - 1)

# ✅ Use def — name things!
def transform(x):
    """Double (x+1) if positive, negate (x-1) if not."""
    if x > 0:
        return (x + 1) * 2
    return -(x - 1)
```

---

### Pitfall 5: Lambda Traceback Confusion

```python
# Lambdas show as <lambda> in error messages:
funcs = [
    lambda x: x / 0,
    lambda x: x / 0,
    lambda x: x / 0,
]

# funcs[1](5)
# Traceback:
#   File "script.py", line ?, in <lambda>
#   ZeroDivisionError: division by zero
# Which lambda?? All show as <lambda>!

# ✅ Named functions have clear tracebacks:
def divide_func_1(x): return x / 0
def divide_func_2(x): return x / 0
# Traceback clearly shows: in divide_func_1
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📝 Analogy: Lambda = Sticky Note Instructions

```
NAMED FUNCTION (def):
  📋 A formal procedure with a title and documentation
  "Case Review Protocol (Rev. 3, updated Q1 2026)
   Step 1: Review filings...
   Step 2: Check precedents..."
  → Reusable, documented, filed in the cabinet

LAMBDA FUNCTION:
  📌 A sticky note on Harvey's desk
  "Sort by revenue, highest first"
  → Quick instruction, used once, thrown away

WHEN TO USE WHICH:
  📋 def   → You'll need it again, or it's complex
  📌 lambda → One-time instruction, fits on a sticky note

DONNA'S RULE:
  "If the sticky note needs more than one line, 
   it's not a sticky note — it's a memo. Use def."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Sorting Practice**
```
Given a list of attorneys with name, rate, and win_rate:
  a) Sort by rate (ascending)
  b) Sort by win_rate (descending)  
  c) Sort by name length
  d) Sort by rate descending, then name alphabetically
```

**Exercise 2: Filter and Transform**
```
Given a list of case revenues:
[2500000, 800000, 600000, 1200000, 150000, 950000, 350000]

Using lambda:
  a) Filter to only those >= $500K
  b) Map each to "$(amount)M" format
  c) Find the max and min
  d) Calculate the total using reduce
```

**Exercise 3: Simple Callbacks**
```
Write a function apply_operation(a, b, operation=lambda x, y: x + y):
  - Default: addition
  - Test with: subtraction, multiplication, power
  - All operations passed as lambdas
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Custom Sorter**
```
Build a multi_sort(data, *keys) function where keys
are lambda functions:

result = multi_sort(clients,
    lambda c: -c["revenue"],     # Primary: revenue desc
    lambda c: c["name"],         # Secondary: name asc
)
```

**Exercise 5: Pipeline of Transforms**
```
Build a pipeline function that chains lambdas:

result = pipeline(
    data,
    lambda x: [i for i in x if i > 0],    # filter positive
    lambda x: sorted(x, reverse=True),     # sort descending
    lambda x: x[:5],                       # top 5
    lambda x: sum(x) / len(x),             # average
)
```

**Exercise 6: Lambda vs operator Module**
```
Rewrite these lambdas using operator module functions:
  a) lambda x: x["name"]
  b) lambda x: x.revenue
  c) lambda x: x.upper()
  d) lambda a, b: a + b
  e) lambda x: x[2]

Then benchmark: which is faster for sorting 1M items?
```

---

### 🔴 Advanced Exercises

**Exercise 7: Functional Toolkit**
```
Build compose() and pipe() using lambdas:

double = lambda x: x * 2
add_one = lambda x: x + 1

# compose: right-to-left
f = compose(add_one, double)  # add_one(double(x))
f(5)  # 11

# pipe: left-to-right  
g = pipe(double, add_one)     # add_one(double(x))
g(5)  # 11
```

**Exercise 8: Memoized Lambda**
```
Build a memoize() wrapper that works with lambdas:

fib = memoize(lambda self, n: n if n < 2 else self(n-1) + self(n-2))
print(fib(100))   # instant
```

**Exercise 9: Query DSL**
```
Build a query mini-language using lambdas:

results = (Query(clients)
    .where(lambda c: c["revenue"] > 500000)
    .where(lambda c: c["type"] == "Merger")
    .order_by(lambda c: -c["revenue"])
    .select(lambda c: {"name": c["name"], "rev": c["revenue"]})
    .limit(5)
    .execute())
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Late binding in loop
buttons = []
for label in ["Save", "Load", "Delete"]:
    buttons.append(lambda: print(f"Clicked: {label}"))

buttons[0]()    # "Clicked: Delete" — WRONG!

# Bug 2: Lambda returns None unexpectedly
transform = lambda x: x.append(1)  # .append() returns None!
result = transform([1, 2, 3])
print(result)   # None!

# Bug 3: Lambda with assignment
# bad = lambda x: total = total + x   # SyntaxError!

# Bug 4: Nested lambda madness
f = lambda x: (lambda y: (lambda z: x + y + z))
# f(1)(2)(3)  — works but unreadable

# Bug 5: Lambda vs comprehension confusion
nums = [1, 2, 3, 4, 5]
result = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, nums)))
# Works but... why not a comprehension?
```

### Fixed Code ✅

```python
# Fix 1: Default argument captures current value
buttons = []
for label in ["Save", "Load", "Delete"]:
    buttons.append(lambda l=label: print(f"Clicked: {l}"))

buttons[0]()    # "Clicked: Save" ✅

# Fix 2: Return the new list, not the append result
transform = lambda x: x + [1]      # ✅ Creates new list
result = transform([1, 2, 3])
print(result)   # [1, 2, 3, 1]

# Fix 3: Use def for statements
def accumulate(x):
    global total
    total = total + x

# Fix 4: Use def for clarity
def add_three(x, y, z):
    return x + y + z

# Fix 5: Use comprehension
result = [x**2 for x in nums if x % 2 == 0]   # ✅ Clearer!
```

---

## 8. 🌍 Real-World Application

### Where Lambdas Shine

| Scenario | Example |
|----------|---------|
| **Sorting** | `sorted(data, key=lambda x: x.date)` |
| **Django ORM** | `queryset.annotate(total=Sum('amount'))` |
| **Pandas** | `df.apply(lambda row: row['a'] + row['b'], axis=1)` |
| **GUI callbacks** | `button.config(command=lambda: switch_page(2))` |
| **Default factories** | `defaultdict(lambda: {'count': 0})` |
| **Functional tools** | `reduce(lambda a, b: a * b, numbers)` |
| **Testing** | `mock.side_effect = lambda x: x * 2` |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a lambda function?

**Q2.** What can a lambda contain — expressions or statements?

**Q3.** What is the most common use case for lambda?

**Q4.** Why shouldn't you assign a lambda to a variable?

**Q5.** What is the "late binding" pitfall with lambdas in loops?

**Q6.** How do you fix late binding in a loop lambda?

**Q7.** When should you use `operator.itemgetter` instead of lambda?

**Q8.** Can a lambda have multiple lines?

**Q9.** What shows up in a traceback when a lambda raises an error?

**Q10.** When is a list comprehension preferred over `map()` + lambda?

---

### 📋 Answer Key

> **Q1.** A small anonymous function defined inline with `lambda params: expression`. Returns the expression result automatically.

> **Q2.** Expressions only — no `if`/`for`/`while` statements, no assignments, no `try`/`except`. (Ternary expressions and comprehension expressions are allowed.)

> **Q3.** As a `key` argument to `sorted()`, `min()`, `max()`.

> **Q4.** PEP 8 discourages it: `double = lambda x: x * 2`. Use `def double(x): return x * 2` — named functions have better tracebacks and docstrings.

> **Q5.** Lambdas capture the variable by reference, not by value. In a loop, all lambdas share the final loop value.

> **Q6.** Use a default argument: `lambda i=i: i` — the default is evaluated at creation time, capturing the current value.

> **Q7.** For simple dict/tuple key access: `itemgetter("key")` is faster and clearer than `lambda x: x["key"]`.

> **Q8.** No. A lambda is limited to a single expression. For multi-line logic, use `def`.

> **Q9.** `<lambda>` — no function name, making it harder to debug.

> **Q10.** Almost always — `[x*2 for x in items if x > 0]` is clearer than `list(map(lambda x: x*2, filter(lambda x: x > 0, items)))`.

---

## 🎬 End of Lesson Scene

**Harvey:** *(reviewing the roster)* "One line. Sorted by revenue, descending."

**Mike:** "Lambda functions — sticky notes for Python. Short, inline, anonymous. For quick sorting keys, simple transforms, and throwaway callbacks."

**Harvey:** "And if the sticky note needs more than one line?"

**Mike:** "Then it's not a lambda — it's a `def`."

**Harvey:** "Next: enums. Named constants that give structure to categories and states."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Lambda syntax: `lambda params: expression` | ✅ |
| 2 | Lambdas with `sorted()`, `min()`, `max()` | ✅ |
| 3 | Lambdas with `map()`, `filter()`, `reduce()` | ✅ |
| 4 | Lambda as callbacks and default arguments | ✅ |
| 5 | Conditional expressions in lambdas | ✅ |
| 6 | Lambda vs `def` — when to use which | ✅ |
| 7 | Late binding pitfall and fix (default args) | ✅ |
| 8 | `operator` module alternatives | ✅ |
| 9 | Lambda limitations (no statements, no docstring) | ✅ |
| 10 | Comprehension vs `map()`/`filter()` + lambda | ✅ |

---

> **Next Lesson →** Level 3, Lesson 7: *Create Enums with the enum Module in Python*
