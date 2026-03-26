# 🏛️ Level 3 – Building | Lesson 12: Apply Duck Typing in Python

## *"If It Bills Like a Client..."*

> **O'Reilly Skill Objective:** *Apply duck typing to write flexible, Pythonic code that relies on behavior (methods/attributes) rather than explicit type checking.*

> **Previously Learned:** Classes, inheritance, dunder methods, `@property`, `isinstance()`, type hints, `ABC`, exception handling, testing

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

*Mike writes a function that processes only `Client` objects. Louis wants to use it with `Prospect` objects.*

```python
def generate_report(client):
    if not isinstance(client, Client):
        raise TypeError("Expected a Client object!")
    return f"{client.name}: ${client.revenue:,.2f}"
```

**Louis:** "My `Prospect` class has `.name` and `.revenue` too. Why can't I use your function?"

**Mike:** "Because I'm checking the TYPE, not the CAPABILITY."

**Harvey:** "If it has a `.name` and a `.revenue`, it can generate a report. You don't ask for a birth certificate. You ask: can it do the job?"

**Mike:** *"If it walks like a duck and quacks like a duck... it IS a duck."*

```python
# Duck typing — check capability, not identity
def generate_report(entity):
    return f"{entity.name}: ${entity.revenue:,.2f}"
# Works with Client, Prospect, Partner, or ANY object with name + revenue!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Python doesn't care WHAT something is. It cares what it CAN DO."

| Approach | Philosophy | Pythonic? |
|----------|-----------|:---------:|
| **Type checking** | "Are you a `Client`?" | ❌ Rigid |
| **Duck typing** | "Do you have `.name` and `.revenue`?" | ✅ Flexible |
| **Protocols** | "Do you conform to the `Billable` interface?" | ✅ Modern |

```python
# Java/C# mindset: "What TYPE are you?"
def process(obj: Client) -> str: ...    # Only Client objects!

# Python mindset: "What can you DO?"
def process(obj) -> str:                # Anything with the right methods!
    return obj.name + ": " + str(obj.revenue)
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Duck Typing Basics ⭐

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

class Prospect:
    def __init__(self, name, estimated_revenue):
        self.name = name
        self.revenue = estimated_revenue

class Department:
    def __init__(self, name, budget):
        self.name = name
        self.revenue = budget

# Duck-typed function — works with ALL of them!
def display_entity(entity):
    """Works with anything that has .name and .revenue."""
    return f"  {entity.name}: ${entity.revenue:,.2f}"

# All three work — no isinstance check needed!
entities = [
    Client("Wayne Enterprises", 2500000),
    Prospect("Queen Industries", 800000),
    Department("Litigation", 1200000),
]

for e in entities:
    print(display_entity(e))
# Wayne Enterprises: $2,500,000.00
# Queen Industries: $800,000.00
# Litigation: $1,200,000.00
```

---

### 3.2 Duck Typing with Built-in Protocols ⭐⭐

```python
# Python's built-in functions use duck typing!

# len() calls __len__ — works on ANY object with __len__
class Portfolio:
    def __init__(self, clients):
        self._clients = clients
    def __len__(self):
        return len(self._clients)

print(len(Portfolio(["Wayne", "Stark", "Oscorp"])))    # 3
print(len([1, 2, 3]))                                  # 3
print(len("hello"))                                     # 5
# All different types — all work with len()!

# for loops call __iter__ — works on ANY iterable
class CaseQueue:
    def __init__(self, cases):
        self.cases = cases
    def __iter__(self):
        return iter(self.cases)

for case in CaseQueue(["Wayne v. Stark", "Oscorp IP"]):
    print(f"  {case}")
# Works just like iterating a list!

# sorted() uses __lt__ — works on ANY comparable
class Priority:
    def __init__(self, level, name):
        self.level = level
        self.name = name
    def __lt__(self, other):
        return self.level < other.level
    def __repr__(self):
        return f"Priority({self.name})"

priorities = [Priority(3, "Low"), Priority(1, "Critical"), Priority(2, "High")]
print(sorted(priorities))    # [Priority(Critical), Priority(High), Priority(Low)]
```

**Python operations and their duck type protocols:**

| Operation | Requires | Called By |
|-----------|----------|-----------|
| `len(obj)` | `__len__` | `len()` |
| `for x in obj` | `__iter__` | `for` loop |
| `obj[key]` | `__getitem__` | Subscript |
| `str(obj)` | `__str__` | `print()`, f-strings |
| `a + b` | `__add__` | `+` operator |
| `a == b` | `__eq__` | `==` operator |
| `a < b` | `__lt__` | `<`, `sorted()` |
| `bool(obj)` | `__bool__` | `if obj:` |
| `obj()` | `__call__` | Call syntax |

---

### 3.3 EAFP — The Duck Typing Error Strategy ⭐

```python
# LBYL (Look Before You Leap) — check type first
# ❌ Anti-Pythonic
def process_lbyl(entity):
    if hasattr(entity, 'name') and hasattr(entity, 'revenue'):
        return f"{entity.name}: ${entity.revenue:,.2f}"
    else:
        raise TypeError("Entity must have name and revenue")

# EAFP (Easier to Ask Forgiveness) — try it, handle failure
# ✅ Pythonic!
def process_eafp(entity):
    try:
        return f"{entity.name}: ${entity.revenue:,.2f}"
    except AttributeError as e:
        raise TypeError(f"Entity missing required attribute: {e}") from e

# Both work, but EAFP is:
# - More Pythonic
# - Faster in the success case (no hasattr overhead)
# - Handles edge cases better (properties that raise, etc.)
```

---

### 3.4 Protocols — Formalized Duck Typing (Python 3.8+) ⭐⭐

```python
from typing import Protocol, runtime_checkable

# Define WHAT an object must be able to do (not what it IS)
@runtime_checkable
class Billable(Protocol):
    """Any object with name and revenue is Billable."""
    name: str
    revenue: float

@runtime_checkable
class Sortable(Protocol):
    """Any object that can be compared."""
    def __lt__(self, other) -> bool: ...

@runtime_checkable
class Displayable(Protocol):
    """Any object with a display method."""
    def display(self) -> str: ...

# Classes don't need to inherit or declare anything!
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

class Prospect:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

# Both satisfy Billable — without inheriting from it!
print(isinstance(Client("Wayne", 2500000), Billable))     # True
print(isinstance(Prospect("Queen", 800000), Billable))    # True
print(isinstance("just a string", Billable))               # False

# Use in type hints for documentation
def generate_report(entity: Billable) -> str:
    """Works with any Billable entity."""
    return f"{entity.name}: ${entity.revenue:,.2f}"

# IDE knows entity has .name and .revenue — autocompletion works!
```

---

### 3.5 ABC vs Protocol vs Duck Typing — Comparison ⭐⭐

```python
from abc import ABC, abstractmethod
from typing import Protocol, runtime_checkable

# === Approach 1: Abstract Base Class (ABC) ===
# Pro: Enforces implementation at class definition
# Con: Requires inheritance — couples classes together

class BillableABC(ABC):
    @abstractmethod
    def get_billing_amount(self) -> float: ...

class ClientABC(BillableABC):    # MUST inherit
    def get_billing_amount(self) -> float:
        return self.hours * self.rate

# === Approach 2: Protocol ===
# Pro: No inheritance needed — structural typing
# Con: Only checked by type checkers (runtime with @runtime_checkable)

@runtime_checkable
class BillableProtocol(Protocol):
    def get_billing_amount(self) -> float: ...

class ClientProto:               # NO inheritance — just has the method
    def get_billing_amount(self) -> float:
        return self.hours * self.rate

print(isinstance(ClientProto(), BillableProtocol))    # True!

# === Approach 3: Pure Duck Typing ===
# Pro: Maximum flexibility — no boilerplate at all
# Con: Errors caught at runtime, not at definition

class ClientDuck:
    def get_billing_amount(self):
        return self.hours * self.rate

def bill(entity):    # No type declaration at all
    return entity.get_billing_amount()
```

**Decision guide:**

| When to use | Approach |
|-------------|----------|
| Need guaranteed method implementation | ABC |
| Want type hints without inheritance | Protocol |
| Quick/informal, trust collaborators | Duck typing |
| Public library API | Protocol + ABC |
| Internal code, small team | Duck typing |

---

### 3.6 Duck Typing with Iterables ⭐

```python
def total_revenue(entities):
    """Works with lists, tuples, generators, custom iterables!"""
    return sum(e.revenue for e in entities)

# List
clients = [Client("Wayne", 2500000), Client("Stark", 800000)]
print(total_revenue(clients))    # 3300000

# Generator (lazy)
def active_clients():
    yield Client("Wayne", 2500000)
    yield Client("Stark", 800000)

print(total_revenue(active_clients()))    # 3300000

# Custom iterable
class ClientRoster:
    def __init__(self):
        self._clients = []
    def add(self, client):
        self._clients.append(client)
    def __iter__(self):
        return iter(self._clients)

roster = ClientRoster()
roster.add(Client("Wayne", 2500000))
roster.add(Client("Stark", 800000))
print(total_revenue(roster))    # 3300000

# All three work because total_revenue only needs iteration!
```

---

### 3.7 Duck Typing in Function Parameters

```python
# === File-like objects ===
import io

def write_report(output, data):
    """Writes to anything file-like (has .write())."""
    output.write(f"Report: {len(data)} entries\n")
    for entry in data:
        output.write(f"  {entry}\n")

# Works with real files
with open("report.txt", "w") as f:
    write_report(f, ["Wayne", "Stark"])

# Works with StringIO (in-memory)
buffer = io.StringIO()
write_report(buffer, ["Wayne", "Stark"])
print(buffer.getvalue())

# Works with sys.stdout
import sys
write_report(sys.stdout, ["Wayne", "Stark"])

# Any object with .write() works!
import os
os.remove("report.txt")

# === Callable objects ===
def apply_discount(amount, strategy):
    """strategy can be a function, lambda, or callable object."""
    return strategy(amount)

# Function
def flat_discount(amount):
    return amount * 0.9

# Lambda
percentage = lambda amount: amount * 0.85

# Callable object
class TieredDiscount:
    def __call__(self, amount):
        if amount > 100000: return amount * 0.8
        if amount > 50000: return amount * 0.85
        return amount * 0.9

# All three work — they're all callable!
print(apply_discount(100000, flat_discount))         # 90000
print(apply_discount(100000, percentage))             # 85000
print(apply_discount(100000, TieredDiscount()))       # 80000
```

---

### 3.8 When NOT to Duck Type

```python
# === Sometimes type checking IS appropriate ===

# 1. Security — don't trust untrusted input
def transfer_funds(amount, account):
    if not isinstance(amount, (int, float)):
        raise TypeError("Amount must be numeric!")
    if amount <= 0:
        raise ValueError("Amount must be positive!")
    # ...

# 2. API boundaries — validate external data
def api_endpoint(request_data):
    if not isinstance(request_data, dict):
        return {"error": "Expected JSON object"}, 400

# 3. Multiple dispatch — different behavior per type
def process(data):
    if isinstance(data, str):
        return parse_string(data)
    elif isinstance(data, dict):
        return parse_dict(data)
    elif isinstance(data, list):
        return parse_list(data)
    else:
        raise TypeError(f"Unsupported type: {type(data)}")

# 4. Performance — avoiding duck typing overhead
# When you KNOW the type and need maximum speed
```

---

### 3.9 `hasattr`, `getattr`, and Defensive Duck Typing

```python
# === hasattr — check before access ===
def safe_display(entity):
    name = getattr(entity, 'name', 'Unknown')
    revenue = getattr(entity, 'revenue', 0)
    return f"{name}: ${revenue:,.2f}"

print(safe_display(Client("Wayne", 2500000)))   # Wayne: $2,500,000.00
print(safe_display("just a string"))             # Unknown: $0.00

# === Duck typing with graceful degradation ===
def rich_display(entity):
    """Use all available features, degrade gracefully."""
    parts = []
    
    # Required
    try:
        parts.append(entity.name)
    except AttributeError:
        parts.append("Unknown Entity")
    
    # Optional enhancements
    if hasattr(entity, 'revenue'):
        parts.append(f"${entity.revenue:,.2f}")
    
    if hasattr(entity, 'tier'):
        parts.append(f"[{entity.tier}]")
    
    if hasattr(entity, 'case_count'):
        parts.append(f"({entity.case_count} cases)")
    
    return " — ".join(parts)

class FullClient:
    name = "Wayne"
    revenue = 2500000
    tier = "Platinum"
    case_count = 12

class MinimalClient:
    name = "Prospect X"

print(rich_display(FullClient()))
# Wayne — $2,500,000.00 — [Platinum] — (12 cases)

print(rich_display(MinimalClient()))
# Prospect X
```

---

### 3.10 Solving Mike's Problem — Universal Billing System

```python
# ============================================
# Pearson Specter Litt — Universal Billing System
# Duck typing for maximum flexibility
# ============================================

from typing import Protocol, runtime_checkable
from functools import reduce

# === Protocols (optional — for documentation and IDE support) ===

@runtime_checkable
class Billable(Protocol):
    """Anything with name and a billing amount."""
    name: str
    def get_amount(self) -> float: ...

@runtime_checkable
class Displayable(Protocol):
    def display(self) -> str: ...

# === Various entity types — NO shared base class! ===

class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    def get_amount(self):
        return self.revenue
    def display(self):
        return f"  🏢 Client: {self.name} — ${self.revenue:,.2f}"

class Case:
    def __init__(self, title, hours, rate):
        self.name = title
        self.hours = hours
        self.rate = rate
    def get_amount(self):
        return self.hours * self.rate
    def display(self):
        return f"  📋 Case: {self.name} — {self.hours}h × ${self.rate}/hr = ${self.get_amount():,.2f}"

class Expense:
    def __init__(self, description, amount):
        self.name = description
        self.amount = amount
    def get_amount(self):
        return self.amount
    def display(self):
        return f"  💰 Expense: {self.name} — ${self.amount:,.2f}"

class ProBono:
    """Zero billing — still processable!"""
    def __init__(self, client, hours):
        self.name = f"{client} (Pro Bono)"
        self.hours = hours
    def get_amount(self):
        return 0
    def display(self):
        return f"  🤝 Pro Bono: {self.name} — {self.hours}h (no charge)"

# === Duck-typed functions — work with ANY Billable entity ===

def total(entities):
    """Sum get_amount() for any iterable of Billable entities."""
    return sum(e.get_amount() for e in entities)

def display_all(entities):
    """Display any entity that has a .display() method."""
    for e in entities:
        if hasattr(e, 'display'):
            print(e.display())
        else:
            print(f"  ??? {e.name}: ${e.get_amount():,.2f}")

def top_n(entities, n=3):
    """Top N by amount — uses duck-typed .get_amount()."""
    return sorted(entities, key=lambda e: e.get_amount(), reverse=True)[:n]

def group_by_type(entities):
    """Group by class name — works with any type."""
    groups = {}
    for e in entities:
        type_name = type(e).__name__
        groups.setdefault(type_name, []).append(e)
    return groups

def filter_above(entities, threshold):
    """Filter entities above a billing threshold."""
    return [e for e in entities if e.get_amount() > threshold]


# === Demonstrate ===
print("=" * 60)
print(f"{'🏛️  UNIVERSAL BILLING SYSTEM':^60}")
print("=" * 60)
print(f"{'(Duck Typing — No Base Class Required!)':^60}")

# Mix ALL types in one collection!
items = [
    Client("Wayne Enterprises", 2500000),
    Client("Stark Industries", 800000),
    Case("Wayne v. Stark", 85, 450),
    Case("Oscorp IP", 62, 400),
    Expense("Expert Witness Fee", 15000),
    Expense("Court Filing", 2500),
    ProBono("Legal Aid Society", 40),
]

# Protocol check
print(f"\n🔍 Protocol compliance:")
for item in items[:3]:
    print(f"  {item.name}: Billable={isinstance(item, Billable)}")

# Display all
print(f"\n📋 ALL ITEMS:")
display_all(items)

# Totals
print(f"\n💰 TOTALS:")
print(f"  Grand total: ${total(items):,.2f}")

# By type
print(f"\n📊 BY TYPE:")
for type_name, group in group_by_type(items).items():
    subtotal = total(group)
    print(f"  {type_name:>10}: {len(group)} items = ${subtotal:,.2f}")

# Top 3
print(f"\n🏆 TOP 3:")
for item in top_n(items, 3):
    print(f"  {item.name}: ${item.get_amount():,.2f}")

# Filter
big_items = filter_above(items, 10000)
print(f"\n🔍 ABOVE $10K: {len(big_items)} items")
for item in big_items:
    print(f"  {item.name}: ${item.get_amount():,.2f}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Assuming All Ducks Are Equal

```python
# ❌ Both have .name — but different meanings!
class Client:
    def __init__(self):
        self.name = "Wayne Enterprises"    # Company name

class File:
    def __init__(self):
        self.name = "/data/invoices.csv"   # Filepath!

# duck_function processes ANY .name — but are they interchangeable?
# Not always! Context matters.

# ✅ Use Protocols to make intent explicit
from typing import Protocol

class NamedEntity(Protocol):
    """An entity with a business name."""
    name: str
    revenue: float    # Extra attribute makes it clear this isn't a File
```

---

### Pitfall 2: Missing Methods at Runtime

```python
# ❌ Error only when the method is CALLED, not when passed
def process(entity):
    return entity.get_amount()    # AttributeError if missing!

class Incomplete:
    name = "Oops"
    # No get_amount()!

# Fails at runtime, not at definition
# process(Incomplete())    # AttributeError!

# ✅ EAFP — catch and provide helpful error
def process(entity):
    try:
        return entity.get_amount()
    except AttributeError:
        raise TypeError(
            f"{type(entity).__name__} does not support get_amount(). "
            f"It must implement the Billable protocol."
        )
```

---

### Pitfall 3: Over-Using isinstance 🔥

```python
# ❌ Type checking everywhere — defeats duck typing!
def bad_process(entity):
    if isinstance(entity, Client):
        return entity.revenue
    elif isinstance(entity, Case):
        return entity.hours * entity.rate
    elif isinstance(entity, Expense):
        return entity.amount
    else:
        raise TypeError("Unknown type!")

# ✅ Duck typing — one interface, many types
def good_process(entity):
    return entity.get_amount()    # They all have this method!
```

---

### Pitfall 4: Mutable Duck Confusion

```python
# Different objects may return different types
class ClientA:
    def get_amount(self):
        return 2500000        # int

class ClientB:
    def get_amount(self):
        return "2500000"      # String!

# sum() fails on the string version!
# total = sum(e.get_amount() for e in [ClientA(), ClientB()])
# TypeError: unsupported operand type(s) for +: 'int' and 'str'

# ✅ Protocols with type hints catch this in static analysis
class Billable(Protocol):
    def get_amount(self) -> float: ...    # Return type is specified
```

---

### Pitfall 5: Shadowing Built-in Protocols

```python
# ❌ Implementing __len__ that returns a string
class BadContainer:
    def __len__(self):
        return "five"    # TypeError: 'str' object cannot be interpreted as an integer

# ❌ __bool__ that returns non-bool
class Weird:
    def __bool__(self):
        return 42    # Should return True/False!

# ✅ Follow the protocol contract exactly
class GoodContainer:
    def __len__(self):
        return 5    # Must return non-negative int
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🦆 Analogy: Duck Typing = The Firm's Policy on Qualifications

```
TYPE CHECKING (credential-based hiring):
  📜 "Show me your law degree from Harvard."
  → Only Harvard grads can practice here.
  → Mike Ross? Rejected. No degree.
  → But Mike is the best lawyer they've ever had.

DUCK TYPING (performance-based hiring):
  ⚖️ "Can you read a contract?"      → Yes ✅
     "Can you argue a case?"          → Yes ✅
     "Can you generate a report?"     → Yes ✅
  → Mike Ross? Hired. He can DO the job.

PROTOCOL (skill certification):
  📋 "We need someone with these skills:
       ☐ Contract analysis
       ☐ Case argumentation
       ☐ Report generation"
  → Anyone with these skills qualifies.
  → No need to ask WHERE they learned them.

DONNA'S RULE:
  "Don't ask what something IS.
   Ask what something CAN DO.
   The best lawyer at this firm never went to law school."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Universal Formatter**
```
Write display(obj) that works with:
  - Strings (just return it)
  - Numbers (format with commas)
  - Objects with .display() (call it)
  - Objects with __str__ (use str())
  - Dicts (format as key: value)

No isinstance checks! Use EAFP.
```

**Exercise 2: Duck-Typed sort_by**
```
Write sort_by(items, attr) that sorts any objects by attribute:
  sort_by(clients, "revenue")
  sort_by(cases, "hours")
  sort_by(employees, "name")
Works with any type — no type checking.
```

**Exercise 3: Universal Length**
```
Write safe_len(obj) that returns:
  - len(obj) if __len__ exists
  - count of characters if string
  - count of items if iterable (by exhausting)
  - 1 for any single value
  - 0 for None
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Plugin System**
```
Build a plugin system using duck typing:
  - Plugins implement .process(data) → data
  - Any object with a .process method is a valid plugin
  - Chain plugins: data → plugin1 → plugin2 → plugin3

class UpperPlugin:
    def process(self, data): return data.upper()

class TrimPlugin:
    def process(self, data): return data.strip()

pipeline = PluginPipeline([UpperPlugin(), TrimPlugin()])
pipeline.run("  hello  ")    # "HELLO"
```

**Exercise 5: Serializer**
```
Build serialize(obj) that handles any type:
  - Has .to_dict()? Call it
  - Has .to_json()? Call it
  - Has __dict__? Use it
  - Is a dataclass? Use dataclasses.asdict
  - Is a basic type? Return directly
  - Else: use str()
```

**Exercise 6: Protocol Hierarchy**
```
Define a Protocol hierarchy for a document system:

class Readable(Protocol):
    def read(self) -> str: ...

class Writable(Protocol):
    def write(self, data: str) -> None: ...

class ReadWritable(Readable, Writable, Protocol): ...

Show 3 different classes that satisfy these protocols
without inheriting from them.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Adapter Pattern**
```
Build an adapter that makes incompatible objects duck-compatible:

class LegacyClient:
    def get_client_name(self): return "Wayne"
    def get_annual_revenue(self): return 2500000

# Adapter makes it compatible with Billable protocol
adapted = BillableAdapter(LegacyClient())
adapted.name        # "Wayne"
adapted.revenue     # 2500000
adapted.get_amount()  # 2500000
```

**Exercise 8: Generic Repository**
```
Build a Repository that stores ANY type:
  repo = Repository()
  repo.add(Client("Wayne", 2500000))
  repo.add(Case("Merger", 85))
  
  repo.find(lambda x: x.name == "Wayne")
  repo.filter(lambda x: x.get_amount() > 10000)
  repo.sort_by("name")
  repo.group_by(lambda x: type(x).__name__)
```

**Exercise 9: Type-Safe Duck Typing**
```
Build a @duck_check decorator:

@duck_check(name=str, revenue=(int, float), get_amount=callable)
def process(entity):
    ...

process(valid_client)      # ✅ Works
process(invalid_object)    # TypeError with clear message
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: isinstance everywhere
def process(entity):
    if isinstance(entity, Client):
        return entity.revenue
    elif isinstance(entity, Case):
        return entity.hours * entity.rate
    # New type? Must modify this function!

# Bug 2: Assuming attribute types
def total(entities):
    return sum(e.revenue for e in entities)
# Crashes if any entity has revenue as a string!

# Bug 3: hasattr is not enough
class Tricky:
    @property
    def name(self):
        raise RuntimeError("Not available!")

hasattr(Tricky(), 'name')    # Python 3: False (raises caught silently)
# In Python 2: True! (different behavior)

# Bug 4: Confusing duck types
class File:
    name = "data.csv"    # filename

class Client:
    name = "Wayne"       # business name

def greet(entity):
    print(f"Hello, {entity.name}!")

greet(File())     # "Hello, data.csv!" — probably wrong!

# Bug 5: Protocol without @runtime_checkable
class Billable(Protocol):
    def get_amount(self) -> float: ...

isinstance(Client(), Billable)    # TypeError! Not runtime_checkable!
```

### Fixed Code ✅

```python
# Fix 1: Unify interface with duck typing
def process(entity):
    return entity.get_amount()   # ✅ All types implement this!

# Fix 2: Handle type mismatches
def total(entities):
    return sum(float(e.revenue) for e in entities)

# Fix 3: Use EAFP instead of hasattr
try:
    name = entity.name
except (AttributeError, RuntimeError):
    name = "Unknown"

# Fix 4: Use Protocol to distinguish
@runtime_checkable
class BusinessEntity(Protocol):
    name: str
    revenue: float     # File doesn't have this!

# Fix 5: Add @runtime_checkable
@runtime_checkable
class Billable(Protocol):
    def get_amount(self) -> float: ...

isinstance(Client(), Billable)    # ✅ True
```

---

## 8. 🌍 Real-World Application

### Duck Typing in Python Ecosystem

| Library | Duck Typing Usage |
|---------|------------------|
| **Django ORM** | QuerySets behave like lists (iterate, slice, len) |
| **pathlib** | `Path / "file"` — `__truediv__` on path objects |
| **SQLAlchemy** | Session objects are context managers |
| **requests** | Response has `.json()`, `.text`, `.status_code` |
| **pytest** | Fixtures are any callable that yields |
| **logging** | Handlers just need `.emit()` method |
| **Flask** | Request/Response objects duck-type WSGI |
| **typing** | `Protocol` formalizes duck typing for type checkers |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is duck typing?

**Q2.** What is the famous phrase that describes duck typing?

**Q3.** How does Python use duck typing with `len()`?

**Q4.** What is EAFP and how does it relate to duck typing?

**Q5.** What is a `Protocol` and how does it differ from an ABC?

**Q6.** When IS `isinstance()` appropriate?

**Q7.** What is the risk of pure duck typing?

**Q8.** How do you make a Protocol work with `isinstance()`?

**Q9.** What is the difference between structural typing and nominal typing?

**Q10.** Why is duck typing considered "Pythonic"?

---

### 📋 Answer Key

> **Q1.** An approach where you rely on an object's behavior (methods/attributes) rather than its type. If it has the right methods, it works.

> **Q2.** "If it walks like a duck and quacks like a duck, then it IS a duck."

> **Q3.** `len()` calls `obj.__len__()`. Any object with `__len__` works with `len()` — lists, strings, dicts, custom classes.

> **Q4.** "Easier to Ask Forgiveness than Permission" — try the operation and handle `AttributeError` instead of checking types beforehand. It's the error-handling strategy that enables duck typing.

> **Q5.** A `Protocol` defines what methods/attributes an object must have (structural typing). Unlike ABC, classes don't need to inherit from it — they just need to have the right methods.

> **Q6.** Security validation, API boundaries, multiple dispatch (different behavior per type), and when processing untrusted external data.

> **Q7.** Errors are caught at runtime, not definition time. An object might have the right attribute name but wrong semantics (e.g., `File.name` vs `Client.name`).

> **Q8.** Decorate it with `@runtime_checkable`: `@runtime_checkable class MyProtocol(Protocol): ...`

> **Q9.** Structural typing (duck typing/Protocol): compatibility based on structure (has the right methods). Nominal typing (ABC/inheritance): compatibility based on declared type hierarchy.

> **Q10.** Python is dynamically typed — duck typing leverages this by focusing on capability over identity, enabling flexible, reusable, loosely-coupled code.

---

## 🎬 End of Lesson Scene

**Louis:** "So my `Prospect` class works with Mike's billing function?"

**Mike:** "It has `.name` and `.get_amount()`. It walks like a Billable. It quacks like a Billable. It bills like a Billable."

**Harvey:** "Then it IS a Billable."

**Jessica:** "Level 3 complete. You've mastered generators, OOP, exception handling, context managers, testing, and now — the philosophy that ties it all together. If it can do the job, let it do the job."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Duck typing philosophy and EAFP | ✅ |
| 2 | Built-in duck type protocols (`__len__`, `__iter__`, etc.) | ✅ |
| 3 | `Protocol` for formalized structural typing | ✅ |
| 4 | `@runtime_checkable` for isinstance support | ✅ |
| 5 | ABC vs Protocol vs pure duck typing | ✅ |
| 6 | Duck typing with iterables and callables | ✅ |
| 7 | File-like objects and the write protocol | ✅ |
| 8 | When NOT to duck type (security, APIs) | ✅ |
| 9 | `hasattr`/`getattr` for defensive duck typing | ✅ |
| 10 | Adapter pattern for incompatible interfaces | ✅ |

---

## 🎓 Level 3 Complete!

**Congratulations!** You've completed all 12 lessons of Level 3 — Building.

| Lesson | Topic | Status |
|:------:|-------|:------:|
| 1 | Generator Functions | ✅ |
| 2 | Instance Methods | ✅ |
| 3 | Dunder Methods | ✅ |
| 4 | @property | ✅ |
| 5 | Exception Handling | ✅ |
| 6 | Lambda Functions | ✅ |
| 7 | Enums | ✅ |
| 8 | Context Managers | ✅ |
| 9 | Custom Context Managers | ✅ |
| 10 | Logging Module | ✅ |
| 11 | Unit Testing | ✅ |
| 12 | Duck Typing | ✅ |

> **Next → Level 4: Advancing** 🚀
